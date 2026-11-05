트러블슈팅 기록

## 문제 1. 컨테이너 이름을 잘못 입력함

### 1. 문제 상황

Docker 컨테이너의 로그를 확인하는 과정에서 실제 컨테이너 이름과 다르게 입력하여 오류를 재현했다.

**잘못 입력한 명령어**

```bash
docker logs codyssey-weeb
```

**실제 오류 메시지**

```text
Error response from daemon: No such container: codyssey-weeb
```

### 2. 원인 가설

Docker가 해당 컨테이너를 찾지 못하는 것을 확인했다.

컨테이너가 삭제되었거나 이름을 잘못 입력했을 가능성이 있다고 생각했다.

### 3. 확인 과정

실제 컨테이너의 이름을 확인하기 위해 다음 명령어를 실행했다.

```bash
docker ps -a
```

**확인 결과 (관련 항목 발췌)**

```text
CONTAINER ID   IMAGE             NAMES
57989477c60c   codyssey-web:v1   codyssey-web
```

실제 컨테이너 이름이 `codyssey-weeb`이 아니라 `codyssey-web`이라는 것을 확인했다.

### 4. 해결 방법

올바른 컨테이너 이름을 사용하여 로그를 다시 확인했다.

```bash
docker logs codyssey-web
```

### 5. 최종 결과

**실제 로그 출력 중 일부**

```text
/docker-entrypoint.sh: Configuration complete; ready for start up
```

```text
"GET / HTTP/1.1" 200 615
```

컨테이너의 로그가 정상적으로 출력되었으며, 웹 서버 접속 요청에 대한 HTTP 200 응답도 확인했다.

### 6. 배운 점

Docker 명령어를 사용할 때 컨테이너 이름을 정확하게 입력하는 것이 중요하다는 것을 배웠다.

컨테이너를 찾을 수 없다는 오류가 발생하면 `docker ps -a` 명령어로 실제 컨테이너 목록과 이름을 먼저 확인해야 한다는 것을 알게 되었다.


[PS C:\Users\dlals26842658053\codyssey-workstation> docker ps -a
CONTAINER ID   IMAGE             COMMAND                   CREATED             STATUS                           PORTS                                     NAMES
3e5e03eb9e08   codyssey-web:v1   "/docker-entrypoint.…"   12 minutes ago      Up 12 minutes                    0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   codyssey-bind
57989477c60c   codyssey-web:v1   "/docker-entrypoint.…"   17 minutes ago      Up 17 minutes                    0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   codyssey-web
0d3e9886710b   ubuntu            "sleep infinity"          49 minutes ago      Up 48 minutes                                                              ubuntu-running
4f4cd868f7cd   ubuntu            "bash"                    52 minutes ago      Exited (0) 49 minutes ago                                                  ubuntu-practice
46f46c6c645a   hello-world       "/hello"                  53 minutes ago      Exited (0) 53 minutes ago                                                  cranky_leakey
78fe639571ec   ubuntu            "sleep infinity"          About an hour ago   Exited (137) About an hour ago                                             permission-lab
PS C:\Users\dlals26842658053\codyssey-workstation> docker logs codyssey-web
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/11/04 06:23:04 [notice] 1#1: using the "epoll" event method
2026/11/04 06:23:04 [notice] 1#1: nginx/1.31.8
2026/11/04 06:23:04 [notice] 1#1: built by gcc 15.2.0 (Alpine 15.2.0) 
2026/11/04 06:23:04 [notice] 1#1: OS: Linux 6.18.54.1-microsoft-standard-WSL2
2026/11/04 06:23:04 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/11/04 06:23:04 [notice] 1#1: start worker processes
2026/11/04 06:23:04 [notice] 1#1: start worker process 30
2026/11/04 06:23:04 [notice] 1#1: start worker process 31
2026/11/04 06:23:04 [notice] 1#1: start worker process 32
2026/11/04 06:23:04 [notice] 1#1: start worker process 33
2026/11/04 06:23:04 [notice] 1#1: start worker process 34
2026/11/04 06:23:04 [notice] 1#1: start worker process 35
2026/11/04 06:23:04 [notice] 1#1: start worker process 36
2026/11/04 06:23:04 [notice] 1#1: start worker process 37
2026/11/04 06:23:04 [notice] 1#1: start worker process 38
2026/11/04 06:23:04 [notice] 1#1: start worker process 39
2026/11/04 06:23:04 [notice] 1#1: start worker process 40
2026/11/04 06:23:04 [notice] 1#1: start worker process 41
2026/11/04 06:23:04 [notice] 1#1: start worker process 42
2026/11/04 06:23:04 [notice] 1#1: start worker process 43
2026/11/04 06:23:04 [notice] 1#1: start worker process 44
2026/11/04 06:23:04 [notice] 1#1: start worker process 45
2026/11/04 06:23:04 [notice] 1#1: start worker process 46
2026/11/04 06:23:04 [notice] 1#1: start worker process 47
2026/11/04 06:23:04 [notice] 1#1: start worker process 48
2026/11/04 06:23:04 [notice] 1#1: start worker process 49
2026/11/04 06:23:04 [notice] 1#1: start worker process 50
2026/11/04 06:23:04 [notice] 1#1: start worker process 51
2026/11/04 06:23:04 [notice] 1#1: start worker process 52
2026/11/04 06:23:04 [notice] 1#1: start worker process 53
172.17.0.1 - - [04/Nov/2026:06:23:18 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/156.0.0.0 Safari/537.36 Edg/156.0.0.0" "-"
172.17.0.1 - - [04/Nov/2026:06:23:18 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/156.0.0.0 Safari/537.36 Edg/156.0.0.0" "-"
2026/11/04 06:23:18 [error] 32#32: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080", referrer: "http://localhost:8080/"
172.17.0.1 - - [04/Nov/2026:06:24:04 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.21.0" "-"]





## 문제 2. 컨테이너 내부 파일 경로 입력 오류

### 1. 문제 상황

Docker 컨테이너 내부에 있는 웹 서버 파일을 확인하는 과정에서 경로를 잘못 입력하여 오류가 발생했다.

**오류 메시지를 기준으로 재구성한 잘못된 명령어**

```bash
docker exec codyssey-web ls /usr/nginx/html
```

**기록된 오류 메시지**

```text
ls: /usr/nginx/html: No such file or directory
```

### 2. 원인 가설

처음에는 파일이 없는 것으로 생각했지만, 파일 경로를 잘못 입력했을 가능성이 있다고 판단했다.

실제 경로에 `/share`가 빠져 있어 파일을 찾지 못한 것으로 추정했다.

### 3. 확인 과정

올바른 파일 경로로 다시 확인했다.

```bash
docker exec codyssey-web ls /usr/share/nginx/html
```

이 명령어로 `index.html` 파일이 존재하는 것을 확인했다.

### 4. 해결 방법

파일의 실제 내용을 확인하기 위해 다음 명령어를 실행했다.

```bash
docker exec codyssey-web cat /usr/share/nginx/html/index.html
```

### 5. 최종 결과

**실제 출력 결과 중 일부**

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Codyssey Workstation</title>
</head>
<body>
    <h1>안녕하세요! 코디세이입니다.</h1>
    <p>Docker 웹 서버 실행 성공!</p>
    <p>내가 만든 첫 번째 웹사이트</p>
</body>
</html>
```

`index.html` 파일의 내용이 정상적으로 출력되는 것을 확인했다.

### 6. 배운 점

파일을 찾을 수 없다는 오류가 발생했을 때는 파일이 실제로 없는 것인지, 경로를 잘못 입력한 것인지 먼저 확인해야 한다는 것을 배웠다.

이번 실습을 통해 컨테이너 내부의 파일 경로를 정확히 확인하는 것이 중요하다는 것을 알게 되었다.

[PS C:\Users\dlals26842658053\codyssey-workstation> docker exec codyssey-web cat /usr/share/nginx/html/index.html

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Codyssey Workstation</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #edf5ff;
            text-align: center;
            padding-top: 100px;
        }

        h1 {
            color: #17477b;
        }

        p {
            font-size: 20px;
        }
    </style>
</head>
<body>
    <h1>안녕하세요! 코디세이입니다.</h1>
    <p>Docker 웹 서버 실행 성공!</p>
    <p>내가 만든 첫 번째 웹사이트</p>
</body>
</html>]





이번 실습에서 배운 점

컨테이너 이름과 파일 위치를 정확히 확인하는 것이 중요하다는 것을 배웠다.

오류가 발생했을 때 같은 명령을 반복하기보다 실제 목록과 경로를 확인하고 수정하는 방법을 익혔다.
