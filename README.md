# Codyssey Development Workstation

CLI · Linux 권한 · Docker · Git/GitHub 개발 환경 구축 실습.

## 1. 실습 결과

| 항목 | 확인 결과 | 사진·기록 |
|---|---|---|
| CLI | 파일 생성·복사·이동·삭제 | [터미널 화면](screenshots/02-terminal.png) · [로그](logs/verified/01-terminal.txt) |
| Linux 권한 | 파일 `600 → 644`, 폴더 `700 → 755` | [권한 변경](screenshots/03-permission-file.png) · [755 최종 확인](screenshots/04-permission-folder-755.png) |
| Docker 기본 | Linux 엔진, `hello-world`, Ubuntu 컨테이너 실행 | [환경](screenshots/05-docker-server.png) · [실행](screenshots/06-hello-world.png) |
| Dockerfile | NGINX 기반 이미지 `codyssey-web:v1` 빌드 | [Dockerfile](screenshots/10-dockerfile.png) · [빌드](screenshots/11-web-build.png) |
| 웹 서버 | `localhost:8080` 접속, HTTP `200` | [브라우저](screenshots/12-browser.png) · [로그](logs/verified/06-webserver.txt) |
| 바인드 마운트 | HTML 수정 전·후 화면 반영 | [변경 전](screenshots/14-bind-before.png) · [변경 후](screenshots/15-bind-after.png) |
| Docker 볼륨 | 컨테이너 삭제 후 `SAVE-OK` 유지 | [삭제 후 재확인](screenshots/17-volume-after.png) · [로그](logs/verified/08-volume.txt) |
| Git/GitHub | 원격 저장소 연결, 커밋·업로드 확인 | [원격 주소](screenshots/22-github-remote.png) · [Push 결과](screenshots/git-push-success.png) |
| 트러블슈팅 | 컨테이너 이름·내부 경로 오류 2건 | [해결 기록](logs/troubleshooting.md) |

## 2. 실행 환경

| 항목 | 환경 |
|---|---|
| OS | Windows |
| 터미널 | PowerShell 5.1 |
| 편집기 | Visual Studio Code |
| Docker | 29.9.2 / Linux 컨테이너 |
| Git | 2.56.0.windows.4 / `main` |
| 웹 서버 | `nginx:alpine` |

## 3. 다른 PC에서 웹 서버 실행

**준비:** Docker Desktop 설치 및 Linux 컨테이너 엔진 실행. 아래 명령은 PowerShell에서 실행.

```powershell
git clone https://github.com/hyeonla906/codyssey-workstation.git
cd codyssey-workstation
docker build -t codyssey-web:v1 .
docker run -d --name codyssey-web -p 8080:80 codyssey-web:v1
curl.exe http://localhost:8080
```

**확인:** 브라우저에서 `http://localhost:8080` 접속. `codyssey-web` 컨테이너와 웹페이지 표시 확인.

## 4. CLI · Linux 권한

### 4-1. PowerShell 파일 관리

```powershell
pwd
Get-ChildItem -Force
mkdir practice\lesson1
cd practice\lesson1
New-Item empty.txt -ItemType File
"Hello Codyssey" | Set-Content hello.txt
Get-Content hello.txt
Copy-Item hello.txt copy.txt
Rename-Item copy.txt renamed.txt
mkdir backup
Move-Item renamed.txt backup
Remove-Item backup\renamed.txt
cd ..\..
```

**확인:** `Hello Codyssey` 출력. 파일 생성·복사·이동·삭제 확인. [터미널 사진](screenshots/02-terminal.png) · [실행 로그](logs/verified/01-terminal.txt)

### 4-2. Linux 권한 변경

Ubuntu 컨테이너(`permission-lab`)에서 실행.

```powershell
docker run -d --name permission-lab ubuntu sleep infinity
docker exec permission-lab touch /test.txt
docker exec permission-lab chmod 600 /test.txt
docker exec permission-lab chmod 644 /test.txt
docker exec permission-lab mkdir /testdir
docker exec permission-lab chmod 700 /testdir
docker exec permission-lab chmod 755 /testdir
docker exec permission-lab ls -ld /testdir
```

| 대상 | 권한 | 확인 |
|---|---|---|
| `/test.txt` | `600`, `644` | [파일 권한](screenshots/03-permission-file.png) |
| `/testdir` | `700`, `755` | [폴더 권한](screenshots/04-permission-folder.png) |

**755 최종 출력:** `drwxr-xr-x` 확인. [재확인 사진](screenshots/04-permission-folder-755.png) · [권한 로그](logs/verified/02-permissions.txt)

## 5. Docker 기본 실행

```powershell
docker --version
docker info
docker pull ubuntu
docker run hello-world
docker run -it --name ubuntu-practice ubuntu bash
docker images
docker ps -a
docker run -d --name ubuntu-running ubuntu sleep infinity
docker exec ubuntu-running echo Hello-from-exec
docker stop ubuntu-running
docker start ubuntu-running
docker logs ubuntu-running
docker stats --no-stream
```

**확인:** Docker Linux 엔진, `Hello from Docker!` 출력, Ubuntu 내부 명령 실행, 컨테이너 실행·중지·재시작 확인.

**사진·로그:** [Docker 정보](screenshots/05-docker-client.png) · [서버 정보](screenshots/05-docker-server.png) · [Ubuntu 실습](screenshots/07-ubuntu.png) · [컨테이너 관리](screenshots/08-containers.png) · [실행 로그](logs/verified/04-containers.txt)

## 6. Dockerfile · 웹 서버

**Dockerfile**

```dockerfile
FROM nginx:alpine
COPY app/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

**빌드 및 실행**

```powershell
docker build -t codyssey-web:v1 .
docker run -d --name codyssey-web -p 8080:80 codyssey-web:v1
docker ps
docker logs codyssey-web
curl.exe http://localhost:8080
```

**확인:** 이미지 생성, `8080:80` 포트 연결, 직접 작성한 HTML 표시, `GET /` 요청 HTTP `200`.

**사진·로그:** [HTML](screenshots/09-html-code.png) · [Dockerfile](screenshots/10-dockerfile.png) · [빌드](screenshots/11-web-build.png) · [웹페이지](screenshots/12-browser.png) · [운영 화면](screenshots/13-web-operations.png) · [로그](logs/verified/06-webserver.txt)

## 7. 바인드 마운트

PC의 `app` 폴더를 컨테이너 웹 폴더에 연결. PC에서 HTML 수정 후 `8081` 웹페이지 반영 확인.

```powershell
$appPath = (Resolve-Path .\app).Path
docker run -d --name codyssey-bind -p 8081:80 --mount "type=bind,source=$appPath,target=/usr/share/nginx/html,readonly" codyssey-web:v1
curl.exe http://localhost:8081
docker inspect codyssey-bind --format '{{json .Mounts}}'
```

**확인:** [변경 전](screenshots/14-bind-before.png) → [변경 후](screenshots/15-bind-after.png). `readonly`는 컨테이너 쪽 쓰기 제한이며, PC에서 파일을 수정하면 반영됨.

**로그:** [바인드 마운트 로그](logs/verified/07-bind.txt). 사진과 로그의 문구는 확인 시점에 따라 다름.

## 8. Docker 볼륨 · 데이터 보존

```powershell
docker volume create codyssey-data
docker volume ls
docker run -d --name volume-first --mount type=volume,source=codyssey-data,target=/data ubuntu sleep infinity
docker exec volume-first sh -c "echo SAVE-OK > /data/check.txt"
docker exec volume-first cat /data/check.txt
docker stop volume-first
docker rm volume-first
docker run --rm --mount type=volume,source=codyssey-data,target=/data ubuntu cat /data/check.txt
docker volume inspect codyssey-data
```

**확인:** 컨테이너 삭제 전·후 모두 `SAVE-OK` 출력. 볼륨 데이터 유지.

**사진·로그:** [삭제 전](screenshots/16-volume-before.png) · [삭제 후](screenshots/17-volume-after.png) · [검증 로그](logs/verified/08-volume.txt)

## 9. Git · GitHub

```powershell
git --version
git status
git remote -v
git log -1 --oneline
git push origin main
```

**확인:** Git 버전, 로컬 `main` 브랜치, GitHub 원격 주소, 커밋 기록, `push` 결과 및 원격 파일 목록 확인.

**파일 수정 후 업로드 순서**

```powershell
git add README.md logs/troubleshooting.md
git commit -m "Update workstation documentation"
git push origin main
```

**사진:** [Git 버전](screenshots/23-git-version.png) · [브랜치](screenshots/20-git.png) · [원격 주소](screenshots/22-github-remote.png) · [Push 결과](screenshots/git-push-success.png) · [GitHub 파일 목록](screenshots/github-file-list.png)

## 10. 오류 해결

| 오류 | 원인 | 해결 | 사진 |
|---|---|---|---|
| `No such container` | 컨테이너 이름 오타 | `docker ps -a`로 이름 확인 후 로그 재실행 | [사진](screenshots/18-error-name.png) |
| `No such file or directory` | NGINX 내부 경로 오타 | `/usr/share/nginx/html`로 경로 수정 | [사진](screenshots/19-error-path.png) |

[트러블슈팅 상세 기록](logs/troubleshooting.md) · [오류 실행 로그](logs/verified/09-troubleshooting.txt)

## 11. 파일 구성

```text
Dockerfile                NGINX 커스텀 이미지 설정
app/index.html            웹페이지
practice/                 CLI 실습 파일
logs/verified/            명령어 실행 기록
logs/troubleshooting.md   오류 2건 해결 기록
screenshots/              실습 사진
README.md                 실습 요약 및 재현 방법
```
