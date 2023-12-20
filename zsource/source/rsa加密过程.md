**目录：**

[一、什么是RSA加密算法：](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fjavaforall.cn%2F%23main-toc&source=article&objectId=1812698)

[二、RSA加密过程：](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fjavaforall.cn%2F%23%25E4%25BA%258C%25E3%2580%2581RSA%25E5%258A%25A0%25E5%25AF%2586%25E8%25BF%2587%25E7%25A8%258B%25EF%25BC%259A&source=article&objectId=1812698)

[三、RAS解密过程：](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fjavaforall.cn%2F%23%25E4%25B8%2589%25E3%2580%2581RAS%25E8%25A7%25A3%25E5%25AF%2586%25E8%25BF%2587%25E7%25A8%258B%25EF%25BC%259A&source=article&objectId=1812698)

[四、生成密钥对：](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fjavaforall.cn%2F%23%25E5%259B%259B%25E3%2580%2581%25E7%2594%259F%25E6%2588%2590%25E5%25AF%2586%25E9%2592%25A5%25E5%25AF%25B9%25EF%25BC%259A&source=article&objectId=1812698)

[五、实践：](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fjavaforall.cn%2F%23%25E4%25BA%2594%25E3%2580%2581%25E5%25AE%259E%25E8%25B7%25B5%25EF%25BC%259A&source=article&objectId=1812698)

[六、Java进行 RSA 加解密时不得不考虑到的那些事儿：](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fjavaforall.cn%2F%23%25E5%2585%25AD%25E3%2580%2581Java%25E8%25BF%259B%25E8%25A1%258C%2520RSA%2520%25E5%258A%25A0%25E8%25A7%25A3%25E5%25AF%2586%25E6%2597%25B6%25E4%25B8%258D%25E5%25BE%2597%25E4%25B8%258D%25E8%2580%2583%25E8%2599%2591%25E5%2588%25B0%25E7%259A%2584%25E9%2582%25A3%25E4%25BA%259B%25E4%25BA%258B%25E5%2584%25BF%25EF%25BC%259A&source=article&objectId=1812698)

---

## 一、什么是RSA加密算法：

RSA加密算法是一种非对称加密算法，所谓非对称，就是指该算法加密和解密使用不同的密钥，即使用加密密钥进行加密、解密密钥进行解密。在RAS算法中，加密密钥（即公开密钥）PK是公开信息，而解密密钥（即秘密密钥）SK是需要保密的。加密算法E和解密算法D也都是公开的。虽然解密密钥SK是由公开密钥PK决定的，由于无法计算出大数n的欧拉函数phi(N)，所以不能根据PK计算出SK。

也就是说，对极大整数做因数分解的难度决定了RSA算法的可靠性。理论上，只要其钥匙的长度n足够长，用RSA加密的信息实际上是不能被解破的。

RSA算法通常是先生成一对RSA密钥，其中之一是保密密钥，由用户保存；另一个为公开密钥，可对外公开。为提高保密强度，RSA密钥至少为500位长，一般推荐使用1024位。这就使加密的计算量很大。为减少计算量，在传送信息时，常采用传统加密方法与公开密钥加密方法相结合的方式，即信息采用改进的DES或IDEA密钥加密，然后使用RSA密钥加密对话密钥和信息摘要。对方收到信息后，用不同的密钥解密并可核对信息摘要。

**RSA密钥长度随着保密级别提高，增加很快。**下表列出了对同一安全级别所对应的密钥长度。

|保密级别|对称密钥长度（bit）|RSA密钥长度（bit）|ECC密钥长度（bit）|保密年限|
|---|---|---|---|---|
|80|80|1024|160|2010|
|112|112|2048|224|2030|
|128|128|3072|256|2040|
|192|192|7680|384|2080|
|256|256|15360|512|2120|

## 二、RSA加密过程：

RSA的加密过程可以使用一个通式来表达：
也就是说RSA加密是对明文的E次方后除以N后求余数的过程。从通式可知，只要知道E和N任何人都可以进行RSA加密了，所以说E、N是RSA加密的密钥，也就是说**E和N的组合就是公钥**，我们用(E,N)来表示公钥：
不过E和N不并不是随便什么数都可以的，它们都是经过严格的数学计算得出的，关于E和N拥有什么样的要求及其特性后面会讲到。E是加密（Encryption）的首字母，N是数字（Number）的首字母。

## 三、RAS解密过程：

RSA的解密同样可以使用一个通式来表达：
也就是说对密文进行D次方后除以N的余数就是明文，这就是RSA解密过程。知道D和N就能进行解密密文了，所以D和N的组合就是私钥：
从上述可以看出RSA的加密方式和解密方式是相同的，加密是求“E次方的mod N”;解密是求“D次方的mod N”。此处D是解密（Decryption）的首字母；N是数字（Number）的首字母。
小结下：

|公钥|（E，N）|
|---|---|
|私钥|（D，N）|
|密钥对|（E，D，N）|
|加密|密文＝明文EmodN密文＝明文EmodN|
|解密|明文＝密文DmodN明文＝密文DmodN|

## 四、生成密钥对：

既然公钥是（E，N），私钥是（D，N），所以密钥对即为（E，D，N），但密钥对是怎样生成的？步骤如下：

- 求N
- 求L（L为中间过程的中间数）
- 求E
- 求D

**4.1 求N：**

准备两个互质数p，q。这两个数不能太小，太小则会容易破解，将p乘以q就是N。如果互质数p和q足够大，那么根据目前的计算机技术和其他工具，至今也没能从N分解出p和q。换句话说，只要密钥长度N足够大（一般1024足矣），基本上不可能从公钥信息推出私钥信息。

> N = p * q

**4.2 求L：**

L 是 p－1 和 q－1的最小公倍数，可用如下表达式表示

> L = lcm（p－1，q－1）

**4.3 求E：**

E必须满足两个条件：E是一个比1大比L小的数，E和L的最大公约数为1；

用gcd(X,Y)来表示X，Y的最大公约数则E条件如下：

> 1 < E < L gcd（E，L）=1

之所以需要E和L的最大公约数为1，是为了保证一定存在解密时需要使用的数D。现在我们已经求出了E和N也就是说我们已经生成了密钥对中的公钥了。

**4.4 求D：**

数D是由数E计算出来的，数D必须保证足够大。D、E和L之间必须满足以下关系：

> 1 < D < L E＊D mod L ＝ 1

只要D满足上述2个条件，则通过E和N进行加密的密文就可以用D和N进行解密。简单地说条件2是为了保证密文解密后的数据就是明文。

现在私钥自然也已经生成了，密钥对也就自然生成了。

小结：

|求N|N＝ p ＊ q ；p，q为质数|
|---|---|
|求L|L＝lcm（p－1，q－1） ；L为p－1、q－1的最小公倍数|
|求E|1 < E < L，gcd（E，L）=1；E，L最大公约数为1（E和L互质）|
|求D|1 < D < L，E＊D mod L ＝ 1|

## 五、实践：

为了计算方便，p q 的值取小一旦，假设：p ＝ 17，q ＝ 19，

则：

（1）求N：N ＝ p ＊ q ＝ 323；

（2）求L：L ＝ lcm（p－1， q－1）＝ lcm(16，18） ＝ 144，144为16和18对最小公倍数；

（3）求E：1 < E < L ，gcd（E，L）=1，即1 < E < 144，gcd（E，144） ＝ 1，E和144互为质数，E = 5显然满足上述2个条件，故E ＝ 5，此时**公钥= (E，N）＝（5，323）**；

（4）求D：求D也必须满足2个条件：1 < D < L，E＊D mod L ＝ 1，即1 < D < 144，5 ＊ D mod 144 ＝ 1，显然当D＝ 29 时满足上述两个条件。1 < 29 < 144，5＊29 mod 144 ＝ 145 mod 144 ＝ 1，此时**私钥＝（D，N）＝（29，323）**；

（5）加密：准备的明文必须是小于N的数，因为加密或者解密都要 mod N，其结果必须小于N。

假设明文 ＝ 123，则 密文＝（123的5次方）mod 323=225

（6）解密：明文＝（225的29次方）mod 323 =123，所以解密后的明文为123。

## 六、Java进行 RSA 加解密时不得不考虑到的那些事儿：

**1、质数的选择：**

首先要使用概率算法来验证随机产生的大的整数是否是质数，这样的算法比较快而且可以消除掉大多数非质数。假如有一个数通过了这个测试的话，那么要使用一个精确的测试来保证它的确是一个质数。除此之外这样找到的p和q还要满足一定的要求，首先它们不能太靠近，此外p-1或q-1的因子不能太小，否则的话N也可以被很快地分解。

寻找质数的算法不能给攻击者任何信息，比如这些质数是怎样找到的？尤其产生随机数的软件必须非常好。要求是随机和不可预测。这两个要求并不相同。一个随机过程可能可以产生一个不相关的数的系列，但假如有人能够预测出（或部分地预测出）这个系列的话，那么它就已经不可靠了。比如有一些非常好的随机数算法，但它们都已经被发表，因此它们不能被使用，因为假如一个攻击者可以猜出p和q一半的位的话，那么他们就已经可以轻而易举地推算出另一半。

**2、RSA加密算法的缺点：**

（1）产生密钥很麻烦，受到质数产生技术的限制，因而难以做到一次一密；

（2）运算速度慢：由于进行的都是大数计算，使得RSA最快的情况也比DES慢上好几倍，无论是软件还是硬件实现，速度一直是RSA的缺陷，所以一般只用于少量数据的加密。RSA的速度是对应同样安全级别的对称[密码算法](https://cloud.tencent.com/developer/tools/blog-entry?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%25AF%2586%25E7%25A0%2581%25E7%25AE%2597%25E6%25B3%2595&source=article&objectId=1812698)的1/1000左右。

一般使用对称算法来加密数据，然后用RSA来加密对称密钥，然后将用RSA加密的对称密钥和用对称算法加密的消息发送出去。这样一来对随机数的要求就更高了，尤其对产生对称密码的要求非常高，因为否则的话可以越过RSA来直接攻击对称密码。

**3、加密的系统不要具备解密的功能，否则 RSA 可能不太合适：**

公钥加密，私钥解密。加密的系统和解密的系统分开部署，加密的系统不应该同时具备解密的功能，这样即使黑客攻破了加密系统，他拿到的也只是一堆无法破解的密文数据。否则的话，你就要考虑你的场景是否有必要用 RSA 了。

**4、可以通过修改生成密钥的长度来调整密文长度：**

生成密文的长度等于密钥长度。密钥长度越大，生成密文的长度也就越大，加密的速度也就越慢，而密文也就越难被破解掉。我们必须通过定义密钥的长度在”安全”和”加解密效率”之间做出一个平衡的选择。

**5、生成密文的长度和明文长度无关，但明文长度不能超过密钥长度：**

不管明文长度是多少，RSA 生成的密文长度总是固定的。但是明文长度不能超过密钥长度。

比如 Java 默认的 RSA 加密实现不允许明文长度超过密钥长度减去 11(单位是字节，也就是 byte)。也就是说，如果我们定义的密钥(我们可以通过 java.security.KeyPairGenerator.initialize(int keysize) 来定义密钥长度)长度为 1024(单位是位，也就是 bit)，生成的密钥长度就是 1024位 / 8位/字节 = 128字节，那么我们需要加密的明文长度不能超过 128字节 -11 字节 = 117字节。也就是说，我们最大能将 117 字节长度的明文进行加密，否则会出问题(抛诸如 javax.crypto.IllegalBlockSizeException: Data must not be longer than 53 bytes 的异常)。

**6、可以通过调整算法提供者来减小密文长度：**

Java 默认的 RSA 实现 “RSA/None/PKCS1Padding” 要求最小密钥长度为 512 位(否则会报 java.security.InvalidParameterException: RSA keys must be at least 512 bits long 异常)，也就是说生成的密钥、密文长度最小为 64 个字节。如果你还嫌大，可以通过调整算法提供者来减小密文长度：

```javascript
Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider());
final KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA", "BC");
keyGen.initialize(128);
```

复制

如此这般得到的密文长度为 128 位(16 个字节)。但是这么干之前请先回顾一下上面第 4 点所述。

**7、 byte[].toString() 返回的实际上是内存地址，不是将数组的实际内容转换为 String：**

Java 中数组的 toString() 方法返回的并非数组内容，它返回的实际上是数组存储元素的类型以及数组在内存的位置的一个标识。如果我们对密钥以 byte[].toString() 进行持久化存储或者和其他一些字符串如 json 传输，那么密钥的解密者得到的将只是一串毫无意义的字符，当他解码的时候很可能会遇到 “javax.crypto.BadPaddingException” 异常。

**8、字符串用以保存文本信息，字节数组用以保存二进制数据：**

java.lang.String 保存明文，byte 数组保存二进制密文，在 java.lang.String 和 byte[] 之间不应该具备互相转换。如果你确实必须得使用 java.lang.String 来持有这些二进制数据的话，最安全的方式是使用 Base64(推荐 Apache 的 commons-codec 库的 org.apache.commons.codec.binary.Base64)：

```javascript
      // use String to hold cipher binary data
      Base64 base64 = new Base64(); 
      String cipherTextBase64 = base64.encodeToString(cipherText);
      
      // get cipher binary data back from String
      byte[] cipherTextArray = base64.decode(cipherTextBase64);
```

复制

**9、每次生成的密文都不一致证明你选用的加密算法很安全：**

一个优秀的加密必须每次生成的密文都不一致，即使每次你的明文一样、使用同一个公钥。因为这样才能把明文信息更安全地隐藏起来。

Java 默认的 RSA 实现是 “RSA/None/PKCS1Padding”(比如 Cipher cipher = Cipher.getInstance(“RSA”);句，这个 Cipher 生成的密文总是不一致的)，Bouncy Castle 的默认 RSA 实现是 “RSA/None/NoPadding”。

为什么 Java 默认的 RSA 实现每次生成的密文都不一致呢，即使每次使用同一个明文、同一个公钥？这是因为 RSA 的 PKCS #1 padding 方案在加密前对明文信息进行了随机数填充。

你可以使用以下办法让同一个明文、同一个公钥每次生成同一个密文，但是你必须意识到你这么做付出的代价是什么。比如，你可能使用 RSA 来加密传输，但是由于你的同一明文每次生成的同一密文，攻击者能够据此识别到同一个信息都是何时被发送。

```javascript
Security.addProvider(new org.bouncycastle.jce.provider.BouncyCastleProvider());
final Cipher cipher = Cipher.getInstance("RSA/None/NoPadding", "BC");
```

复制

**10、Cipher 是有状态的，而且是线程不安全的：**

javax.crypto.Cipher 是有状态的，不要把 Cipher 当做一个静态变量，除非你的程序是单线程的，也就是说你能够保证同一时刻只有一个线程在调用 Cipher。否则可能会遇到 java.lang.ArrayIndexOutOfBoundsException: too much data for RSA block 异常。遇见这个异常，你需要先确定你给 Cipher 加密的明文(或者需要解密的密文)是否过长；排除掉明文(或者密文)过长的情况，你需要考虑是不是你的 Cipher 线程不安全了。

**11、对于加密后的数据，如果要打印出来，必须以十六进制或者BCD码的形式打印：**

不能new String（byte[]）后，再从这个String里getbytes（），也不要用base64，不然会破坏原数据。

比如：

```javascript
		byte[ ] bytes=new byte[ ]{108, -56, 111, 34, -67};
		byte[ ] newBytes=new String(bytes).getBytes();
		StringBuffer sb=new StringBuffer();
		for(int i=0; i<newBytes.length; i++){
			sb.append(newBytes[i]+"|");
		}
		System.out.println(sb.toString());
```

复制

将一个byte数组new String后再getbytes出来后，看看运行结果：

最后一个byte由-67变为了63。