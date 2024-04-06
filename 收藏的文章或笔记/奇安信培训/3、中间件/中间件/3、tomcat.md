#### 1、环境变量配置
JAVA_HOME   CATALINA_HOME  JAVAPATH  TOMCATPATH
#### 2、更改默认端口
server.xml：在Connector标签中修改端口
```xml
  <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```
#### 3、修改项目根目录
server.xml:
在最后</host>标签前：
```xml
			   <!--更改默认根目录，注意下面的斜杠是正斜杠-->
			<Context path='' docBase='E:/dev/apache-tomcat-10.0.22/www'debug='0'/>
```
#### 4、修改默认页面
web.xml：这个配置文件有4000多行，直接翻到最后,有一个welcome-file-list标签：
```xml
    <welcome-file-list>
	    <welcome-file>one.html</welcome-file>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
```
#### 5、配置管理后台的用户名密码（manager app）
tomcat-user.xml:就在tomcat-users这个大标签下添加以下两行
```xml
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
              
<role rolename="manager-gui"/>
<user username="tomcat" password="Hh12345@" roles="manager-gui"/>	
```
#### 6、对war包热部署功能的关闭：
server.xml
	host标签中有两个true选项，都改成false
```xml
      <Host name="localhost"  appBase="webapps"
            unpackWARs="false" autoDeploy="false">
```