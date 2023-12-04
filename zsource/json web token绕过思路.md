## JWT伪造

  这是一道`b00t2root`的一道web题，觉得很有意思，并且结合了加密的知识，所以记录一下。

  首先了解下JWT：

> JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。JWT常被用于前后端分离，可以和Restful API配合使用，常用于构建身份认证机制。

  JWT的数据格式分为三个部分： headers , payloads，signature(签名)，它们使用`.`点号分割。拿道题后看了一下cookie，发现是如下格式：

  
[![](https://delcoding.github.io/images/posts/other/18.png)](https://delcoding.github.io/images/posts/other/18.png)  

  

|   |   |
|---|---|
|1|eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6ImZhbHNlIn0.oe4qhTxvJB8nNAsFWJc7_m3UylVZzO3FwhkYuESAyUM|

  进行base64解密后发现：

  
[![](https://delcoding.github.io/images/posts/other/19.png)](https://delcoding.github.io/images/posts/other/19.png)  

  
  所以，我们的目的就是把`false`改成`true`，而且要通过`服务器的验证`，这点很重要，并不是直接把`false`改成`true`就万事大吉了。因为服务器收到token后会对token的`有效性`进行验证。

  验证方法：首先服务端会产生一个`key`，然后以这个`key`作为密钥，使用第一部分选择的加密方式（这里就是`HS256`），对第一部分和第二部分`拼接的结果`进行加密，然后把加密结果放到`第三部分`。

|   |   |
|---|---|
|1|服务器每次收到信息都会对它的前两部分进行加密，然后比对加密后的结果是否跟客户端传送过来的第三部分相同，如果相同则验证通过，否则失败。|

  因为加密算法我们已经知道了，如果我们只要再得到加密的`key`，我们就能伪造数据，并且通过服务器的检查。

  这里我使用了这个工具进行破解：[C语言版JWT破解工具](https://github.com/brendan-rius/c-jwt-cracker)，下载安装完毕后，直接进行破解，如图：

  
[![](https://delcoding.github.io/images/posts/other/20.png)](https://delcoding.github.io/images/posts/other/20.png)  

  
  所以，我的加密密钥就是：54l7y。然后我们去验证一下，这个网站可以提供验证服务：[https://jwt.io/](https://jwt.io/)。当我们使用破解出来的key时，我们能完美还原出原始数据，这证明我们的key是正确的。

  
[![](https://delcoding.github.io/images/posts/other/21.png)](https://delcoding.github.io/images/posts/other/21.png)  

  
  最后我们把`false`改成`true`，然后使用key进行加密，可以得到如下：

  
[![](https://delcoding.github.io/images/posts/other/22.png)](https://delcoding.github.io/images/posts/other/22.png)  

  
  然后我们拿着这个token刷新一下：

  
[![](https://delcoding.github.io/images/posts/other/23.png)](https://delcoding.github.io/images/posts/other/23.png)  

  
  可以看到flag已经出来了