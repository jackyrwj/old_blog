---

layout: post
title: "上传sdk到maven公共仓库"
date: 2023-6-26
tags: [maven]
comments: true
author: jackyrwj
toc: true

---
在网关项目客户端模块中，基于Spring Boot Starter开发了一个客户端SDK方便用户调用。
并把这个sdk上传到远程maven仓库方便用户直接引入依赖来调用。本文介绍一下大致的流程以及容易遇到的问题。

1、注册一个https://issues.sonatype.org/的账号
![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230626213038.png)



新建一个项目，注意问题类型要为new project 概要和描述正常写就行，
首先要确定封装好的sdk已经上传到GitHub仓库，那么此处的groupid填github的groupid。
project url就填github地址，scm url就填github地址后加.git
点击创建就可以了，这个过程需要仓库那边机器人审核，等待大概十来分钟。
![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230626213605.png)

过一会刷新页面后发现被驳回了, 主要的问题是 groupId 必须io.github.jackyrwj,还需要用jackyrwj这个账号创建一个临时仓库, 验证这是你的仓库。
![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230626213747.png)
![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230626213848.png)


2、安装并配置GPG
下载Gpg4win 3.1.5秘钥生成工具 下载跳转
一个傻瓜式的秘钥生成工具，正常生成就可以，记住passphrase，生成后点击发布：
```java
# 将公钥发布到GPG密钥服务器 gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 公钥ID
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys C66BFD3024684919EA795F3EDD1B77F983DC64C4
```
![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230626213925.png)

3、配置maven的setting.xml
```java
      <server>
          <id>ossrh</id>
          <username>sonatype账号</username>
          <password>sonatype密码</password>
      </server>

```

4、配置项目pom.xml文件
distributionManagement节和maven-compiler-plugin节的配置信息根据自己的实际情况做修改。注意groupid与创建问题时的groupid保持一致，version需要设置
<version>1.0-SNAPSHOP</version>
```java
```Java
<!--<?xml version="1.0" encoding="UTF-8"?>-->
<!--<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"-->
<!--         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">-->
<!--    <modelVersion>4.0.0</modelVersion>-->
<!--    <parent>-->
<!--        <groupId>org.springframework.boot</groupId>-->
<!--        <artifactId>spring-boot-starter-parent</artifactId>-->
<!--        <version>2.7.5</version>-->
<!--        <relativePath/> &lt;!&ndash; lookup parent from repository &ndash;&gt;-->
<!--    </parent>-->
<!--    <groupId>com.yupi</groupId>-->
<!--    <artifactId>yuapi-interface</artifactId>-->
<!--    <version>0.0.1-SNAPSHOT</version>-->
<!--    <name>yuapi-interface</name>-->
<!--    <description>yuapi-interface</description>-->
<!--    <properties>-->
<!--        <java.version>1.8</java.version>-->
<!--    </properties>-->
<!--    <dependencies>-->
<!--        <dependency>-->
<!--            <groupId>com.yupi</groupId>-->
<!--            <artifactId>yuapi-client-sdk</artifactId>-->
<!--            <version>0.0.1</version>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.springframework.boot</groupId>-->
<!--            <artifactId>spring-boot-starter-web</artifactId>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>cn.hutool</groupId>-->
<!--            <artifactId>hutool-all</artifactId>-->
<!--            <version>5.8.16</version>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.springframework.boot</groupId>-->
<!--            <artifactId>spring-boot-devtools</artifactId>-->
<!--            <scope>runtime</scope>-->
<!--            <optional>true</optional>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.projectlombok</groupId>-->
<!--            <artifactId>lombok</artifactId>-->
<!--            <optional>true</optional>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>org.springframework.boot</groupId>-->
<!--            <artifactId>spring-boot-starter-test</artifactId>-->
<!--            <scope>test</scope>-->
<!--        </dependency>-->
<!--    </dependencies>-->

<!--    <build>-->
<!--        <plugins>-->
<!--            <plugin>-->
<!--                <groupId>org.springframework.boot</groupId>-->
<!--                <artifactId>spring-boot-maven-plugin</artifactId>-->
<!--                <configuration>-->
<!--                    <excludes>-->
<!--                        <exclude>-->
<!--                            <groupId>org.projectlombok</groupId>-->
<!--                            <artifactId>lombok</artifactId>-->
<!--                        </exclude>-->
<!--                    </excludes>-->
<!--                </configuration>-->
<!--            </plugin>-->
<!--        </plugins>-->
<!--    </build>-->


<!--</project>-->
```


5、发布
修改完后就可以deploy了  在cmd中执行命令:
mvn -U clean deploy -P release -DskipTests -Dgpg.passphrase=密钥
或者
mvn clean deploy -Darguments= "gpg.passphrase=密钥"
第一次部署需要设置distributionManagement为以下内容
```java
<distributionManagement>
    <repository>
        <id>ossrh</id>
        <name>Sonatype Nexus Snapshots</name>
        <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2</url>
    </repository>
    <snapshotRepository>
        <id>ossrh</id>
        <name>Nexus Release Repository</name>
        <url>https://s01.oss.sonatype.org/content/repositories/snapshots//</url>
    </snapshotRepository>
</distributionManagement>
```

上传成功如下：

![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230626215529.png)

maven仓库搜索即可以看到自己的项目

![](https://raw.githubusercontent.com/jackyrwj/picb/master/20230626215938.png)



