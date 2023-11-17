### 1、目录结构
![[Pasted image 20231117140949.png]]
### 2、启动与关闭jboos
运行standalone.bat启动jboss，关闭dos界面即关闭jboss
### 3、修改访问ip的限制
standalone下的configuration目录下有一个standalone.xml文件：
原本只允许本地访问，按下面的修改，使得其他主机可以访问
![[Pasted image 20231117142313.png]]
### 4、修改端口

### 5、默认根目录和默认主页
创建一个WEB-INF目录
![[Pasted image 20231117143810.png]]
在web-inf中创建一个名为jboss-web.xml文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<jboss-web>
;将当前目录作为根目录
	<context-root>/</context-root>
</jboss-web>

```
jboss中不能更改默认页面文件名，他的默认页面是固定的就是index.html

在standalone下的configuration目录下有一个standalone.xml文件中修改默认欢迎界面：
注意这个字段的修改是一次性的，关闭后就不再能够开启默认欢迎页面，修改为false的字段会在配置文件中消失
![[Pasted image 20231117144303.png]]
### 6、jboss热部署：指的是项目文件的热部署
在deployment-scanner标签内添加auto-deploy-exploded="true"
![[Pasted image 20231117141344.png]]
在stadalone/deployments中创建一个以.war结尾的文件夹，他会自动生成一个文件（文件名为你的文件夹名.deployed或.undeployed），如果在.war结尾的文件夹中创建文件他会自动部署文件。