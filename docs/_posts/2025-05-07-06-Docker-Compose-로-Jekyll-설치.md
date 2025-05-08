---
layout: post
title:  "Docker Compose 로 Jekyll 설치"
date:   2025-05-07 15:06:00 +0900
categories: linux
---
- 컨테이너 포함 항목: jekyll

- Docker, Docker Compose 가 설치 되어 있어야 함

### 1. 디렉토리 생성 및 이동`

```bash
mkdir -p ~/jekyll-site/docs
cd ~/jekyll-site
```

### 2. `docker-compose.yml` 파일 작성

```yaml
version: '3.8'
services:
jekyll:
    image: jekyll/jekyll:4.2.2
    container_name: jekyll-server
    command: jekyll serve --watch --force_polling --host 0.0.0.0
    ports:
        - "4000:4000"
    volumes:
        - ./docs:/srv/jekyll
    working_dir: /srv/jekyll
    tty: true
    restart: unless-stopped
```

### 3. `./docs/Gemfile` 파일에 다음 내용 추가

```ruby
gem 'webrick'
```

### 4. 컨테이너 실행

```bash
docker-compose up -d
```

### 5. 사이트 접속

브라우저 주소창에 'localhost:4000' 또는 '<서버 IP 주소>:4000' 입력