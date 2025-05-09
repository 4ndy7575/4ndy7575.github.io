---
layout: post
title:  "Docker Compose 로 Gitea 설치"
date:   2025-05-07 15:11:00 +0900
categories: docker
---
- 컨테이너 포함 항목: `gitea`  
- `Docker`, `Docker Compose` 가 설치 되어 있어야 함  

1. 디렉토리 생성 및 이동

    ```bash
    mkdir -p ~/gitea
    cd ~/gitea
    ```

2. `docker-compose.yml` 파일 작성

    ```yaml
    version: "3"

    services:
      gitea:
        image: gitea/gitea:latest
        container_name: gitea
        restart: always
        environment:
          - USER_UID=1000
          - USER_GID=1000
        volumes:
          - ./gitea:/data
        ports:
          - "3000:3000"   # Web UI
          - "222:22"      # SSH
    ```

3. 컨테이너 실행

    ```bash
    docker-compose up -d
    ```

4. 사이트 접속

    브라우저 주소창에 [localhost:3000](http://localhost:3000) 또는 `<서버 IP 주소>:3000` 입력  