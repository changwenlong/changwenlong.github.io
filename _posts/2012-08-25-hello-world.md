���ù���ֻ�м򵥼�������ȴ��ס���Ҵ���졣

## ����tomcatȨ��
��CATALINA_HOME/conf/tomcat-users.xml������Ȩ��(����Ȩ�޾���Ҫ):
```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="111111" roles="manager-gui,manager-script"/>
```

## pom.xml�����Ӳ��
- ����1��
	```xml
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
	```
	
- ����2��

	��sever���õ�maven��settings.xml��
	
	```xml
	<server>
	  <id>tomcat</id>
	  <username>tomcat</username>
	  <password>111111</password>
	</server>
	```

	pom.xml����ȫ�ֵ�server
	
	```xml
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
	```
## ��������
- tomcat7:deploy

	tomcat7:redeploy

## ��������
1. ��������tomcat�������в�������
2. tomcatȨ�����
3. maven����tomcat7-maven-plugin����
[�ο���ƪ����](http://www.51testing.com/html/94/488194-845177.html "��������")