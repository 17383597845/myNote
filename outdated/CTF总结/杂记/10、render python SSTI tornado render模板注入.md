**原理**  
tornado render是python中的一个渲染函数，也就是一种模板，通过调用的参数不同，生成不同的网页，如果用户对render内容可控，不仅可以注入XSS代码，而且还可以通过{{}}进行传递变量和执行简单的表达式。

源码：[[render]]

由上面可知:render是一个类似模板的东西，可以使用不同的参数来访问网页
在tornado模板中，存在一些可以访问的快速对象，例如
{{ escape(handler.settings[“cookie”]) }}

这两个{{}}和这个字典对象也许大家就看出来了，没错就是这个handler.settings对象
handler 指向RequestHandler

而RequestHandler.settings又指向self.application.settings

列题：
拿到cookie
![[Pasted image 20231209154227.png]]