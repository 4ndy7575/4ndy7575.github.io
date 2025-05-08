# Nginx 삭제 방법

`user`  
nginx 삭제하기

`ChatGPT`  
Ubuntu 환경에서 Nginx를 **완전히 삭제**하는 방법을 순서대로 안내드리겠습니다.  
(패키지 제거 → 설정파일 삭제 → 의존 패키지 정리)

---

### ✅ 1단계: Nginx 패키지 제거

```bash
sudo apt remove nginx nginx-common
```

- `nginx`: Nginx 실행 파일을 포함한 기본 패키지
- `nginx-common`: 설정 파일과 기본 웹 페이지 등을 포함하는 패키지

---

### ✅ 2단계: 남은 설정/캐시 파일 삭제

```bash
sudo apt purge nginx nginx-common
```

- `purge`는 설정 파일까지 모두 삭제합니다.

---

### ✅ 3단계: 의존성 정리

```bash
sudo apt autoremove
```

- 자동 설치된 의존 패키지 중 불필요해진 항목을 정리합니다.

---

### ✅ 4단계: 남은 디렉터리 수동 삭제 (선택)

```bash
sudo rm -rf /etc/nginx /var/log/nginx /var/www/html
```

- 설정 디렉터리: `/etc/nginx`
- 로그 디렉터리: `/var/log/nginx`
- 기본 웹 루트: `/var/www/html` (사용 중인 다른 서비스가 없다면 삭제 가능)

---

### 📌 확인

```bash
which nginx
```

출력이 없으면 삭제 완료된 상태입니다.

---

필요 시 `systemctl`에 남은 서비스 등록 여부도 확인 가능합니다:

```bash
sudo systemctl list-units | grep nginx
```

---

원하신다면 Docker 환경에서의 Nginx 제거 방법도 별도로 안내드릴 수 있습니다.