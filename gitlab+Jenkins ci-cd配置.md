# 一、部署Gitlab+Jenkins环境

```
1、将csphere平台ci-tools镜像导入到自己的平台，并将ci-tools模板也导入到平台
2、部署实例(注意：jenkins要和gitcss部署到一台主机上，不然gitcss不起作用)
```

# 二、配置Gitlab+Jenkins环境

```
1、首次登录Gitlab，需要初始化密码
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E5%88%9D%E5%A7%8B%E5%8C%96%E5%AF%86%E7%A0%81.png)

```
2、配置Gitlab，增加个人访问令牌
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E5%88%9B%E5%BB%BAjenkins%E8%AE%BF%E9%97%AE%E4%BB%A4%E7%89%8C.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E5%BC%80%E5%A7%8B%E5%88%9B%E5%BB%BA.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E5%88%9B%E5%BB%BA%E5%AE%8C%E6%88%90.png)

```
3、配置Gitlab，增加ssh密钥
   
   选择一台Linux主机，执行如下命令：
     ssh-keygen -f /tmp/jenkins.pem -t rsa -C jenkins
   提示输入时按回车即可;
   然后把 /tmp/jenkins.pem.pub 这个文件的内容粘贴到 Gitlab 的 SSH Key 页面
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E5%88%9B%E5%BB%BAssh%E5%AF%86%E9%92%A5.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/ssh%E5%BC%80%E5%A7%8B%E5%88%9B%E5%BB%BA.png)

```
4、登录Jenkins，首次登录的账户和密码均为admin
```
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E7%99%BB%E5%BD%95jenkins.png)

```
5、配置Jenkins与Gitlab通讯密钥配置
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/Jenkins%E5%88%9B%E5%BB%BA%E5%87%AD%E6%8D%AE.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E5%88%9B%E5%BB%BAtoken%E5%87%AD%E6%8D%AE.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E5%88%9B%E5%BB%BAssh%E5%87%AD%E6%8D%AE.png)

```
6、Jenkins全局变量配置
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/Manage%20Jenkins.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/Configure%20System.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/Gitlab.png)

```
7、Gitlab中创建测试代码仓库
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E6%96%B0%E5%BB%BA%E9%A1%B9%E7%9B%AE.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E5%88%9B%E5%BB%BAsms%E9%A1%B9%E7%9B%AE.png)

```
8、Jenkins中创建测试maven项目
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E6%96%B0%E5%BB%BAmaven.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/maven.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/General.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/token.png)
![iamge](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/webhook.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/webhook%20token.png)
![iamge](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/save.png)
![iamge](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E7%AB%8B%E5%8D%B3%E6%9E%84%E5%BB%BA.png)
![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E6%9E%84%E5%BB%BA%E7%BB%93%E6%9E%9C.png)

```
9、Gitlab中配置webhook
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/Gitlab%E9%85%8D%E7%BD%AEwebhook.png)
![iamge](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/webhook%E9%92%A9%E5%AD%90.png)

```
10、测试，在Gitlab测试项目中新建README.md文件，查看Jenkins是否会自动构建
```

![image](https://github.com/lyz-970124/work/blob/master/%E5%9B%BE%E7%89%87/CICD/%E8%87%AA%E5%8A%A8%E6%9E%84%E5%BB%BA.png)

