title: weblogic12c集群搭建
categories:
  - weblogic12c
tags:
  - weblogic12c
date: 2019-08-24 03:23:00
---
# weblogic12c集群搭建

## 环境配置
|服务器名|监听地址|监听端口|备注|
|------|------|------|------|
|AdminServer (管理)|10.40.128.182|7001|管理服务器|
|proxy-server|10.40.128.182|8080|代理服务器|
|Server-0|10.40.128.182|9002|受管服务器|
|Server-1|10.40.128.183|9002|受管服务器|

## 启动管理服务器
进去创建的域所在目录，执行`nohup ./bin/startWeblogic.sh &` <br/>
进入控制台[http://10.40.128.182:7001/console](http://10.40.128.182:7001/console)

## 配置服务器
* 点击环境->服务器->新建<br/>
![weblogic8](/images/pasted-4.png)
* 输入服务器地名Server-0，主机IP 10.40.128.182，端口号 9002 集群不用选择<br/>
![weblogic9](/images/pasted-5.png)
* 同理，配置好Server-1，proxy-server<br/>

##创建集群
* 点击环境->集群->新建->集群
* 选择多点传送，不用管多点传送地址
![weblogic10](/images/pasted-6.png)
* 点击确定

##创建计算机
* 创建计算机以配置节点管理器
* 点击环境->计算机->新建，输入计算机名称，操作系统类型<br/>
![weblogic11](/images/pasted-7.png)
* 将连接类型设为普通，监听地址改为10.40.128.182
![weblogic12](/images/pasted-8.png)
* 点击确定

##配置受管服务器的计算机和集群
* 点击环境->服务器，点击服务server-0
* 配置计算机和集群信息
![weblogic13](/images/pasted-9.png)
* 同理配置Server-1，proxy-server 但是procy-server不需要配置集群。

##启动节点管理器
进入创建的域所在的目录，执行<br/>
`nohup ./bin/startNodeManager.sh > nodeManager.log 2>&1 &`

##配置代理服务器
进入weblogic目录下 ./wlserver/common/bin/
执行`./config.sh`<br/>
选择更新现有配置。<br/>
创建代理服务器，服务器选择proxy-server。<br/>
![weblogic14](/images/pasted-10.png)
进入所在域的目录，进入./apps/OracleProxy4_Cluster-0_proxy-server/WEB-INF,修改weblogic.xml。<br/>
修改如图参数为项目的上下文。<br/>
![weblogic15](/images/pasted-11.png)
然后部署更新代理服务器项目。

##启动服务器
复制10.40.128.182上域所在目录，到10.40.128.183下。<br/>
启动183的weblogic。<br/>
在182执行 `./bin/startManagedWeblogic.sh proxy-server http://10.40.128.182:7001`<br/>
`./bin/startManagedWeblogic.sh Server-0 http://10.40.128.182:7001`<br/>
在183执行<br/>
`./bin/startManagedWeblogic.sh Server-1 http://10.30.128.182:7001`<br/>
可在[http://10.40.128.182:7001/console](http://10.40.128.182:7001/console)服务器上看到执行成功。<br/>
(/images/pasted-12.png)

##配置session复制
在项目的WEF-INF文件夹下新增weblogic.xml:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE weblogic-web-app PUBLIC "-//BEA Systems, Inc.//DTD Web Application 8.1//EN"
        "http://www.bea.com/servers/wls810/dtd/weblogic810-web-jar.dtd">
<weblogic-web-app>
    <session-descriptor>
        <session-param>
            <param-name>CookieName</param-name>
            <param-value>JSESSIONID1</param-value>
        </session-param>
        <session-param>
            <param-name>PersistentStoreType</param-name>
            <param-value>replicated_if_clustered</param-value>
        </session-param>
    </session-descriptor>
</weblogic-web-app>
```

##完成
部署项目，完成后。访问[http://10.40.128.182:8080/core](http://10.40.128.182:8080/core)



###踩坑
* 后台免密启动受管服务器
进入sercers文件夹下需要免密启动的服务器文件夹，进入security目录，创建boot.properties。
格式为：

```
username=weblogic
password=weblogic
```
首次启动后自动会被加密<br/>

* 报错信息如下：
![weblogic17](/images/pasted-13.png)
启动时未能连接到管理服务器。可能是节点管理器未能启动的原因。需要先启动管理服务器上的节点管理器。
`nohup ./startNodeManager.sh > nodeManager.log 2>&1 &`0v0

* 项目启动后发现点击登录后仍然跳转到登录页面不报错，怀疑是session复制的问题。经过研究发现是由于2台受管服务器连接的不是同一个redis。修改config.properties中redis.ip为其中一台服务器的ip。2台服务器的redis最好做一个主备。

* websocket在weblogic服务器下无法使用，sockJs会自动切换到xhr-streaming协议。

* 项目启动后，登陆页面后跳转报错，提示服务器资源无法访问。由于weblogic在访问/时会自动跳转到/index.html，因此需要在页面注册注册index.html。
![weblogic18](/images/pasted-14.png)