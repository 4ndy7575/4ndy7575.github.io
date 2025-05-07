---
layout: post
title:  "Docker Compose 로 Ollama 설치"
date:   2025-05-07 14:57:00 +0900
categories: linux
---
- 컨테이너 포함 항목: ollama, open-webui

- Docker, Docker Compose 가 설치 되어 있어야 함

1. `디렉토리 생성 및 이동`

    ```bash
    mkdir -p ~/ollama
    cd ~/ollama
    ```

2. `docker-compose.yml 파일 작성`

    ```yaml
    version: '3.8'

    services:
    ollama:
        image: ollama/ollama
        container_name: ollama
        ports:
            - "11434:11434"
        volumes:
            - ollama_data:/root/.ollama
        restart: unless-stopped

    open-webui:
        image: ghcr.io/open-webui/open-webui:main
        container_name: open-webui
        ports:
            - "3000:8080"
        volumes:
            - openwebui_data:/app/backend/data
        environment:
            - OLLAMA_BASE_URL=http://ollama:11434
        depends_on:
            - ollama
        restart: unless-stopped

    volumes:
        ollama_data:
        openwebui_data:
    ```

3. `컨테이너 실행`

    ```bash
    docker-compose up -d
    ```

4. `[선택] GPU 를 연산에 활용하기 위한 설정`

    docker-compose.yml 파일 수정 후 컨테이너 실행

    ```yaml
    # ollama 서비스에 다음 내용 추가
        deploy:
          resources:
            reservations:
              devices:
                - capabilities: [gpu]
    ```

5. open webui 접속

    브라우저 주소창에 'localhost:3000' 또는 '<서버 IP 주소>:3000' 입력