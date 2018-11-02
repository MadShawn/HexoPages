---
title: jenkins安装的一些坑
date: 2018-11-02 20:03:48
tags:
---

## jenkins 安装

灰常简单，只需要下载 jenkins.war 包，放到 tomcat 的 webapps 目录下即可。。。  
然后就可以通过**http://localhost:8080/jenkins/**进行访问了
常规的安装到这里就可以结束了，下面说的是一些非常规的安装

## 非根目录安装

假如有一天你想把 jenkins 安装到  一个带前缀的路径下，比如**http://localhost:8080/*abc/xyz/*jenkins/**下，就需要这样配置：

1. 把 jenkins.war 包依然放到 tomcat 的 webapps 下；
2. 然后再 tomcat 的 conf/server.xml 中的`<host></host>`内加上如下配置即可：

```xml
<Context path="/abc/xyz/jenkins" docBase="jenkins.war" debug="0" reloadable="true" crossContext="true">
    <Environment name="JENKINS_HOME" type="java.lang.String" value="/your/jenkins/home" override="true"/>
</Context>
```
