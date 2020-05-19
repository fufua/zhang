# 龙芯CPU安装uos操作系统的情况下安装csphere容器平台 二进制安装
## 准备工作(计算节点和工作节点都要做这个准备工作)
1.安装网桥  
```
apt-get install bridge-utils
```
ps 配置uos apt仓库源  
echo "deb [trusted=yes] http://pools.deepin.cn/ppa/dde-eagle eagle main non-free contrib" >>/etc/apt/sources.list
echo "deb [trusted=yes] http://uos.packages.chinauos.com/uos eagle main non-free contrib" >>/etc/apt/sources.list
2.修改主机名   
```
hostnamectl set-hostname 
```
3.修改IP为固定IP  
```
vi /etc/network/interfaces
auto eth0 
iface eth0 inet static 
address 10.40.64.130/24
gateway 10.40.64.254
```
4.配置lvm自动挂载 并创建docker和etcd的存储目录  
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
mkdir -p /data/docker /data/etcd2
ln -sv /data/docker /var/lib/docker
ln -sv /data/etcd2 /var/lib/etcd2
```

## 控制节点安装  
1.解压安装包  
tar xf controller.tgz -C /  

2.修改配置文件
/etc/csphere/csphere-agent.env  
CONTROLLER_ADDR=控制器IP:port  

/etc/csphere/csphere-public.env  
```
LOCAL_IP={控制器IP}
NET_MASK= {子网位数}
DEFAULT_GW={默认网关IP}
NETWORK={子网段如192.168.3.0}
```

/etc/csphere/csphere-etcd2-controller.env  
IP更换为实际IP  

eg：  
```
ETCD_DATA_DIR=/var/lib/etcd2
ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
ETCD_ADVERTISE_CLIENT_URLS=http://192.168.14.38:2379
ETCD_LISTEN_PEER_URLS=http://192.168.14.38:2380
ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.14.38:2380
ETCD_INITIAL_CLUSTER=default=http://192.168.14.38:2380
ETCD_DEBUG=true
COS_CLUSTER_SIZE=1
```

/etc/csphere/inst-opts.env  
```
COS_CONTROLLER={ip:port}
COS_INETDEV={网卡名称}
```

/etc/csphere/csphere-agent.env  
```
CONTROLLER_ADDR={控制器IP:80}
```

3.启动docker  
cspherectl start docker  

4.导入mongo数据库镜像  
docker load -i mongo.tar  

5.生成证书  
sh /tls.sh  

6.启动csphere  
cspherectl start  

## 计算节点安装

1.解压安装包  
tar xf agent_install.tgz -C /  

2.修改配置文件    

/etc/csphere/csphere-agent.env  
```
CONTROLLER_ADDR={控制器IP:端口}
DNS_ADDR={本机IP}
AUTH_KEY={登录控制器web页面==>系统设置==>生成cos码}
```

/etc/csphere/csphere-public.env  
```
LOCAL_IP={控制器IP}
NET_MASK={子网位数}
DEFAULT_GW={默认网关IP}
NETMASK={当前子网段如192.168.3.0}
```

/etc/csphere/csphere-etcd2-agent.env  
ip 改为实际IP  

eg:  
```
ETCD_NAME=etcd-1604
ETCD_DATA_DIR=/var/lib/etcd2
ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.14.210:2380
ETCD_ADVERTISE_CLIENT_URLS=http://192.168.14.210:2379
ETCD_LISTEN_PEER_URLS=http://192.168.14.210:2380
ETCD_DEBUG=true
ETCD_DISCOVERY=http://192.168.14.38:2379/v2/keys/discovery/hellocsphere
```

/etc/csphere/inst-opts.env  
```
COS_DISCOVERY_URL
COS_CONTROLLER {控制器IP}
```
IP替换为控制节点IP  

3.sh /tls.sh  

4.搭建br0网桥  
/etc/network/interfaces  

eg:  
```
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
5.重启网卡  
systemctl restart networking  

6.启动csphere  
cspherectl start  

7.下面的命令请注意，在每一台agent机器上执行过一次即可不要每次都加  
#IP范围设置为本机容器子网范围  
net-plugin ip-range --ip-start=192.168.14.112/24 --ip-end=192.168.14.120/24

## 镜像仓库
1.控制器节点执行  

```
docker load -i registry-v2.tar
```

2.页面新建仓库  

