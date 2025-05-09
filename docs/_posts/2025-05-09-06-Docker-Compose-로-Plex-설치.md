---
layout: post
title:  "Docker Compose 로 Plex 설치"
date:   2025-05-09 14:52:00 +0900
categories: docker
---
- 컨테이너 포함 항목: `plex`  
- `Docker`, `Docker Compose` 가 설치 되어 있어야 함  
- [Plex 계정 가입](https://www.plex.tv/): 한국에서는 가입 불가하므로 VPN 사용 필요  
- [PLEX CLAIM 토큰](https://www.plex.tv/claim) 발급 (선택)  

1. 디렉토리 생성 및 이동

    ```bash
    mkdir -p ~/plex
    cd ~/plex
    ```

2. 디렉토리 구조 예시

    ```bash
    ~/plex/
    ├── docker-compose.yml
    ├── config/         # 자동 생성됨 (Plex 설정 저장소)
    ├── transcode/      # 임시 파일용
    ├── media/
    │   ├── Movies/          # 영화
    │   ├── TVShows/         # TV 시리즈
    │   ├── Music/           # 음악
    │   ├── Photos/          # 사진
    │   └── Videos/          # 기타 비디오
    ```

3. `docker-compose.yml` 파일 작성

    ```yaml
    version: '3.8'

    services:
      plex:
        image: plexinc/pms-docker
        container_name: plex
        restart: unless-stopped
        ports:
          - "32400:32400"  # 내부 UI용
        network_mode: host  # (or bridge), host: DLNA, mDNS, GDM 등 자동탐색용
        environment:
          - TZ=Asia/Seoul
          - PLEX_CLAIM=claim-xxxxxxx  # 선택, 초기 등록 시 사용 (https://plex.tv/claim)
          - ADVERTISE_IP=http://192.168.0.10:32400/  # Plex 서버 접근용 주소
        volumes:
          - /path/to/plex/config:/config        # Plex 설정 및 메타데이터 저장 위치
          - /path/to/plex/transcode:/transcode  # 임시 트랜스코딩 경로
          - /path/to/media:/media               # 미디어 폴더 (영화, 음악 등)
    ```

    **PLEX_CLAIM 설명**  

    - Plex 계정에 서버를 자동 연결하려면 [이 링크](https://www.plex.tv/claim)에서 claim 토큰을 받아 사용  
    - 4분 유효 → 서버 첫 실행 전에 발급해야 함  (서버 실행 후 나중에 입력 불가, 초기화 필요)
    - 생략해도 수동 등록 가능  

4. 컨테이너 실행

    ```bash
    docker-compose up -d
    ```

5. Plex 접속

    브라우저 주소창에 [localhost:32400/web](http://localhost:32400/web) 또는 `<서버 IP 주소>:32400/web` 입력  

6. [선택] 방화벽 설정

    ```bash
    sudo ufw allow 32400/tcp     # Plex 웹 UI
    sudo ufw allow 1900/udp      # SSDP (DLNA 탐색)
    sudo ufw allow 32410:32414/udp  # DLNA용 포트
    ```

7. 로그인 후 라이브러리 등록

    - `media` 디렉토리는 Plex 컨테이너에 `/media`로 마운트되어야 함  
    - `docker-compose.yml` 수정 예시:  

        ```yaml
        volumes:
          - ./media:/media
        ```
    - 미디어 폴더 경로 선택 (컨테이너 내부 기준):

        | 미디어 종류 | 경로 예시 |
        |-------------|-----------|
        | 영화        | `/media/Movies` |
        | TV 시리즈   | `/media/TVShows` |
        | 음악        | `/media/Music` |
        | 사진        | `/media/Photos` |
        | 기타 비디오  | `/media/Videos` |

    - [선택] 폴더 권한 문제 해결

        Plex가 미디어 폴더를 읽을 수 있도록 **파일 시스템 권한을 확인

        ```bash
        sudo chown -R 1000:1000 ./media
        sudo chmod -R 755 ./media
        ```

        > Plex Docker 컨테이너의 기본 UID가 `1000`인 경우 (환경에 따라 다를 수 있음)

8. DLNA 활성화: 설정 → DLNA → "Enable the DLNA server" 체크

9. [선택] Plex 설정 초기화 후 서버 다시 실행

    설치 중에 문제가 발생하면 초기화 후 다시 실행 가능
    
    ```bash
    # 기존 Plex 설정 삭제 (주의: 모든 설정 초기화됨!)
    sudo rm -rf ./config/*

    # 새로운 Claim 코드 발급
    # https://www.plex.tv/claim

    # docker-compose.yml에 PLEX_CLAIM=claim-xxxxx 추가 후 재실행
    docker-compose up -d
    ```

    > 이 방식은 **기존 설정이 전부 초기화**되므로 신중하게 사용 할 것

10. [선택] 서브 도메인으로 접속하기 위한 설정

    [서브 도메인 설정 방법](2025-05-07-02-도커-컨테이너에-서브도메인-적용.md)을 참고하되 아래 내용을 별도로 설정  

    1. DNS 설정(Cloudflare 등)에서 해당 서브도메인의 프록시 상태를 `DNS only` 로 설정

        - SSL 인증서 발급 이후에도 유지해야함  
        - Plex 는 WebSocket 사용하기 때문  

    2. Cloudflare 사용 시 SSL 모드를 **Full** 이상 사용

        Plex 는 WebSocket 사용하기 때문  

    3. Nginx 리버스 프록시 설정

        Plex 는 WebSocket 사용하기 때문에 프록시에서 WebSocket 헤더 지원 필요  

        ```nginx
        server {
            listen 443 ssl;
            ...
        
            location / {
                ...
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
            }
        ```

    4. `docker-compose.yml` 파일 작성시 환경변수 수정

        ```yaml
        # [삭제 가능] 서브도메인 접속만 사용할 경우, 외부에서 포트를 직접 노출하지 않아도 됨
        ports:
          - "32400:32400"
        
        # Plex 클라이언트가 내부 주소가 아닌 서브도메인으로 인식되도록 수정
        environment:
          - ADVERTISE_IP=https://plex.example.com:443/  
        ```