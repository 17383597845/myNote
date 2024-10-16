bash


创建一个docker容器
`docker run --privileged -it -d -p 5003:5003 --name=arl --restart=always centos /usr/sbin/init`

### 解释：

- `--privileged`：给予容器特权模式，允许访问主机设备和资源。
- `-it`：以交互模式运行容器，`-i` 保持 STDIN 打开，`-t` 分配一个伪终端。
- `-d`：使容器以后台模式运行。
- `-p 5003:5003`：将主机的 5003 端口映射到容器的 5003 端口。
- `--name=arl`：将容器命名为 `arl`。
- `--restart=always`：在容器崩溃或 Docker 重启时，容器将自动重启。
- `centos`：基础镜像，使用的是官方的 CentOS 镜像。
- `/usr/sbin/init`：作为容器的启动命令，`init` 是启动 CentOS 的 init 系统。

常用命令：
```shell
#查看容器
docker ps
docker ps -a #查看所有容器包括已经关闭的
#启动一个容器，启动一个以前不存在的容器相当于创建一个新容器并启动
docker run -it --name my_container centos /bin/bash
#停止一个容器
docker stop my_container
# 进入容器
docker exec -it arl /bin/bash
#删除一个容器
docker rm <container_name_or_id>
docker rm my_container
```