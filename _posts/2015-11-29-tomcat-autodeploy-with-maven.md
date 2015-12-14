---
layout: post
title:  "Maven配置自动化部署项目到tomcat"
date:   2015-11-29 17:26:05
categories: maven tomcat
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

配置过程只有简单几步，但却难住了我大半天。

## 配置tomcat权限

在CATALINA_HOME/conf/tomcat-users.xml中增加权限(两个权限均需要):

	<role rolename="manager-gui"/>
	<role rolename="manager-script"/>
	<user username="tomcat" password="111111" roles="manager-gui,manager-script"/>


## pom.xml中增加插件

- 方法1：
	
		<plugin>
			<groupId>org.apache.tomcat.maven</groupId>
			<artifactId>tomcat7-maven-plugin</artifactId>
			<version>2.2</version>
			<configuration>
				<url>http://localhost:8080/manager/text</url>
				<server>tomcat</server>
				<username>tomcat</username>
				<password>111111</password>
				<path>/${project.build.finalName}</path>
			</configuration>
		</plugin>
	
	
- 方法2：

	将sever配置到maven的settings.xml中
	
		<server>
	 		<id>tomcat</id>
	  		<username>tomcat</username>
	 	 	<password>111111</password>
		</server>
	

	pom.xml引用全局的server
	
		<plugin>
			<groupId>org.apache.tomcat.maven</groupId>
			<artifactId>tomcat7-maven-plugin</artifactId>
			<version>2.2</version>
			<configuration>
				<url>http://localhost:8080/manager/text</url>
				<server>tomcat</server>
		      	<path>/${project.build.finalName}</path>
			</configuration>
		</plugin>
	
## 部署命令

- tomcat7:deploy

		tomcat7:redeploy

## 常见问题

1. 需先启动tomcat，在运行部署命令

2. tomcat权限添加

3. maven下载tomcat7-maven-plugin出错 [参考这篇博客](http://www.51testing.com/html/94/488194-845177.html "常见问题")