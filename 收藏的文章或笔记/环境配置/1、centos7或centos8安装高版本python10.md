1、解决yum命令报错：https://blog.csdn.net/lxcw_sir/article/details/140185068
2、安装：https://www.cnblogs.com/ding2016/p/16973626.html
```shell
cp ./ca.crt /etc/pki/ca-trust/source/anchors/  

ln -s /etc/pki/ca-trust/source/anchors/mitmproxy-ca-cert.cer  /etc/ssl/certs/mitmproxy-ca-cert.cer

update-ca-trust
```