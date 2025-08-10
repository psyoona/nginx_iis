
---

# Edge 서비스 사용 가이드

## 1. 필수 사전 작업

Edge 서비스와 개별 서비스가 통신하려면 **공유 네트워크**가 필요합니다.
이 네트워크는 최초 1회만 생성하면 되고, 서버 재부팅 후에도 유지됩니다.

```bash
# edge_proxy 네트워크 생성
docker network create edge_proxy
```

---

## 2. 서비스 실행 방법

### 2.1 Edge 서비스 실행

```bash
docker compose up -d
```

* `docker-compose.yml`이 있는 경로에서 실행합니다.
* `-d` 옵션은 백그라운드 실행을 의미합니다.

### 2.2 서비스 로그 확인

```bash
docker logs -f edge-nginx
```

* `Ctrl + C`로 로그 보기 종료.

---

## 3. 서비스 재실행 방법

### 3.1 Edge 서비스만 재실행

```bash
docker compose restart
```

또는

```bash
docker compose down && docker compose up -d
```

### 3.2 Edge + 개별 서비스 재실행

각 서비스 컨테이너가 `edge_proxy` 네트워크에 연결되어 있는지 반드시 확인하세요.

1. 서비스 중지

```bash
docker compose down
```

2. 서비스 재실행

```bash
docker compose up -d
```

3. 네트워크 연결 확인

```bash
docker network connect edge_proxy <서비스_컨테이너명>
```

* 연결이 이미 되어 있다면 오류 없이 무시됩니다.

---

## 4. 서비스 중지 방법

```bash
docker compose down
```

* 컨테이너가 삭제되지만, 볼륨과 네트워크(`edge_proxy`)는 유지됩니다.

---

## 5. 네트워크 설정 변경 방법

### 5.1 현재 연결 상태 확인

```bash
docker network inspect edge_proxy
```

### 5.2 네트워크에 서비스 추가

```bash
docker network connect edge_proxy <서비스_컨테이너명>
```

### 5.3 네트워크에서 서비스 제거

```bash
docker network disconnect edge_proxy <서비스_컨테이너명>
```

---

## 6. 신규 서비스 연결 체크리스트 ✅

신규 서비스를 Edge 환경에 연결할 때 반드시 다음 절차를 따릅니다.

1. **서비스 컨테이너 실행**

   ```bash
   docker compose up -d
   ```

2. **서비스 네트워크 연결**

   ```bash
   docker network connect edge_proxy <서비스_컨테이너명>
   ```

3. **Edge에서 접근 가능 여부 테스트**

   * 서비스 도메인: `<서비스명>.yoonslab.com`
   * 예: `curl -I http://myservice.yoonslab.com`
   * 응답 코드 `200 OK` 확인

4. **로그 확인**

   ```bash
   docker logs -f edge-nginx
   ```

   * 요청이 정상적으로 서비스 컨테이너로 전달되는지 확인.

5. **필요 시 방화벽/보안 그룹 설정**

   * 외부 접근을 허용해야 하는 경우, 서버 방화벽 및 클라우드 보안 그룹에서 80/443 포트를 허용.

---

## 7. 참고 사항

* `edge_proxy` 네트워크는 최초 1회 생성 후 계속 사용됩니다.
* 신규 서비스를 추가할 때는 반드시 해당 서비스 컨테이너를 `edge_proxy` 네트워크에 연결해야 합니다.
* `edge`의 Nginx 설정(`wildcard.conf`)은 **서비스 추가/삭제 시 수정할 필요가 없습니다**.
  서비스 명과 동일한 `<서비스명>-nginx` 컨테이너가 존재하면 자동으로 라우팅됩니다.

---