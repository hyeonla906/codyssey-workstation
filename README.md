# Codyssey Development Workstation

Windows PowerShell에서 CLI·Linux 권한·Docker·Git/GitHub 실습.

## 1. 실습 결과

| 항목 | 확인 결과 | 관련 위치 |
|---|---|---|
| CLI | 파일 생성·복사·이동·삭제, 내용 출력 | [CLI](#3-cli-파일-관리) |
| Linux 권한 | 파일 `600`·`644`, 폴더 `700`·`755` | [권한](#4-linux-권한) |
| Docker | Linux 엔진, 이미지·컨테이너 목록, `hello-world` | [Docker](#5-docker-기본-점검) |
| 웹 서버 | `codyssey-web:v1` 빌드, `localhost:8080` 접속 | [웹 서버](#6-dockerfile--웹-서버) |
| 바인드 마운트 | HTML 변경 전·후 화면 비교 (`8081`) | [바인드 마운트](#7-바인드-마운트) |
| 볼륨 | 컨테이너 삭제 후 `SAVE-OK` 재확인 | [볼륨](#8-볼륨-데이터-유지) |
| Git/GitHub | `main`, `origin` 확인, `git push` 실행 | [Git](#9-gitgithub) |
| 오류 해결 | 컨테이너 이름·NGINX 파일 경로 오류 2건 | [오류 해결](#10-오류-해결) |

### 실행 환경

| 항목 | 환경 |
|---|---|
| OS / 터미널 | Windows / PowerShell 5.1 |
| 편집기 | VS Code |
| Docker | 29.9.2 / Linux 컨테이너 |
| Git | 2.56.0.windows.4 / `main` |
| 웹 서버 | `nginx:alpine` 기반 커스텀 이미지 |

### 폴더 구성

```text
Dockerfile                이미지 빌드 설정
app/index.html            웹페이지
practice/                 CLI 실습 파일
logs/verified/            터미널 실행 로그
logs/troubleshooting.md   오류 해결 과정
screenshots/              실습 사진
README.md                 실행 방법·확인 결과
```

실행 파일(`app`), 환경 설정(`Dockerfile`), 기록(`logs`), 사진(`screenshots`) 분리. 수정·확인할 위치 구분.

## 2. 다른 PC에서 실행

**준비:** Docker Desktop 설치, Linux 컨테이너 엔진 실행. 아래 명령은 PowerShell에서 실행.

```powershell
git clone https://github.com/hyeonla906/codyssey-workstation.git
cd codyssey-workstation
docker build -t codyssey-web:v1 .
docker run -d --name codyssey-web -p 8080:80 codyssey-web:v1
curl.exe http://localhost:8080
```

**확인:** 브라우저에서 `http://localhost:8080` 접속. 같은 이름의 컨테이너가 이미 있으면 `docker ps -a`로 확인 후 이름·포트 조정.

### 연결 정보

| 항목 | 연결 | 용도 |
|---|---|---|
| 웹 서버 | PC `8080` → 컨테이너 `80` | 기본 웹페이지 |
| 바인드 마운트 | PC `8081` → 컨테이너 `80` | 파일 변경 반영 |
| 폴더 연결 | PC `app` → `/usr/share/nginx/html` | HTML 공유 |
| Docker 볼륨 | `codyssey-data` → `/data` | 컨테이너 삭제 후 파일 보관 |

컨테이너는 별도 네트워크 공간(네트워크 네임스페이스)을 사용. `-p 8080:80`은 PC의 8080번 요청을 컨테이너의 80번 포트로 전달. `EXPOSE 80`만으로는 PC 포트가 열리지 않음.

## 3. CLI 파일 관리

PowerShell에서 파일 생성·내용 확인·복사·이동·삭제 실행.

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
Get-ChildItem backup
Remove-Item backup\renamed.txt
Get-ChildItem backup
cd ..\..
```

**확인:** `Get-Content hello.txt` → `Hello Codyssey`. 복사·이동 후 파일 목록 확인, 삭제 후 목록에서 제외.

![작업 폴더](screenshots/01-setup.png)

![터미널 실행 결과](screenshots/02-terminal.png)

[실행 로그](logs/verified/01-terminal.txt)

## 4. Linux 권한

Ubuntu 컨테이너에서 파일·폴더의 권한 변경 후 다시 조회.

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

| 권한 | 소유자 | 그룹 | 기타 사용자 | 표시 |
|---|---|---|---|---|
| `600` | 읽기·쓰기 | 없음 | 없음 | `-rw-------` |
| `644` | 읽기·쓰기 | 읽기 | 읽기 | `-rw-r--r--` |
| `700` | 읽기·쓰기·실행 | 없음 | 없음 | `drwx------` |
| `755` | 읽기·쓰기·실행 | 읽기·실행 | 읽기·실행 | `drwxr-xr-x` |

**권한 계산:** `r=4`, `w=2`, `x=1`. `7=4+2+1(rwx)`, `5=4+1(r-x)` → `755 = rwx / r-x / r-x`. 폴더의 `x`는 진입 권한.

**최종 확인:** `docker exec permission-lab ls -ld /testdir` → `drwxr-xr-x ... /testdir`.

![파일 권한 600·644](screenshots/03-permission-file.png)

![폴더 권한 700·755](screenshots/04-permission-folder.png)

![폴더 권한 755 최종 확인](screenshots/04-permission-folder-755.png)

[권한 실행 로그](logs/verified/02-permissions.txt)

## 5. Docker 기본 점검

### 5-1. 엔진 상태

```powershell
docker --version
docker info
```

**사진에서 확인한 출력:**

```text
Docker version 29.9.2, build a82e528
Server Version: 29.9.2
OSType: linux
```

Docker Desktop의 Linux 엔진 실행 확인.

![Docker 버전·Client 정보](screenshots/05-docker-client.png)

![Docker Server 정보·OSType linux](screenshots/05-docker-server.png)

[Docker 점검 명령 로그](logs/verified/03-docker.txt) — 이 파일에는 명령 실행 기록이 있고, 상세 출력은 위 사진에서 확인.

### 5-2. 이미지·컨테이너

```powershell
docker pull ubuntu
docker run hello-world
docker run -it --name ubuntu-practice ubuntu bash
docker images
docker ps
docker ps -a
docker run -d --name ubuntu-running ubuntu sleep infinity
docker exec ubuntu-running echo Hello-from-exec
docker stop ubuntu-running
docker start ubuntu-running
```

**확인:** `docker run hello-world` → `Hello from Docker!`. 이미지 목록 사진의 출력 일부:

```text
hello-world:latest
ubuntu:latest
```

`docker ps -a`에서 `ubuntu-practice` 등 중지된 컨테이너까지 확인.

**차이:** 이미지는 실행 환경의 저장본. 컨테이너는 이미지를 실행한 작업 공간. `docker build`로 이미지 생성, `docker run`으로 컨테이너 실행.

![hello-world 실행 결과](screenshots/06-hello-world.png)

![Ubuntu 내부 명령 실행](screenshots/07-ubuntu.png)

![docker images 및 docker ps -a 출력](screenshots/08-containers.png)

[컨테이너 실행 로그](logs/verified/04-containers.txt) — 이미지·컨테이너 목록 출력은 위 사진에서 확인.

## 6. Dockerfile · 웹 서버

**Dockerfile**

```dockerfile
FROM nginx:alpine
COPY app/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

- `FROM`: 기본 이미지 선택
- `COPY`: HTML 파일을 이미지에 복사
- `EXPOSE`: 컨테이너 내부 포트 표시

**빌드·실행**

```powershell
docker build -t codyssey-web:v1 .
docker images
docker run -d --name codyssey-web -p 8080:80 codyssey-web:v1
docker ps
docker logs codyssey-web
curl.exe http://localhost:8080
```

**빌드 로그 확인:** `naming to docker.io/library/codyssey-web:v1` → `unpacking to docker.io/library/codyssey-web:v1`. 이미지 생성 완료 기록.

**웹 서버 확인:** `localhost:8080` 접속, HTML 표시, NGINX `GET /` HTTP `200` 확인.

![HTML 작성](screenshots/09-html-code.png)

![Dockerfile](screenshots/10-dockerfile.png)

![이미지 빌드 출력](screenshots/11-web-build.png)

![8080 웹페이지 접속](screenshots/12-browser.png)

![컨테이너 실행·로그](screenshots/13-web-operations.png)

[웹 서버 빌드·실행 로그](logs/verified/06-webserver.txt)

## 7. 바인드 마운트

PC의 `app` 폴더를 컨테이너의 NGINX 웹 폴더에 연결. HTML 변경 전·후 비교.

```powershell
$appPath = (Resolve-Path .\app).Path
docker run -d --name codyssey-bind -p 8081:80 --mount "type=bind,source=$appPath,target=/usr/share/nginx/html,readonly" codyssey-web:v1
curl.exe http://localhost:8081
docker inspect codyssey-bind --format '{{json .Mounts}}'
```

**확인:** `localhost:8081`에서 PC의 HTML 수정 전·후 내용 반영. `readonly`는 컨테이너 내부 쓰기 제한.

**경로 구분:** `app/index.html`은 프로젝트 기준 상대경로. `$appPath`는 `Resolve-Path`로 구한 PC의 절대경로로, 마운트 원본 폴더를 명확히 지정. `/usr/share/nginx/html`은 컨테이너 내부 절대경로.

![바인드 마운트 변경 전](screenshots/14-bind-before.png)

![바인드 마운트 변경 후](screenshots/15-bind-after.png)

[바인드 마운트 로그](logs/verified/07-bind.txt) — 사진과 로그의 문구는 기록 시점에 따라 다름.

## 8. 볼륨 데이터 유지

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

**확인:** `volume-first` 삭제 후 새 컨테이너에서 `SAVE-OK` 출력. 컨테이너를 지워도 볼륨 파일 유지.

**볼륨 정보:** `Name: codyssey-data`, `Driver: local`, `Mountpoint: /var/lib/docker/volumes/codyssey-data/_data` (`docker volume inspect` 기록).

| 방식 | 장점 | 주의 사항 |
|---|---|---|
| 바인드 마운트 | PC 파일 변경을 컨테이너에서 즉시 확인 | PC 폴더 경로에 영향받음 |
| Docker 볼륨 | 컨테이너 삭제 후에도 데이터 보관 | PC에서 파일을 직접 수정하기는 불편 |

![볼륨 삭제 전](screenshots/16-volume-before.png)

![컨테이너 삭제 후 볼륨 확인](screenshots/17-volume-after.png)

[볼륨 실습 로그](logs/verified/08-volume.txt)

## 9. Git/GitHub

```powershell
git --version
git status
git remote -v
git log -1 --oneline
git push origin main
```

**확인:** Git `2.56.0.windows.4`, `main` 브랜치, `origin` 원격 주소. `git push origin main` 실행 및 GitHub 동기화 상태 확인.

```text
origin  https://github.com/hyeonla906/codyssey-workstation.git (fetch)
origin  https://github.com/hyeonla906/codyssey-workstation.git (push)
```

**Git 작업 순서:** `git status` → `git add` → `git commit` → `git push`. `commit`은 로컬 변경 기록, `push`는 원격 저장소 반영.

![Git 버전](screenshots/23-git-version.png)

![현재 브랜치](screenshots/20-git.png)

![git remote -v 출력](screenshots/22-github-remote.png)

![git push 실행 화면](screenshots/25-git-push-terminal.png)

`git remote -v` 실행 결과는 위 사진에서 확인.

**Git 사용자 설정 확인:** `git config --list` 또는 `git config --get user.name`, `git config --get user.email`. 현재 제공된 사진·로그에는 이 출력이 없어 추가 캡처 필요. 공개 자료에 이메일·토큰 등 개인정보 노출 주의.

## 10. 오류 해결

| 발생 | 원인 가설 | 확인 | 조치·결과 |
|---|---|---|---|
| `No such container: codyssey-weeb` | 컨테이너 삭제 또는 이름 오타 | `docker ps -a`로 실제 이름 `codyssey-web` 확인 | `docker logs codyssey-web` → NGINX 로그 확인 |
| `No such file or directory` | NGINX 내부 폴더 경로 오타 | `/usr/share/nginx/html` 확인 | `docker exec codyssey-web cat /usr/share/nginx/html/index.html` → HTML 출력 |

![컨테이너 이름 오타 수정](screenshots/18-error-name.png)

![NGINX 내부 파일 경로 수정](screenshots/19-error-path.png)

[오류 해결 기록](logs/troubleshooting.md) · [실행 로그](logs/verified/09-troubleshooting.txt)

## 11. 포트 접속 문제 확인

웹페이지가 열리지 않을 때 PowerShell에서 순서대로 확인.

```powershell
docker ps
curl.exe http://localhost:8080
Get-NetTCPConnection -LocalPort 8080 -State Listen -ErrorAction SilentlyContinue
```

- `docker ps`: 웹 서버 컨테이너와 포트(`8080->80`) 확인
- `curl.exe`: 웹 서버 응답 확인
- `Get-NetTCPConnection`: PC의 8080번 포트를 사용 중인 프로세스 ID(`OwningProcess`) 확인
- 포트 충돌 시 `OwningProcess` 값으로 사용 프로그램 확인: `Get-Process -Id 1234` (`1234`를 조회한 PID로 변경)
- 사용 중인 Docker 컨테이너가 원인이면 `docker stop codyssey-web`로 중지. 다른 프로그램이면 해당 프로그램에서 종료하거나 PC 포트 변경

**대체 포트 실행 예시(미실행):**

```powershell
docker run -d --name codyssey-web-alt -p 8082:80 codyssey-web:v1
```
