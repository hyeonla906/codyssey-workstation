# Codyssey Development Workstation

Windows PowerShell에서 CLI, Linux 권한, Docker, Git/GitHub 실습.

## 1. 환경 및 실행

| 항목 | 실습 환경 |
|---|---|
| OS / 터미널 | Windows / PowerShell 5.1 |
| 편집기 | VS Code |
| Docker | 29.9.2 / Linux 컨테이너 |
| Git | 2.56.0.windows.4 / `main` |

**웹 서버 실행** — Docker Desktop 실행 후 PowerShell에서 입력.

```powershell
git clone https://github.com/hyeonla906/codyssey-workstation.git
cd codyssey-workstation
docker build -t codyssey-web:v1 .
docker run -d --name codyssey-web -p 8080:80 codyssey-web:v1
curl.exe http://localhost:8080
```

**결과:** `http://localhost:8080`에서 HTML 응답 확인.

**연결:** PC `8080` → 컨테이너 `80` / 바인드 마운트는 PC `8081` → 컨테이너 `80`.

## 2. CLI 파일 관리

**실행:** 파일 생성 → 내용 확인 → 복사 → 이름 변경 → 이동 → 삭제.

```powershell
mkdir practice\lesson1
cd practice\lesson1
"Hello Codyssey" | Set-Content hello.txt
Get-Content hello.txt
Copy-Item hello.txt copy.txt
Rename-Item copy.txt renamed.txt
mkdir backup
Move-Item renamed.txt backup
Get-ChildItem backup
Remove-Item backup\renamed.txt
Get-ChildItem backup
cd ..\..
```

**결과:** `Hello Codyssey` 출력, 이동·삭제 전후 파일 목록 확인.

![CLI 실행 결과](screenshots/02-terminal.png)

[터미널 전체 기록](logs/verified/01-terminal.txt)

## 3. Linux 파일·폴더 권한

**실행:** Ubuntu 컨테이너에서 권한 변경 후 `ls`로 확인.

```powershell
docker run -d --name permission-lab ubuntu sleep infinity
docker exec permission-lab touch /test.txt
docker exec permission-lab chmod 600 /test.txt
docker exec permission-lab ls -l /test.txt
docker exec permission-lab chmod 644 /test.txt
docker exec permission-lab ls -l /test.txt
docker exec permission-lab mkdir /testdir
docker exec permission-lab chmod 700 /testdir
docker exec permission-lab ls -ld /testdir
docker exec permission-lab chmod 755 /testdir
docker exec permission-lab ls -ld /testdir
```

**권한:** `r=4`, `w=2`, `x=1`. `755` = 소유자 `rwx`, 그룹·다른 사용자 `r-x`.

**결과:** 파일 `600 → 644`, 폴더 `700 → 755` (`drwxr-xr-x`).

![파일 권한 변경](screenshots/03-permission-file.png)

![폴더 755 확인](screenshots/04-permission-folder-755.png)

[권한 실행 기록](logs/verified/02-permissions.txt)

## 4. Docker 기본 점검

```powershell
docker --version
docker info
docker run hello-world
docker run -it --name ubuntu-practice ubuntu bash
docker images
docker ps -a
```

**확인:** Docker `29.9.2`, Server `OSType: linux`, `hello-world` 실행, 이미지·컨테이너 목록.

**차이:** 이미지 = 실행에 필요한 파일 묶음 / 컨테이너 = 이미지를 실행한 작업 공간.

![Docker Server 상태](screenshots/05-docker-server.png)

![hello-world 실행](screenshots/06-hello-world.png)

![Ubuntu 컨테이너 실습](screenshots/07-ubuntu.png)

![이미지·컨테이너 목록](screenshots/08-containers.png)

[Docker 점검 기록](logs/verified/03-docker.txt) · [컨테이너 실행 기록](logs/verified/04-containers.txt)

## 5. Dockerfile 및 웹 서버

**Dockerfile** — `nginx:alpine`에 HTML 파일 추가.

```dockerfile
FROM nginx:alpine
COPY app/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

`FROM` = 기본 이미지 / `COPY` = HTML 복사 / `EXPOSE` = 내부 사용 포트 표시.

```powershell
docker build -t codyssey-web:v1 .
docker run -d --name codyssey-web -p 8080:80 codyssey-web:v1
docker ps
docker logs codyssey-web
curl.exe http://localhost:8080
```

**결과:** `codyssey-web:v1` 이미지 빌드, 웹페이지 응답, NGINX 요청 로그에서 HTTP `200` 확인.

**포트:** 컨테이너는 별도 네트워크 공간 사용. `-p 8080:80`으로 PC에서 접속 가능. `EXPOSE 80`만으로는 포트가 연결되지 않음.

![이미지 빌드](screenshots/11-web-build.png)

![8080 웹페이지](screenshots/12-browser.png)

[웹 서버 실행 기록](logs/verified/06-webserver.txt)

## 6. 바인드 마운트

**실행:** PC의 `app` 폴더를 컨테이너 웹 폴더에 연결.

```powershell
$appPath = (Resolve-Path .\app).Path
docker run -d --name codyssey-bind -p 8081:80 --mount "type=bind,source=$appPath,target=/usr/share/nginx/html,readonly" codyssey-web:v1
curl.exe http://localhost:8081
docker inspect codyssey-bind --format '{{json .Mounts}}'
```

**결과:** PC의 HTML 변경 전·후 내용이 `localhost:8081`에 반영.

**경로:** `app`은 프로젝트 기준 상대경로. `$appPath`는 PC 절대경로. `/usr/share/nginx/html`은 컨테이너 내부 경로. `readonly`는 컨테이너 안에서 쓰기 제한.

![변경 전](screenshots/14-bind-before.png)

![변경 후](screenshots/15-bind-after.png)

[바인드 마운트 기록](logs/verified/07-bind.txt)

## 7. Docker 볼륨: 컨테이너 삭제 전·후

```powershell
docker volume create codyssey-data
docker run -d --name volume-first --mount type=volume,source=codyssey-data,target=/data ubuntu sleep infinity
docker exec volume-first sh -c "echo SAVE-OK > /data/check.txt"
docker exec volume-first cat /data/check.txt
docker stop volume-first
docker rm volume-first
docker run --rm --mount type=volume,source=codyssey-data,target=/data ubuntu cat /data/check.txt
docker volume inspect codyssey-data
```

**결과:** `volume-first` 삭제 후 새 컨테이너에서 `SAVE-OK` 재출력. 볼륨 `codyssey-data` 유지.

**저장 위치 확인:** `docker volume inspect` → `/var/lib/docker/volumes/codyssey-data/_data`.

**차이:** 바인드 마운트는 PC 파일을 직접 연결 / 볼륨은 Docker가 데이터를 별도로 관리.

![컨테이너 삭제 전](screenshots/16-volume-before.png)

![컨테이너 삭제 후 데이터 확인](screenshots/17-volume-after.png)

[볼륨 실행 기록](logs/verified/08-volume.txt)

## 8. Git / GitHub

```powershell
git status
git remote -v
git config --list | Select-String '^(user\.name|user\.email)='
git log -1 --oneline
git push origin main
```

**확인:** `main` 브랜치, `origin` 원격 주소, 사용자 이름·이메일 설정, GitHub 업로드.

**구분:** `git add` = 변경 파일 선택 / `git commit` = 로컬 기록 저장 / `git push` = GitHub 반영.

![원격 저장소 확인](screenshots/22-github-remote.png)

![Git 사용자 설정 확인](screenshots/24-git-config.png)

![GitHub Push 결과](screenshots/25-git-push-terminal.png)

## 9. 오류 해결

| 발생 | 원인 | 확인·수정 |
|---|---|---|
| `No such container: codyssey-weeb` | 컨테이너 이름 오타 | `docker ps -a` → `docker logs codyssey-web` |
| `No such file or directory` | NGINX 경로의 `share` 누락 | `/usr/share/nginx/html/index.html` 경로로 재확인 |

![컨테이너 이름 오류](screenshots/18-error-name.png)

![NGINX 경로 오류](screenshots/19-error-path.png)

[오류 해결 상세 기록](logs/troubleshooting.md) · [실행 기록](logs/verified/09-troubleshooting.txt)

## 10. 접속 오류 점검

**점검 순서:** 컨테이너 실행 → HTTP 응답 → 포트 사용 확인.

```powershell
docker ps
curl.exe http://localhost:8080
Get-NetTCPConnection -LocalPort 8080 -State Listen -ErrorAction SilentlyContinue
```

**포트 충돌:** `OwningProcess`의 PID로 `Get-Process -Id 1234` 확인 (`1234`는 조회된 PID로 변경). 기존 프로그램 확인 후 컨테이너 중지 또는 PC 포트 변경.

## 파일 구성

```text
Dockerfile                웹 서버 이미지 설정
app/index.html            웹페이지
practice/                 CLI 실습 파일
logs/verified/            실행 기록
logs/troubleshooting.md   오류 해결 내용
screenshots/              실행 사진
README.md                 실습 요약
```
