---
layout: post
title:  "Docker Compose 로 Nextcloud 설치"
date:   2025-05-07 14:52:00 +0900
categories: linux
---
- 컨테이너 포함 항목: `nextcloud`, `mariaDB`, `redis`, `collabora`, `nginx-proxy`  
- `Docker`, `Docker Compose` 가 설치 되어 있어야 함  

1. 디렉토리 생성 및 이동

    ```bash
    mkdir -p ~/nextcloud
    cd ~/nextcloud
    ```

2. `docker-compose.yml` 파일 작성

    ```yaml
    version: '3'

    services:
      nextcloud:
        image: nextcloud
        container_name: nextcloud
        restart: unless-stopped
        ports:
          - 8080:80
        volumes:
          - ./nextcloud/html:/var/www/html
          - ./nextcloud/custom_apps:/var/www/html/custom_apps
          - ./nextcloud/config:/var/www/html/config
          - ./nextcloud/data:/var/www/html/data
        environment:
          - MYSQL_PASSWORD=your_db_password
          - MYSQL_DATABASE=nextcloud
          - MYSQL_USER=nextcloud
          - MYSQL_HOST=nextclouddb
          - REDIS_HOST=redis

      nextclouddb:
        image: mariadb
        container_name: nextcloud-db
        restart: unless-stopped
        volumes:
          - ./nextclouddb:/var/lib/mysql
        environment:
          - MYSQL_ROOT_PASSWORD=your_root_password
          - MYSQL_PASSWORD=your_db_password
          - MYSQL_DATABASE=nextcloud
          - MYSQL_USER=nextcloud
      
      redis:
        image: redis:alpine
        container_name: redis
        restart: unless-stopped
        volumes:
          - ./redis:/data
      
      collabora:
        image: collabora/code
        container_name: collabora
        restart: unless-stopped
        environment:
          - domain=your_domain\\.com
          - username=admin
          - password=your_password
        ports:
          - 9980:9980
      
      nginx-proxy:
        image: 'jc21/nginx-proxy-manager:latest'
        container_name: nginx-proxy
        restart: unless-stopped
        ports:
          - '80:80'
          - '81:81'
          - '443:443'
        volumes:
          - ./nginx/data:/data
          - ./nginx/letsencrypt:/etc/letsencrypt
    ```

3. 컨테이너 실행

    ```bash
    docker-compose up -d
    ```

4. Nextcloud 접속

    브라우저 주소창에 `localhost:8080` 또는 `<서버 IP 주소>:8080` 입력  