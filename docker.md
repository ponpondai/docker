---
tags: Container
---
# Docker
## Docker概念圖
![](https://i.imgur.com/DLpqRA0.png)
- Docker 使用的 Linux 核心模組功能有下列各項：
1. Namespace - 用來隔離不同 Container 的執行空間
2. Cgroup - 用來分配硬體資源
3. chroot - 針對正在運作的軟體行程和它的子行程，改變它的根目錄
4. Bridge (Network) - 建立虛擬網路, 連接不同 Container 
5. Overlay2(chroot) - 用來建立不同 Container 的檔案系統
## Docker  運作架構
![](https://i.imgur.com/GmU7xuH.png)


##  Docker 命令運作圖
![](https://i.imgur.com/8PSmHM0.png)
## 實作 Docker
- 安裝 Docker CE
> $ sudo  apt  install  docker.io
- 將 user 帳號加入 docker 群組後, 就不需使用 sudo 命令執行 docker
> $ sudo  usermod  -aG  docker user
- 顯示 Docker 版本
>$ docker info
```
 Server Version: 19.03.8
 Storage Driver: overlay2
 ......
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
```
- 沒有任何 Container 執行時, 只有 dockerd 及 containerd 這二個 Daemon 在運作
> $ ps aux | grep -v grep | grep containerd
```
root         604  0.3  1.1 969256 46788 ?        Ssl  14:48   0:06 /usr/bin/containerd
root        1034  0.1  2.2 1008892 89944 ?       Ssl  14:52   0:03 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```
- 檢視 Docker 運作資訊 - 執行 Container

> $ docker run --rm -d alpine sleep 120

>  $ ps aux | grep -v grep | grep containerd
```
root         604  0.3  1.1 969256 46788 ?        Ssl  14:48   0:09 /usr/bin/containerd

root        1034  0.1  2.2 1008892 90176 ?       Ssl  14:52   0:03 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

root        1724  0.0  0.1 108572  5144 ?        Sl   15:41   0:00 containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/370e60c26bad84ada53e6ee55ece894d22a31aabb160de1ab9f26994e89c7a12 -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
```
- 搜尋與下載 Docker Image
> $ docker pull busybox
```
Using default tag: latest
latest: Pulling from library/busybox
7520415ce762: Pull complete 
Digest: sha256:32f093055929dbc23dec4d03e09dfe971f5973a9ca5cf059cbfb644c206aa83f
Status: Downloaded newer image for busybox:latest
```
> $ docker images
```
REPOSITORY   TAG        IMAGE ID      CREATED          SIZE
busybox      latest     1c35c4412082  5 days ago       1.22MB
```
- 管理軟體貨櫃 (Container) 主機
>$ docker run --name b1 -it busybox /bin/sh

>/ # exit

>$ docker ps -a
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
54ab5a1a4690        busybox             "/bin/sh"           5 minutes ago       Exited (0) 9 seconds ago                       b1
```
>$docker start b1

>$docker exec -it b1 sh      #只能本機連接

>/#exit

>$docker stop b1

> $docker rm b1
## 撰寫Dockerfile
- 使用ADD讓本機壓縮檔在image裡面自動解壓縮
``` 
FROM ubuntu:18.04
ADD julia-1.4.1-linux-x86_64.tar.gz /opt
RUN ln -s /opt/julia-1.4.1/bin/julia /usr/bin/julia
```
- 撰寫 alpine.base Dockerfile
```
FROM alpine:3.11.6
RUN apk update && apk upgrade && apk add --no-cache nano sudo wget curl \
    tree elinks bash shadow procps util-linux coreutils binutils findutils grep && \
    wget https://busybox.net/downloads/binaries/1.28.1-defconfig-multiarch/busybox-x86_64 && \
    chmod +x busybox-x86_64 && mv busybox-x86_64 bin/busybox1.28 && \
    mkdir -p /opt/www && echo "let me go" > /opt/www/index.html
CMD ["/bin/bash"
```
- 建立 alpine.base image
>$ docker build --no-cache  -t alpine.base base/
### 執行 alpine.base image 內定命令
> $ docker run --rm -it alpine.base

> bash-5.0# /bin/busybox1.28 | head -n 1
```
BusyBox v1.28.1 (2018-02-15 14:34:02 CET) multi-call binary.
```
>bash-5.0# /bin/busybox | head -n 1
```
BusyBox v1.30.1 (2019-06-12 17:51:55 UTC) multi-call binary.
```
>bash-5.0# busybox httpd -h /opt/www
```
httpd: applet not found
```
>bash-5.0# busybox1.28 httpd -h /opt/www

>bash-5.0# curl http://localhost
```
let me go
```
>bash-5.0# exit
- 自製 Alpine OpenSSH Server 的 Docker Image
- ENTRYPOINT 所指定的執行命令, 在建立 Container 時強制執行此命令並且不可被其它命令取代
```
FROM alpine.base
RUN apk update && \
    apk add --no-cache openssh-server tzdata && \
    # 設定時區
    cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
    ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
    echo -e 'Welcome to Alpine 3.11.6\n' > /etc/motd && \ 
    # 建立管理者帳號 bigred   
    adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && echo '%wheel ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers && \
    echo -e "bigred\nbigred\n" | passwd bigred &>/dev/null && [ "$?" == "0" ] && echo "bigred ok"
 
EXPOSE 22
 
ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"] 
```
## Docker Image 備份與還原
- 備份 Image 
> $ docker save alpine.plus > alpine.plus.tar
- 還原 Image
>$ docker load < alpine.plus.tar

>$ docker history alpine.plus
- 還原的 image 的目錄架構, 與原先 Image目錄架構一樣
```
IMAGE              CREATED           CREATED BY                                      SIZE                COMMENT
eb14bc497f7c        5 hours ago         /bin/sh -c #(nop)  CMD ["-D"]                   0B                  
1e05936fd376        5 hours ago         /bin/sh -c #(nop)  ENTRYPOINT ["/usr/sbin/ss…   0B                  
c5c25dc4a60c        5 hours ago         /bin/sh -c #(nop)  EXPOSE 22                    0B                  
6a42f8c6183f        5 hours ago         /bin/sh -c apk update &&     apk add --no-ca…   4MB                 
37dfd3d3318e        6 hours ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
86c8cd04a262        6 hours ago         /bin/sh -c apk update && apk upgrade && apk …   36.4MB              
f70734b6a266        2 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B                  
<missing>           2 weeks ago         /bin/sh -c #(nop) ADD file:b91adb67b670d3a6f…   5.61MB
```
## yaml template
- database
```yaml=
version: '3.7'
services:
  sqldb:
    image: mariadb
    container_name: sqldb
    hostname: sqldb
    volumes:
      - ./sql:/docker-entrypoint-initdb.d:ro
    environment:
      LANG: C.UTF-8
      MYSQL_DATABASE: "sqldb"
      MYSQL_ROOT_PASSWORD: "root"
```

