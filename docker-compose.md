# Docker Compose

## 安裝 Docker Compose
>$ sudo apt update; sudo apt install docker-compose -y; sudo reboot

顯示 docker-compose 版本
>$ docker-compose version

docker-compose version 1.25.0, build unknown
docker-py version: 4.1.0
CPython version: 3.8.5
OpenSSL version: OpenSSL 1.1.1f  31 Mar 2020

## 確認Docker Compose跟Docker版本是否符合
![](https://i.imgur.com/98Qx72t.png)



## 顯示 docker-compose 版本
>$ docker-compose version

docker-compose version 1.25.0, build unknown
docker-py version: 4.1.0
CPython version: 3.8.5
OpenSSL version: OpenSSL 1.1.1f  31 Mar 2020

## docker compose 常用執行命令
- 用yaml檔建立多個container
>$ docker-compose -f #檔名.yaml up -d
- 觀察yaml檔啟動的container
>$ docker-compose -f myapp.yml ps
- 用yaml檔刪除多個container
>$ docker-compose -f myapp.yml down

## 檢視docker網路
### 執行docker compose時會建立一個網路(名稱為在所在目錄)
!!!如果在同一個資料夾裡面compose第二次時，會在產生網路覆蓋掉舊的網路，但舊的container還是會有網路
>$ docker network list

NETWORK ID      NAME                DRIVER          SCOPE
0c5c9d2d38b3    bridge              bridge          local
fa440baaa779    host                host            local
806a77c3730e    none                null            local
36bdc5c064c8    wulin_default       bridge          local

## docker compose yaml template
- database
```yaml=
version: '3.7' #使用docker compose版本 3.7
services:
  sqldb:  # services名稱
    image: mariadb  
    container_name: sqldb
    hostname: sqldb
    volumes:
      - ./db:/var/lib/mysql:rw  #mount本機的./db 到mariadb container 裡面  /var/lib/mysql 在本機永久保存mariadb的資料
    environment:
      MYSQL_DATABASE: "sqldb" #建置過程中sqldb database
      MYSQL_ROOT_PASSWORD: "root" # 設定root密碼為root
```
---
## 實際使用範例

### 批次建立 學生主機
- 單一學生範例
>$ nano dkclass.temp
``` 
  s${n}:
    image: dafu/alpine.plus
    ports:
      - "221${n}:22"
```
- 使用dkclass.sh，裡面會使用dkclass.temp來批次產生yaml檔資訊
>$ nano dkclass.sh
```
#!/bin/bash

echo $'version: \'3\'
services:' > dkclass.yaml

export n=0
while [ $n -lt 30 ]
do
  cat dkclass.temp | envsubst >> dkclass.yaml
  n=$(( $n+1 ))
done
```
