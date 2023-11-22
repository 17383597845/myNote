dhcp配置（在交换机上，dhcp工作在二层）：
基于接口的配置
```bash
dhcp enble
int g0/0/1.10 #以这个接口的地址作为地址池中的资源，并且网关就是它
dhcp select interface
dhcp server dns-list 114.114.114.114
dis this
```

基于地址池的配置
```
ip pool xxx
netbios-type
network 192.168.1.0 mask 24
gateway-list 192.168.1.1
dns-list 8.8.8.8
int g0/0/1.10
dhcp select global
```

nat地址转换：
以下四种都叫做s.nat
	![[Pasted image 20231110163105.png]]
	![[Pasted image 20231110163121.png]]

	![[Pasted image 20231110163526.png]]

重点napt和easyip
![[Pasted image 20231110163740.png]]
![[Pasted image 20231110164634.png]]
```bash
#配置地址池只有122.1.2.1这一个地址
nat address-group 1 122.1.2.1 122.1.2.1

```

![[Pasted image 20231110164448.png]]

![[Pasted image 20231110164558.png]]

nat server:
d.nat:
![[Pasted image 20231110164903.png]]
![[Pasted image 20231110164959.png]]

![[Pasted image 20231110165102.png]]



考试时：
检查接口IP：
dis ip int brief
检查链路聚合做成功没：
![[Pasted image 20231110172613.png]]
检查vlan配置：
![[Pasted image 20231110172752.png]]

检查vlan通信的信息
![[Pasted image 20231110173043.png]]

检查静态路由
dis ip routing-table



ospf邻居关系建立成功，下面表的最后一栏变成full得分
![[Pasted image 20231110173923.png]]

路由推送或引用（到对面看路由）：

![[Pasted image 20231110174036.png]]

dhcp：
![[Pasted image 20231110174420.png]]

除了上面的配置还需要pc机器上的ip截图


检查nat：
![[Pasted image 20231110174717.png]]