---
layout: post
title:  "使用Maven构建Java web项目"
date:   2015-11-29
categories: maven java
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 本机环境
> win7 64bit
> 
> Eclipse Java EE IDE :  Luna Release (4.4.0)
> 
> apache-maven-3.2.3

## 创建Maven项目

选择maven-archetype-webapp

![选择archetype]({{"/static/imgs/create-javaweb-project-with-maven-archetpe.png"}})

## 项目依赖jdk1.7

修改Java Bulid Path和 Java Compiler

## 配置Project Facets

右键项目-->properties-->Project Facets

- java 选择1.7-->Apply

	![java选择1.7]({{"/static/imgs/create-javaweb-project-with-maven-apply.png"}}) 

- Dynamic Web Module 选择3.0 

	![Dynamic Web Module 选择3.0]({{"/static/imgs/create-javaweb-project-with-maven-module.png"}})

报错：Cannot change version of project facet Dynamic web 3.0

- 打开项目目录，   
.settings目录下的org.eclipse.wst.common.project.facet.core.xml文件

	> 修改前
	
		<?xml version="1.0" encoding="UTF-8"?>
		<faceted-project>
		  <fixed facet="wst.jsdt.web"/>
		  <installed facet="jst.web" version="2.3"/>
		  <installed facet="wst.jsdt.web" version="1.0"/>
		  <installed facet="java" version="1.7"/>
		</faceted-project>
	

	> 修改后
	
		<?xml version="1.0" encoding="UTF-8"?>
		<faceted-project>
		  <fixed facet="wst.jsdt.web"/>
		  <installed facet="jst.web" version="3.0"/>
		  <installed facet="wst.jsdt.web" version="1.0"/>
		  <installed facet="java" version="1.7"/>
		</faceted-project>
	

- 刷新项目 此时Dynamic Web Module为3.0

- 修改web.xml

	> 修改前
	
		<!DOCTYPE web-app PUBLIC
		 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
		 "http://java.sun.com/dtd/web-app_2_3.dtd" >
		
		<web-app>
		  <display-name>Archetype Created Web Application</display-name>
		</web-app>
	

	> 修改后
	
		<?xml version="1.0" encoding="UTF-8"?>
		<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
			xmlns="http://java.sun.com/xml/ns/javaee" 
			xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" 
			id="WebApp_ID" version="3.0">
		  <display-name>Archetype Created Web Application</display-name>
		</web-app>
	



	
