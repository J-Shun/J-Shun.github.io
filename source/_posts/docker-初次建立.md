---
title: Docker 初次建立
date: 2024-11-05 11:23:19
tags:
---

## 重點

- Dockerfile：用來設置和建構映像檔的指令檔
- image （映像檔）：內含了運行程式碼所需的一切檔案、依賴、配置
- container （容器）：透過 image 創造的一個進程。可以放置在任一平台

## Docker 指令格式

```
docker 對象 指令 參數
```

範例：
```bash
// 列出 container
docker container ls 
docker ps

// 刪除 container 
docker container rm { container id }
docker container rm { container name }
```

## Dockerfile

純文字文件

裡面所設置的指令將被用來創建出映像檔（image）

一個 Dockerfile 可以創建出無數個 image

可以想像成藍圖，用來規劃環境如何被設置和建構

### 注意點

docker-compose 這個 yml 檔的所在位置會成為 Dockerfile 中路徑的基準

除了本地端路徑外，對應的路徑是指 linux 環境中的路徑

部分 image 會有指定的 linux 路徑，可以從官方檔確認

## 建構鏡像（image）

使用指令：`docker build -t { image name } .`

- 「.」不可省略，表示把當前目錄下的檔案及資料夾作為 Dockerfile 配置文件的上下文。專案是在本地環境而不是 Docker，所以透過「.」將當前專案送到 Docker 的環境
- -t 表示 tag，也就是將鏡像賦予標籤

## 創建容器（container）

使用指令：`docker run -d -p 80:80 --name { name } { image name }`

- -d 表示後台運行容器。以前端為例就是用 nginx 作為 web 服務器，並且持續運行
- —name 設置容器名字，若沒有則 docker 會自動生成

## 範例：使用 Docker 建立 nginx （不使用 Dockerfile）

### 建立相關檔案

在專案資料夾內建立 index.html，內容隨意，可辨識即可

### pull Nginx 的 image

使用指令： `docker pull nginx`

這項指令會下載最新的 nginx 

結束後可透過 `docker images` 指令查看是否有剛剛下載的映像檔

因為下載 nginx 的 image，因此便不需要透過 Dockerfile

### 啟動 nginx container

使用指令 `docker run --name {container名稱} -p 8080:80 -v {index.html所在路徑}:
           /usr/share/nginx/html -d nginx`

範例：
```bash
docker run --name nginxContainer -p 8080:80 -v $(pwd)/index.html:/usr/share/nginx/html/index.html -d nginx
```

- —name { container }：爲 container 取名
- -p 8080:80：將本機 8080 port 對應到容器的 80 port
- -v $(pwd)/index.html:/usr/share/nginx/html/index.html：將當前路徑的的 index.html 檔案掛載到容器的指定 linux 路徑
- -d nginx 在背景執行 nginx 容器

使用 `docker ps` 可確認正在跑的 container

有的話便可前往 `localhost:8080` 確認是否有出現 index.html 內容

### 關閉及刪除 container

可透過下列指令停止及刪除容器：

- 停止：`docker stop { container id or name }` 
- 刪除：`docker rm { container id or name }`


## 小結

順序：Dockerfile -> 鏡像 -> 容器

一個鏡像可生成多個容器，放置到各個環境運行

前端中 Docker 的價值：
- 保證運行環境統一
- 一次建構後能夠到處使用

## 參考資料

- [Docker 建立 Nginx 基礎分享](https://medium.com/@xroms123/docker-%E5%BB%BA%E7%AB%8B-nginx-%E5%9F%BA%E7%A4%8E%E5%88%86%E4%BA%AB-68c0771457fb)