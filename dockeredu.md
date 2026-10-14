# Docker 容器化教學 — 從入門到實戰

> 適合完全沒接觸過 Docker 的初學者，由淺入深，每個指令都有實際範例。
> 環境：Ubuntu 24.04 / Debian，Docker Engine 已安裝。

---

## 目錄

1. [什麼是 Docker？為什麼要用它](#1-什麼是-docker為什麼要用它)
2. [核心概念：映像檔與容器](#2-核心概念映像檔與容器)
3. [安裝 Docker](#3-安裝-docker)
4. [第一個容器：docker run](#4-第一個容器docker-run)
5. [容器生命週期管理](#5-容器生命週期管理)
6. [查看與除錯：docker logs](#6-查看與除錯docker-logs)
7. [映像檔管理：docker image](#7-映像檔管理docker-image)
8. [資料持久化：Volume 掛載](#8-資料持久化volume-掛載)
9. [網路管理：docker network](#9-網路管理docker-network)
10. [多容器應用：Docker Compose](#10-多容器應用docker-compose)
11. [撰寫 Dockerfile](#11-撰寫-dockerfile)
12. [常見問題排解](#12-常見問題排解)
13. [指令速查表](#13-指令速查表)

---

## 1. 什麼是 Docker？為什麼要用它

### 傳統部署的問題

想像你要在一台新伺服器上跑一個網站，你需要：

- 安裝 Python/Node.js/Java
- 安裝一堆相依套件
- 設定環境變數
- 處理版本衝突（「我電腦上可以跑啊！」）

每台機器環境都不同，導致「在我機器上是好的」這種經典問題。

### Docker 的解法

Docker 把**應用程式 + 所有相依套件 + 執行環境**打包成一個標準化的「容器影像」（Image）。

```
你的應用
  ↓ 打包
Docker Image（一個檔案，到處都能跑）
  ↓ 執行
Container（正在運行的實例）
```

### 三個好處

| 好處 | 說明 |
|------|------|
| **環境一致** | 開發、測試、正式環境跑同一個 Image，不會有「我電腦可以」的問題 |
| **快速部署** | 不用重新安裝環境，Image 拉下來就能跑 |
| **資源節省** | 比虛擬機輕量，一台機器可以跑很多容器 |

### Docker vs 虛擬機

```
虛擬機（VM）                    Docker 容器
┌─────────────────┐            ┌─────────────────┐
│  App A  │ App B │            │  App A  │ App B │
│ 客體OS │ 客體OS│            ├─────────┴─────────┤
├─────────┴───────┤            │   Docker Engine  │
│   Hypervisor    │            ├───────────────────┤
├─────────────────┤            │   Host 作業系統   │
│   Host 作業系統  │            ├───────────────────┤
├─────────────────┤            │     硬體          │
│     硬體        │            └───────────────────┘
└─────────────────┘
   幾個 GB，開機幾分鐘          幾十 MB，啟動幾秒
```

---

## 2. 核心概念：映像檔與容器

這兩個名詞一定要分清楚，這是 Docker 最重要的基礎概念。

### Image（映像檔）

- **唯讀的模板**，包含執行應用所需的一切（程式碼、執行環境、相依套件）
- 就像「食譜」或「安裝光碟」
- 不會改變，可以重複使用
- 存在本機或遠端倉庫（Docker Hub）

### Container（容器）

- Image 的**執行實例**
- 就像「用食譜煮出來的菜」或「用光碟裝好的系統」
- 可以啟動、停止、刪除
- 多個容器可以用同一個 Image 建立

### 比喻

```
Image    =  食譜（固定的，不會變）
Container =  用食譜煮出來的菜（每次煮都是一個新實例）
```

### 常用關係圖

```
Docker Hub（遠端倉庫）
    │ docker pull（拉取）
    ▼
本機 Images ──────┐
    │ docker run  │
    ▼             │
Container A    Container B   ← 同一個 Image 可以跑多個容器
    │             │
    ▼             ▼
  執行中        執行中
```

---

## 3. 安裝 Docker

### Ubuntu / Debian 安裝（官方一鍵腳本）

```bash
# 更新套件索引
sudo apt update

# 安裝相依套件
sudo apt install -y ca-certificates curl

# 下載並執行官方安裝腳本
curl -fsSL https://get.docker.com | sudo sh

# 把自己加入 docker 群組（這樣不用每次加 sudo）
sudo usermod -aG docker $USER

# 重新登入讓群組生效（或執行以下指令暫時生效）
newgrp docker
```

### 驗證安裝

```bash
docker --version
# Docker version 27.x.x, build xxxxx

docker run hello-world
# 如果看到 "Hello from Docker!" 就成功了
```

### 沒有 sudo 權限時

如果執行 `docker ps` 出現：

```
permission denied while trying to connect to the Docker daemon socket
```

代表你還沒加入 docker 群組，執行：

```bash
sudo usermod -aG docker $USER
# 然後登出再登入
```

---

## 4. 第一個容器：docker run

`docker run` 是最重要的指令，它做兩件事：
1. 如果本機沒有這個 Image，自動從 Docker Hub 拉取
2. 用這個 Image 建立並啟動一個容器

### 最簡單的範例

```bash
docker run hello-world
```

會發生什麼事：

```
1. Docker 找不到 hello-world 這個 Image
2. 自動從 Docker Hub 下載（pull）
3. 用這個 Image 建立一個容器
4. 容器執行，印出問候訊息
5. 容器結束（這個 Image 設計成跑完就結束）
```

### 執行一個 Nginx 網站

```bash
docker run -d -p 8080:80 --name my-nginx nginx
```

逐個參數解釋：

| 參數 | 意思 |
|------|------|
| `-d` | Detached，背景執行（不然會卡住你的終端機） |
| `-p 8080:80` | Port 映射，本機 8080 → 容器 80 |
| `--name my-nginx` | 給容器取名字（不取會自動產生亂碼名） |
| `nginx` | 使用哪個 Image |

然後打開瀏覽器：http://localhost:8080 → 看到 Nginx 歡迎頁面

### 執行 MySQL 資料庫

```bash
docker run -d \
  --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  -e MYSQL_DATABASE=mydb \
  -p 3306:3306 \
  mysql:8.0
```

| 參數 | 意思 |
|------|------|
| `-e` | Environment，設定環境變數（MySQL 需要設定密碼） |
| `mysql:8.0` | 指定版本（`:8.0` 是 tag，不指定預設用 `latest`） |

連線測試：

```bash
docker exec -it my-mysql mysql -uroot -pmypassword
# 進入 MySQL 命令列
```

### 互動式容器（Ubuntu 終端機）

```bash
docker run -it --name my-ubuntu ubuntu:24.04 /bin/bash
```

| 參數 | 意思 |
|------|------|
| `-it` | Interactive + TTY，讓你可以在容器內打字 |
| `/bin/bash` | 容器啟動後執行的指令 |

進去後就像在一台 Ubuntu 裡：

```bash
root@container:/# apt update
root@container:/# ls /
root@container:/# exit    # 離開，容器會停止
```

### docker run 常用參數總整理

```bash
docker run [選項] <映像檔> [指令]
```

| 參數 | 說明 | 範例 |
|------|------|------|
| `-d` | 背景執行 | `docker run -d nginx` |
| `-p` | 埠號映射（本機:容器） | `-p 8080:80` |
| `--name` | 容器名稱 | `--name web` |
| `-e` | 環境變數 | `-e PASSWORD=123` |
| `-v` | 掛載 Volume | `-v ./data:/app/data` |
| `-it` | 互動模式 | `-it ubuntu bash` |
| `--rm` | 停止後自動刪除 | `docker run --rm nginx` |
| `--network` | 指定網路 | `--network mynet` |
| `--restart` | 重啟策略 | `--restart always` |
| `-w` | 工作目錄 | `-w /app` |

---

## 5. 容器生命週期管理

### 查看容器

```bash
# 查看運行中的容器
docker ps

# 查看所有容器（包含已停止的）
docker ps -a

# 只顯示容器 ID（方便腳本使用）
docker ps -q
```

輸出說明：

```
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
a1b2c3d4e5f6   nginx   "..."     2 min ago   Up 2 min   0.0.0.0:8080->80/tcp   my-nginx
```

### 啟動 / 停止 / 重啟

```bash
docker start my-nginx      # 啟動已停止的容器
docker stop my-nginx       # 停止（優雅關閉，等 10 秒後強制）
docker kill my-nginx       # 立即殺掉（不等待）
docker restart my-nginx    # 重啟
docker pause my-nginx      # 暫停（凍結程序）
docker unpause my-nginx    # 恢復暫停
```

### 刪除容器

```bash
docker rm my-nginx         # 刪除已停止的容器
docker rm -f my-nginx      # 強制刪除（即使還在運行）
docker rm $(docker ps -aq) # 刪除所有已停止的容器
```

### 進入容器內部

```bash
# 進入正在運行的容器（開一個新 shell）
docker exec -it my-nginx /bin/bash

# 如果容器裡沒有 bash，試試 sh
docker exec -it my-nginx sh
```

### 複製檔案

```bash
# 從容器複製到本機
docker cp my-nginx:/etc/nginx/nginx.conf ./nginx.conf

# 從本機複製到容器
docker cp ./myfile.txt my-nginx:/tmp/myfile.txt
```

### 查看容器詳細資訊

```bash
# 查看詳細設定（IP、掛載、環境變數等）
docker inspect my-nginx

# 只看 IP
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-nginx

# 查看容器資源使用
docker stats

# 查看容器內的程序
docker top my-nginx
```

---

## 6. 查看與除錯：docker logs

`docker logs` 是排查問題最常用的指令。

### 基本用法

```bash
# 查看所有 log
docker logs my-nginx

# 持續追蹤新的 log（像 tail -f）
docker logs -f my-nginx

# 只看最後 50 行
docker logs --tail 50 my-nginx

# 加上時間戳記
docker logs -t my-nginx
```

### 組合技（常用）

```bash
# 即時追蹤最後 100 行
docker logs -f --tail 100 my-nginx

# 只顯示錯誤相關的 log
docker logs my-nginx 2>&1 | grep -i error

# 查看特定時間之後的 log
docker logs --since 10m my-nginx     # 最近 10 分鐘
docker logs --since 2024-01-01 my-nginx
```

### 實際範例：排查網站無法連線

```bash
# 1. 確認容器是否運行
docker ps | grep my-nginx

# 2. 看 log 有沒有錯誤
docker logs my-nginx --tail 30

# 3. 進去容器確認服務是否正常
docker exec -it my-nginx curl localhost:80

# 4. 看容器 IP 和埠號映射
docker port my-nginx
docker inspect my-nginx | grep IPAddress
```

---

## 7. 映像檔管理：docker image

### 查看本機 Image

```bash
docker images

# 或新語法
docker image ls
```

輸出說明：

```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    a6bd71f48f68   2 weeks ago   192MB
mysql        8.0       xxxxxxxx       3 weeks ago   1.2GB
```

### 拉取 Image

```bash
# 拉取最新版
docker pull nginx

# 拉取指定版本
docker pull nginx:1.25

# 拉取特定架構
docker pull --platform linux/amd64 nginx
```

### 刪除 Image

```bash
# 刪除指定 Image
docker rmi nginx

# 刪除多個
docker rmi nginx mysql redis

# 強制刪除（即使有容器在用）
docker rmi -f nginx

# 清理所有沒被使用的 Image
docker image prune

# 清理所有沒被使用的資源（Image、容器、網路、Build 快取）
docker system prune -a
```

### 查看 Image 詳細資訊

```bash
# 查看 Image 的分層結構
docker history nginx

# 查看 Image 的詳細設定
docker inspect nginx

# 查看 Image 的環境變數
docker inspect nginx | grep -A 20 '"Env"'
```

### Image 標籤（Tag）

```bash
# 給現有 Image 打新標籤
docker tag nginx my-nginx:v1.0

# 推到 Docker Hub（需要先登入）
docker login
docker tag my-app myusername/my-app:v1.0
docker push myusername/my-app:v1.0
```

### Image 大小問題

如果 Image 太大（好幾 GB），可以用 `docker history` 看哪一層最佔空間：

```bash
docker history --no-trunc my-app
```

---

## 8. 資料持久化：Volume 掛載

### 為什麼需要 Volume？

容器刪掉後，裡面的資料就消失了。如果資料需要保存（例如資料庫），就要用 Volume。

### 兩種掛載方式

#### 方式一：Bind Mount（綁定本機目錄）

```bash
docker run -d \
  --name my-nginx \
  -v /home/user/mywebsite:/usr/share/nginx/html \
  -p 8080:80 \
  nginx
```

改本機 `/home/user/mywebsite/index.html`，容器裡的網站立即生效。

**適用場景**：開發時，本機和容器共享檔案。

#### 方式二：Named Volume（具名卷）

```bash
docker run -d \
  --name my-mysql \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  mysql:8.0
```

`mysql-data` 是 Docker 管理的卷，即使容器刪了，資料還在。

**適用場景**：正式環境，資料庫持久化。

### 管理 Volume

```bash
# 查看所有 Volume
docker volume ls

# 查看某個 Volume 的詳細資訊
docker volume inspect mysql-data

# 刪除 Volume
docker volume rm mysql-data

# 清理沒被使用的 Volume
docker volume prune
```

### 開發環境常用模式

```bash
# 程式碼掛載 + 自動重載（例如 Node.js）
docker run -d \
  --name dev-app \
  -v $(pwd)/src:/app/src \
  -p 3000:3000 \
  node:20-alpine \
  sh -c "cd /app && npm install && npm run dev"
```

---

## 9. 網路管理：docker network

### Docker 網路基礎

每個容器啟動時，Docker 會給它一個虛擬 IP。容器之間可以透過 IP 或**容器名稱**互相通訊（前提是同一個網路）。

### 查看網路

```bash
docker network ls
```

預設有三個：

| 網路 | 說明 |
|------|------|
| `bridge` | 預設網路，自訂容器如果不指定就用這個 |
| `host` | 直接用本機網路，不隔離 |
| `none` | 沒有網路 |

### 建立自訂網路

```bash
docker network create my-network
```

### 讓容器加入網路

```bash
# 建立時就加入
docker run -d --name web --network my-network nginx

# 執行中加入
docker network connect my-network web

# 離開網路
docker network disconnect my-network web
```

### 實例：網站 + 資料庫通訊

```bash
# 1. 建立網路
docker network create app-net

# 2. 啟動資料庫
docker run -d \
  --name db \
  --network app-net \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# 3. 啟動網站（可以用容器名 db 連接，不需要知道 IP）
docker run -d \
  --name web \
  --network app-net \
  -p 8080:80 \
  my-web-app
```

在網站程式裡，連接字串直接寫：

```
mysql://root:secret@db:3306/mydb
                    ↑
              容器名稱就是主機名
```

這就是容器網路最大的優勢 — 不需要記 IP。

### 查看網路詳情

```bash
# 查看某個網路有哪些容器
docker network inspect app-net

# 只顯示容器名稱
docker network inspect app-net -f '{{range .Containers}}{{.Name}} {{end}}'
```

### DNS 解析問題

有時候容器名稱無法解析，確認：

```bash
# 兩個容器是否在同一個網路
docker network inspect app-net | grep -A 5 Name
```

---

## 10. 多容器應用：Docker Compose

當應用需要多個容器（網站 + 資料庫 + Redis），一個一個 `docker run` 太麻煩。Docker Compose 讓你用一個 YAML 檔案定義所有服務。

### 安裝

```bash
# Ubuntu 通常 Docker 已內建
docker compose version

# 如果沒有
sudo apt install docker-compose-plugin
```

### 第一個 docker-compose.yml

```yaml
services:
  # 網站服務
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    restart: always

  # 資料庫服務
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    volumes:
      - db-data:/var/lib/mysql
    restart: always

  # Redis 快取
  cache:
    image: redis:7-alpine
    restart: always

# 宣告具名卷
volumes:
  db-data:
```

### 啟動與管理

```bash
# 啟動所有服務（背景執行）
docker compose up -d

# 停止所有服務
docker compose down

# 停止並刪除 Volume（資料會消失！）
docker compose down -v

# 重新建置並啟動（修改 Dockerfile 或 docker-compose.yml 後）
docker compose up -d --build

# 查看服務狀態
docker compose ps

# 查看 log
docker compose logs

# 追蹤特定服務的 log
docker compose logs -f web

# 執行一次性指令（例如進資料庫）
docker compose exec db mysql -uroot -psecret

# 重啟單一服務
docker compose restart web
```

### docker-compose.yml 常用設定

```yaml
services:
  app:
    build: .                    # 從 Dockerfile 建置
    # 或
    image: myapp:latest         # 使用現有 Image

    ports:
      - "8080:80"               # 本機埠:容器埠
      - "127.0.0.1:9090:9090"  # 只綁本機（不對外）

    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/mydb
    # 或使用 env 檔案
    env_file:
      - .env

    volumes:
      - ./src:/app/src          # 綁定掛載
      - app-data:/app/data      # 具名卷

    depends_on:
      - db                      # 確保 db 先啟動
      - redis

    restart: unless-stopped      # 重啟策略

    networks:
      - app-net                 # 指定網路

    # 資源限制
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

networks:
  app-net:
    driver: bridge

volumes:
  app-data:
```

### 實際範例：WordPress

```yaml
services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
    volumes:
      - wp-data:/var/www/html
    depends_on:
      - db
    restart: always

  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
      MYSQL_ROOT_PASSWORD: rootpass
    volumes:
      - db-data:/var/lib/mysql
    restart: always

volumes:
  wp-data:
  db-data:
```

存成 `docker-compose.yml`，然後：

```bash
docker compose up -d
# 打開 http://localhost:8080 就有一個 WordPress 了
```

### 環境變數檔案（.env）

把敏感資訊從 docker-compose.yml 抽出來：

```bash
# .env 檔案
MYSQL_ROOT_PASSWORD=mysecret
NGINX_PORT=8080
```

```yaml
# docker-compose.yml
services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}

  web:
    image: nginx
    ports:
      - "${NGINX_PORT}:80"
```

> **注意**：`.env` 不要提交到 Git！加到 `.gitignore`。

---

## 11. 撰寫 Dockerfile

Dockerfile 是建置 Image 的食譜，告訴 Docker 如何一步步建構你的應用。

### 基本語法

```dockerfile
# 使用什麼基礎 Image
FROM python:3.12-slim

# 設定工作目錄（之後的指令都在這裡執行）
WORKDIR /app

# 複製檔案進 Image（本機路徑 → 容器路徑）
COPY requirements.txt .
COPY . .

# 安裝相依套件
RUN pip install --no-cache-dir -r requirements.txt

# 宣告容器對外開放的埠（僅文件用途，實際映射靠 -p）
EXPOSE 8000

# 容器啟動時執行的指令
CMD ["python", "app.py"]
```

### 常用指令說明

| 指令 | 用途 | 範例 |
|------|------|------|
| `FROM` | 基礎 Image | `FROM ubuntu:24.04` |
| `WORKDIR` | 設定工作目錄 | `WORKDIR /app` |
| `COPY` | 複製檔案 | `COPY . /app` |
| `ADD` | 複製（支援 URL、自動解壓縮） | `ADD app.tar.gz /opt/` |
| `RUN` | 建置時執行（安裝套件等） | `RUN apt update` |
| `CMD` | 啟動時預設執行的指令 | `CMD ["nginx", "-g", "daemon off;"]` |
| `ENTRYPOINT` | 容器進入點（較不易被覆蓋） | `ENTRYPOINT ["python"]` |
| `ENV` | 設定環境變數 | `ENV PATH=/app/bin:$PATH` |
| `EXPOSE` | 宣告埠號（僅文件用） | `EXPOSE 8080` |
| `VOLUME` | 宣告掛載點 | `VOLUME /data` |
| `USER` | 切換使用者 | `USER nobody` |
| `ARG` | 建置時傳入的變數 | `ARG VERSION=latest` |

### CMD vs ENTRYPOINT

```dockerfile
# CMD：可以被 docker run 後面的指令覆蓋
CMD ["python", "app.py"]
# docker run my-app python other.py  → 執行 other.py

# ENTRYPOINT：固定執行這個，後面的參數當參數傳入
ENTRYPOINT ["python", "app.py"]
# docker run my-app --debug  → 執行 python app.py --debug
```

### 多階段建構（縮小 Image 大小）

```dockerfile
# 第一階段：建置
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 第二階段：只取建置結果，乾淨又小
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

原本 1GB+ 的 Node.js Image，最後可以縮到 50MB 以下。

### .dockerignore（加速建置、避免洩漏）

```
node_modules
.git
.env
*.log
__pycache__
dist
```

沒加 `.dockerignore`，`COPY . .` 會把 `node_modules`、`.git` 全都複製進去，Image 暴肥。

### 建置與測試

```bash
# 建置 Image
docker build -t my-app:v1 .

# 加上建置參數
docker build --build-arg VERSION=2.0 -t my-app:v2 .

# 指定 Dockerfile 位置
docker build -f Dockerfile.prod -t my-app:prod .

# 建置時不使用快取（從頭來過）
docker build --no-cache -t my-app .
```

### 實例：Python Flask 應用

```dockerfile
# Dockerfile
FROM python:3.12-slim

# 設定環境變數（減少 .pyc 檔、即時輸出 log）
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# 先複製 requirements（利用 Docker 快取機制）
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 再複製程式碼
COPY . .

EXPOSE 5000

# 用 gunicorn 跑（比 Flask 內建伺服器適合正式環境）
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

```dockerfile
# .dockerignore
__pycache__
*.pyc
.git
.env
venv/
*.log
```

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/app
    environment:
      - FLASK_ENV=development
    restart: unless-stopped
```

---

## 12. 常見問題排解

### 容器啟動後馬上退出

```bash
# 看 log 找原因
docker logs my-container

# 常見原因：
# 1. CMD 指令寫錯
# 2. 應用程式啟動失敗（連接資料庫失敗等）
# 3. 缺少必要的環境變數
```

### 無法連線到容器

```bash
# 1. 確認容器在跑
docker ps | grep my-container

# 2. 確認埠號映射正確
docker port my-container

# 3. 從本機測試
curl http://localhost:8080

# 4. 進容器內部測（排除網路問題）
docker exec -it my-container curl localhost:80

# 5. 如果容器用 hostNetwork，埠號映射無效，直接打容器 IP
docker inspect my-container | grep IPAddress
```

### Permission denied

```bash
# docker 指令需要 sudo
sudo usermod -aG docker $USER
# 登出再登入
```

### 磁碟空間不足

```bash
# 查看 Docker 佔用空間
docker system df

# 清理所有沒被使用的資源
docker system prune -a --volumes

# 刪除所有停止的容器
docker container prune
```

### Image 太大

```bash
# 找出哪一層最佔空間
docker history my-app

# 解法：
# 1. 用 Alpine 版本（alpine 比 ubuntu 小很多）
# 2. 多階段建構
# 3. 在同一個 RUN 裡清理暫存檔
```

### 容器名稱衝突

```bash
docker: Error response from daemon: Conflict. The container name "/my-app" is already in use

# 解法：刪掉舊的
docker rm -f my-app
```

### docker compose 指令找不到

新版 Docker 用 `docker compose`（空格），舊版是 `docker-compose`（減號）：

```bash
# 新版（推薦）
docker compose up -d

# 舊版
docker-compose up -d
```

---

## 13. 指令速查表

### 容器操作

```bash
docker run -d -p 8080:80 --name web nginx    # 啟動容器
docker ps                                     # 運行中的容器
docker ps -a                                  # 所有容器
docker stop web                               # 停止
docker start web                              # 啟動
docker restart web                            # 重啟
docker rm -f web                              # 刪除
docker exec -it web bash                      # 進入容器
docker logs -f --tail 100 web                 # 看 log
docker stats                                  # 資源使用
docker inspect web                            # 詳細資訊
docker cp web:/path/file ./                   # 複製出來
```

### 映像檔操作

```bash
docker images                                  # 本機 Image
docker pull nginx:1.25                         # 拉取
docker build -t myapp:v1 .                     # 建置
docker rmi myapp                               # 刪除
docker tag myapp user/myapp:v1                 # 標籤
docker push user/myapp:v1                      # 推送
docker system prune -a                         # 清理
```

### 網路操作

```bash
docker network ls                              # 所有網路
docker network create mynet                    # 建立
docker network connect mynet web               # 容器加入
docker network disconnect mynet web            # 離開
docker network inspect mynet                   # 詳情
docker network rm mynet                        # 刪除
```

### Volume 操作

```bash
docker volume ls                               # 所有卷
docker volume create mydata                    # 建立
docker volume inspect mydata                   # 詳情
docker volume rm mydata                        # 刪除
docker volume prune                            # 清理未使用
```

### Docker Compose

```bash
docker compose up -d                           # 啟動
docker compose down                            # 停止
docker compose down -v                         # 停止並刪卷
docker compose up -d --build                   # 重建
docker compose ps                              # 狀態
docker compose logs -f web                     # log
docker compose exec db bash                    # 進容器
docker compose restart web                     # 重啟單一服務
```

### 系統維護

```bash
docker system df                               # 空間使用
docker system prune                            # 清理
docker system prune -a --volumes               # 深度清理
docker version                                 # 版本
docker info                                    # 系統資訊
```

---

## 學習路徑建議

```
第 1 天：理解概念 → docker run 跑幾個範例
第 2 天：docker logs 排查問題、docker exec 進容器
第 3 天：docker image 管理、build 簡單 Dockerfile
第 4 天：docker network 讓兩個容器通訊
第 5 天：docker compose 跑一個完整應用（網站+DB）
第 6 天：Dockerfile 最佳實踐、多階段建構
第 7 天：實戰 — 把自己的一個專案容器化
```

## 參考資源

- Docker 官方文件：https://docs.docker.com/
- Docker Hub：https://hub.docker.com/
- Play with Docker（線上練習）：https://labs.play-with-docker.com/
