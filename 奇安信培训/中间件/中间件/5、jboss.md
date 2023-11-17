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


### 6、jboss热部署：指的是项目文件的热部署
在deployment-scanner标签内添加auto-deploy-exploded="true"
![[Pasted image 20231117141344.png]]
在stadalone/deployments中创建一个以.war结尾的文件夹，他会自动生成一个文件（文件名为你的文件夹名.deployed或.），如果在.war结尾的文件夹中创建文件他会自动部署文件。