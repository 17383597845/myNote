1、解决yum命令报错：https://blog.csdn.net/lxcw_sir/article/details/140185068
2、安装：https://www.cnblogs.com/ding2016/p/16973626.html
```shell
cp ./ca.crt /etc/pki/ca-trust/source/anchors/  

ln -s /etc/pki/ca-trust/source/anchors/mitmproxy-ca-cert.cer  /etc/ssl/certs/mitmproxy-ca-cert.cer

update-ca-trust
```

## 2、docker和docker-compose安装以及镜像配置
docker：
```
安装一些依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2 wget
下载repo文件
wget -O /etc/yum.repos.d/docker-ce.repo
https://download.docker.com/linux/centos/docker-ce.repo
把软件仓库地址替换为 TUNA:
sudo sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+'
/etc/yum.repos.d/docker-ce.repo
安装
sudo yum makecache fast
sudo yum install docker-ce
```
镜像：
```
https://blog.csdn.net/leviopku/article/details/135628547
```
docker-compose：
```shell
#docker-compose可以自己到官网下载，改名，放到指定文件中即可
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```

## 3、centos配置为代理服务器：
在 CentOS 7 上配置 Squid 以允许特定 IP 使用代理的步骤如下：

### 1. 安装 Squid

如果还未安装 Squid，可以使用以下命令安装：
```bash
sudo yum install squid -y
```

### 2. 编辑 Squid 配置文件

打开 Squid 的配置文件：
```bash
sudo nano /etc/squid/squid.conf
```

### 3. 添加允许的 IP 地址

在配置文件中，找到或添加 ACL（访问控制列表）部分。你可以在文件的底部添加以下行（替换 `YOUR_IP_ADDRESS` 为你想要允许的实际 IP 地址）：
```plaintext
# 允许特定 IP 地址访问
acl allowed_ip src YOUR_IP_ADDRESS
http_access allow allowed_ip

# 拒绝其他所有访问
http_access deny all
```

### 4. 保存并退出

在 `nano` 中，按 `Ctrl + X`，然后按 `Y` 确认保存更改。

### 5. 启动和重启 Squid 服务

启动 Squid 服务并设置为开机自启动：
```bash
sudo systemctl start squid
sudo systemctl enable squid
```

如果已经在运行，重启 Squid 以应用更改：
```bash
sudo systemctl restart squid
```

### 6. 验证配置

从允许的 IP 地址测试 Squid 代理：
```bash
curl -x http://your_squid_server:3128 http://example.com
```

### 7. 检查日志

如果遇到问题，可以查看 Squid 的日志：
```bash
sudo tail -f /var/log/squid/access.log
sudo tail -f /var/log/squid/cache.log
```

如果需要更多帮助或遇到问题，请告诉我！
