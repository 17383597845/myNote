### 一、配置
#### 1、启动mysql服务器
可以使用phpstudy启动，也可以使用原本就有的mysql服务
2、驱动配置
在php.ini 中找到相关配置（搜索pdo或mysqi）

### 二、连接mysql
### 1、mysqli
	特点：既支持面向过程又支持面向对象，而且提供了预处理

```php
<?php
/**
*mysqli连接数据库
*连接失败：
*端口？是否是3306
*mysql的服务是否正常启动
*用户名，密码，数据库名是否正确
*/
//配置主机名，用户名，密码，数据库名
$host = '192.168.240.133';
$username = 'root';
$password = 'Admin@123';
$dbname = 'php';
$conn = new Mysqli($host,$username,$password,$dbname);
if($conn->connect_error){//无法抛出异常，只能使用这种方法
    die('连接失败' . $conn->connect_error);
}
$sql = 'select * from stu';
$res = $conn->query($sql);
//结果集中有多少条数据（面向过程风格）
echo mysqli_num_rows($res);
//查询具体数据（面向对象的风格）
var_dump(res->fetch_array(MYSQLI_ASSOC));

```
mysqli中的相关方法
```php
	mysqli_num_fields : 返回结果集中的字段数
	mysqli_num_rows : 返回结果集中行的数量
	fetch_all() :
	fetch_array(MYSQLI_ASSOC) :
```
### 2、pdo
特点：只支持面向对象的风格
```php
<?php
/**

 * 异常处理，pdo可以抛出异常，mysqli不能抛出异常

 */
 $dsn = 'mysql:host=192.168.240.133;dbname=php';//指定数据库类型，主机名（数据库类型和主机名要用：连接），和数据库名
 $username = 'root';
 $password = 'Admin@123';
 try{
    $pdo = new PDO($dsn,$username,$password);
 }catch(PDOException $e){
    die('数据库连接失败' . $e->getMessage());
 }
 var_dump($pdo);

```
![[Pasted image 20231122100445.png]]

#### 4、预处理
PDOstatement中的方法；

参数占位：
```php
$sql = 'insert into contactInfo(name,address,phone) values(:name,:address,:phone)';
$pdostat = $dbh->prepare($sql);
//绑定参数到占位符
$pdostat->bindParam(':name',$name);
$pdostat->bindParam(":address",$address);
$pdostat0->bindParam(":phone",$phone);
//给参数赋值
$name = 'tom';
$address='xxxx';
$phone = '1234xxxx';
$pdostat->execute();

```
？占位：
```php
$sql = 'insert into contactInfo(name,address,phone) values(?,?,?)';
$pdostat = $dbh->prepare($sql);
//绑定参数到占位符
$pdostat->bindParam(1,$name,PDO::PARAM_STR);
$pdostat->bindParam(2,$address,PDO::PARAM_STR);
$pdostat0->bindParam(3,$phone,PDO::PARAM_STR);
//给参数赋值
$name = 'tom';
$address='xxxx';
$phone = '1234xxxx';
$pdostat->execute();

```
快捷语法：
![[Pasted image 20231122101730.png]]
```php


```