title: Oracle修复系统包损坏
categories:
  - Oracle
tags:
  - Oracle
date: 2019-08-21 23:03:00
---
##### 问题：项目上遇到了dbms_aq包损坏的情况，导致高级队列无法正常使用，oracle定时任务也无法启动。


![upload successful](/images/pasted-3.png)

##### 解决办法：oracle加密系统包损坏后无法通过单独编译包来恢复，需要使用内置脚本catproc.sql重新编译整个oracle的系统工具包。

##### 操作方法：

* 1 在oracle服务器上找到catproc.sql文件
    

![upload successful](/images/pasted-1.png)

* 2 sqlplus下执行此脚本，执行时间可能较久
   

![upload successful](/images/pasted-2.png)

* 3 修复完成后有些过程可能需要重新授权