# 控制器打包
tar -cvzf csphere-controller3.0.tar.gz /etc/csphere/* /etc/prometheus.yml /lib/systemd/system/csphere-agent.service /lib/systemd/system/csphere-controller.service /lib/systemd/system/prometheus.service /lib/systemd/system/csphere-docker-agent.service /lib/systemd/system/csphere-prepare.service /usr/bin/prometheus /usr/bin/csphere /usr/bin/xc /root/mongodb.img /root/registry.img /root/haproxy.tgz /etc/powerdns/backend.py
# agent打包
tar -cvzf csphere-agent3.0.tar.gz /etc/csphere/* /lib/systemd/system/csphere-agent.service /lib/systemd/system/csphere-docker-agent.service /lib/systemd/system/csphere-prepare.service /etc/resolv.dnsmasq.conf /usr/bin/csphere /usr/bin/xc /root/haproxy.tgz

# 控制器安装

## 配置apt镜像仓库
```
echo "deb [trusted=yes] http://pools.deepin.cn/ppa/dde-eagle eagle main non-free contrib" >>/etc/apt/sources.list
echo "deb [trusted=yes] http://uos.packages.chinauos.com/uos eagle main non-free contrib" >>/etc/apt/sources.list
apt-get update
```
## 修改主机名
hostnamectl set-hostname controller

## 控制器配置为固定IP
```
vi /etc/network/interfaces
auto eth0 
iface eth0 inet static 
address 10.40.64.130/24
gateway 10.40.64.254
```
## 配置lvm自动挂载
```
初始化一下磁盘
dd if=/dev/zero of=/dev/sdb bs=512 count=1
创建物理卷（PV）
pvcreate /dev/sdb
创建并加入卷组（VG）
vgcreate vg_docker /dev/sdb
分配卷组中50G创建逻辑卷（LV）
lvcreate -L 8G -n lv_docker vg_docker  
使用所有空间扩容逻辑卷（LV）  
lvextend -l +100%FREE /dev/vg_docker/lv_docker  
格式化LV  
mkfs.xfs -n ftype=1 /dev/vg_docker/lv_docker  
mkdir /data/  
```
说明:   
应该尽量使用UUID来挂载磁盘, 尤其是在云平台上部署的时候, 防止重启后设备名变化导致系统无法启动.   
可以通过目录`/dev/disk/by-uuid/`或`tune2fs`命令查看对应设备的UUID.   
```
UUID=`blkid /dev/vg_docker/lv_docker  |awk '{print $2}' |cut -d'"' -f2`  
echo "UUID=$UUID /data xfs defaults,prjquota 0 0" >> /etc/fstab  
```
创建docker和etcd的存储目录   
```
mount /data  
mkdir /data/docker
ln -sv /data/docker /var/lib/docker
```


## 配置服务自启动  
```
tar -xvzf csphere-controller3.0.tar.gz -C /
systemctl enable csphere-docker-agent.service
systemctl enable csphere-agent.service
systemctl enable prometheus.service
systemctl enable csphere-controller.service
```
## 配置XC软连接 xc-docker
```
ln -s /usr/bin/xc /usr/bin/docker  
ln -s /usr/bin/xc /usr/bin/docker-containerd  
ln -s /usr/bin/xc /usr/bin/docker-containerd-ctr  
ln -s /usr/bin/xc /usr/bin/docker-containerd-shim  
ln -s /usr/bin/xc /usr/bin/docker-runc  
systemctl start csphere-docker-agent.service  

```

## mongo
```
docker load -i mongodb.img 
docker run --name my-mongo --restart=always --net=host -v /data/db:/var/lib/mongodb -d mongod:3.2.8
```

## pdns

```
apt-get install pdns-server pdns-backend-pipe -y
apt-get install python-pip -y
pip install pymongo

chmod +x /etc/powerdns/backend.py

vi /etc/powerdns/pdns.conf
# 找到 launch= 注释掉 在下面添加如下
# launch=
query-cache-ttl=0
cache-ttl=0
loglevel=7
launch=pipe
pipe-command=/etc/powerdns/backend.py
```

vi /etc/csphere/csphere-agent.env
```
ROLE=agent
CONTROLLER_ADDR=192.168.14.196:80
AUTH_KEY=79a4758348c6ce630f0d5b7d866fe4a5
SVRPOOLID=csphere-internal
```

vi /etc/csphere/csphere-controller.env
```
cat csphere-controller.env
ROLE=controller
AUTH_KEY=79a4758348c6ce630f0d5b7d866fe4a5
DEBUG=true
DB_URL=mongodb://127.0.0.1:27017
DB_NAME=csphere
LISTEN_ADDR=192.168.14.196:80
PDNS_URL=
```

vi /etc/csphere/csphere-public.env
```
cat csphere-public.env
LOCAL_IP=192.168.14.196
NET_MASK=22
DEFAULT_GW=192.168.12.1
NETWORK=192.168.12.0
```

vi /etc/csphere/inst-opts.env
```
COS_ROLE=controller
COS_CONTROLLER=192.168.14.196:80
COS_CONTROLLER_PORT=80
COS_AUTH_KEY=79a4758348c6ce630f0d5b7d866fe4a5
COS_INST_CODE=
COS_DISCOVERY_URL=
COS_SVRPOOL_ID=csphere-internal
COS_CLUSTER_SIZE=1
COS_NETMODE=overlay
COS_INETDEV=enp0s18
COS_MONGOREPL=NO
COS_CUSTOM_DOCKERGW=
COS_CUSTOM_DOCKERDNS=
CONTROLLER_FLOAT_IP=
CONTROLLER_SLB_IP=
COS_QC_VXNET=
COS_QC_ACCESSKEY=
COS_QC_SECRETKEY=
COS_QC_ZONE=
COS_BRIDGE_NAME=
COS_IPVLAN_NAME=
QINGCLOUD_API_ENDPOINT=
```

sh tls.sh  
启动controller  
systemctl start prometheus.service  
systemctl start csphere-controller.service  
systemctl start csphere-agent  

## haproxy

```
docker load -i haproxy.tgz
docker images #查看镜像id
docker tag 69d4ab3b1743 csphere/haproxy:2.1-alpine #69d4ab3b1743 上面查询的镜像id
```

# agent安装

## 配置apt镜像仓库
```
echo "deb [trusted=yes] http://pools.deepin.cn/ppa/dde-eagle eagle main non-free contrib" >>/etc/apt/sources.list
echo "deb [trusted=yes] http://uos.packages.chinauos.com/uos eagle main non-free contrib" >>/etc/apt/sources.list
apt-get update
```
## 修改主机名
hostnamectl set-hostname agent

## 配置网桥
```
apt-get install -y bridge-utils
vi /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d
iface enp0s18 inet manual

auto br0
iface br0 inet static
address 192.168.14.210/22
gateway 192.168.12.1
bridge_ports enp0s18
bridge_set off
bridge_fd 0 
```
ifconfig查看网桥有没有配置成功

## 配置lvm自动挂载
```
初始化一下磁盘
dd if=/dev/zero of=/dev/sdb bs=512 count=1
创建物理卷（PV）
pvcreate /dev/sdb
创建并加入卷组（VG）
vgcreate vg_docker /dev/sdb
分配卷组中50G创建逻辑卷（LV）
lvcreate -L 8G -n lv_docker vg_docker
使用所有空间扩容逻辑卷（LV）
lvextend -l +100%FREE /dev/vg_docker/lv_docker
格式化LV
mkfs.xfs -n ftype=1 /dev/vg_docker/lv_docker
mkdir /data/
```
说明:  
应该尽量使用UUID来挂载磁盘, 尤其是在云平台上部署的时候, 防止重启后设备名变化导致系统无法启动.  
可以通过目录`/dev/disk/by-uuid/`或`tune2fs`命令查看对应设备的UUID.  
```
UUID=`blkid /dev/vg_docker/lv_docker  |awk '{print $2}' |cut -d'"' -f2`
echo "UUID=$UUID /data xfs defaults,prjquota 0 0" >> /etc/fstab
```
创建docker和etcd的存储目录  
```
mount /data  
mkdir /data/docker  
ln -sv /data/docker /var/lib/docker  
```

## 配置服务自启动
```
tar -xvzf csphere-agent3.0.tar.gz -C /

systemctl enable csphere-agent.service
systemctl enable csphere-docker-agent.service
```
## xc-docker

```
ln -s /usr/bin/xc /usr/bin/docker
ln -s /usr/bin/xc /usr/bin/docker-containerd
ln -s /usr/bin/xc /usr/bin/docker-containerd-ctr
ln -s /usr/bin/xc /usr/bin/docker-containerd-shim
ln -s /usr/bin/xc /usr/bin/docker-runc
```
## 启动docker
```
systemctl start csphere-docker-agent.service
docker network create --driver=bridge --subnet=192.168.12.0/22 --gateway=192.168.12.1  -o com.docker.network.bridge.name=br0 -o com.docker.network.bridge.enable_ip_masquerade=false bridge1
注：子网需要根据当前网路来决定
```
 


vi /etc/csphere/csphere-agent.env
```
NETWORK=bridge1对应上面创建docker网络的bridge1
```

vi /etc/csphere/csphere-public.env
```
LOCAL_IP=192.168.14.197
NET_MASK=22
DEFAULT_GW=192.168.12.1
NETWORK=192.168.12.0
```
vi /etc/csphere/inst-opts.env
```
COS_ROLE=agent
COS_CONTROLLER=192.168.14.197:80
COS_CONTROLLER_PORT=
COS_AUTH_KEY=
COS_INST_CODE=5114
COS_SVRPOOL_ID=
COS_CLUSTER_SIZE=
COS_NETMODE=ipvlan
COS_INETDEV=eth0
COS_MONGOREPL=
COS_CUSTOM_DOCKERGW=
COS_CUSTOM_DOCKERDNS=
CONTROLLER_FLOAT_IP=
CONTROLLER_SLB_IP=
COS_QC_VXNET=
COS_QC_ACCESSKEY=
COS_QC_SECRETKEY=
COS_QC_ZONE=
COS_BRIDGE_NAME=
COS_IPVLAN_NAME=
QINGCLOUD_API_ENDPOINT=
```
## dnsmasq

```
apt-get install dnsmasq -y
```

vi /etc/resolv.conf
```
# DNS 入口（本机的IP）
nameserver 192.168.14.200
```
vi /etc/resolv.dnsmasq.conf
```
# 上级DNS
nameserver 114.114.114.114
```

vi /etc/dnsmasq.conf
```
port=53
resolv-file=/etc/resolv.dnsmasq.conf
strict-order
cache-size=2500
# 控制器DNS的IP
server=/csphere.local/192.168.14.22
```
## 启动dnsmasq
```
systemctl enable dnsmasq.service 
systemctl restart dnsmasq.service
```


## haproxy

```
docker load -i haproxy.tgz
docker images #查看镜像id
docker tag 69d4ab3b1743 csphere/haproxy:2.1-alpine #69d4ab3b1743 上面查询的镜像id
```

## 启动agent

sh tls.sh  
systemctl start csphere-agent.service  
