# 트러블슈팅 기록

Docker 실습 오류 2건: **발생 → 확인 → 수정 → 결과**.

## 1. 컨테이너 이름 오타

**발생**

```powershell
docker logs codyssey-weeb
```

```text
Error response from daemon: No such container: codyssey-weeb
```

**원인:** 실제 컨테이너 이름은 `codyssey-web`. `weeb` 오타.

**확인 및 수정**

```powershell
docker ps -a
docker logs codyssey-web
```

**결과:** NGINX 기동 로그와 `GET /` 요청의 HTTP `200` 확인.

**사진·로그:** [오류·해결 화면](../screenshots/18-error-name.png) · [실행 로그](verified/09-troubleshooting.txt)

## 2. 컨테이너 내부 경로 오타

**발생:** 웹 서버 파일 확인 중 경로 오류.

**오류 명령 (메시지 기준 재구성)**

```powershell
docker exec codyssey-web ls /usr/nginx/html
```

**기록된 오류**

```text
ls: /usr/nginx/html: No such file or directory
```

**원인:** 올바른 경로 `/usr/share/nginx/html`에서 `share` 누락.

**확인 및 수정**

```powershell
docker exec codyssey-web ls /usr/share/nginx/html
docker exec codyssey-web cat /usr/share/nginx/html/index.html
```

**결과:** `index.html` 확인 및 HTML 내용 출력.

**사진·로그:** [오류·해결 화면](../screenshots/19-error-path.png) · [실행 로그](verified/09-troubleshooting.txt)

