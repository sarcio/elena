
## Docker 시작하기 

### 서버 환경 준비
Docker를 사용하는 환경을 준비합니다. 
여기서는 AWS 환경(Amazon Linux 2)에 Docker를 설치하는 방법을 설명합니다.

### 도커 환경 구성
yum 리포지터리 설정을 통해 Docker CE를 설정합니다. 

#### 도커를 설치하기 위한 리포지토리 설정

우선 yum-util과 devicemapper 패키지를 설치합니다. 
(Amazon Linux 2는 이미 설치되어 잇습니다) 

```bash
# yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

Docker CE를 설치하기 위해 Docker CE yum 리포지토리를 추가합니다. 

```bash
# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo

# ll /etc/yum.repos.d/
total 12
-rw-r--r-- 1 root root  982 Jun 26 00:04 amzn2-core.repo
-rw-r--r-- 1 root root  763 Aug 11 02:03 amzn2-extras.repo
-rw-r--r-- 1 root root 2424 Aug 23 20:27 docker-ce.repo

```
docker-ce.repo가 추가되어 있응 것을 확인할 수 있습니다. 

#### 도커 설치

Docker를 설치합니다. 

```bash
# yum install docker-ce

Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                                                            | 2.4 kB  00:00:00
docker-ce-stable                                                                                                      | 2.9 kB  00:00:00
docker-ce-stable/x86_64/primary_db                                                                                    |  15 kB  00:00:00
Resolving Dependencies
--> Running transaction check
---> Package docker-ce.x86_64 0:18.06.1.ce-3.el7 will be installed
--> Processing Dependency: container-selinux >= 2.9 for package: docker-ce-18.06.1.ce-3.el7.x86_64
--> Processing Dependency: libcgroup for package: docker-ce-18.06.1.ce-3.el7.x86_64
--> Processing Dependency: libltdl.so.7()(64bit) for package: docker-ce-18.06.1.ce-3.el7.x86_64
--> Running transaction check
---> Package docker-ce.x86_64 0:18.06.1.ce-3.el7 will be installed
--> Processing Dependency: container-selinux >= 2.9 for package: docker-ce-18.06.1.ce-3.el7.x86_64
---> Package libcgroup.x86_64 0:0.41-13.amzn2 will be installed
---> Package libtool-ltdl.x86_64 0:2.4.2-22.2.amzn2.0.2 will be installed
--> Finished Dependency Resolution
Error: Package: docker-ce-18.06.1.ce-3.el7.x86_64 (docker-ce-stable)
           Requires: container-selinux >= 2.9
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest

```

Requires: container-selinux >= 2.9

container-selinux 패키지 버전이 낮아서 에러가 발생했습니다. 

container-selinux 최신 패키지를 설치하려면 CentOS 리포지토리에서 rpm 패키지를 내려받아 설치합니다. 
- CentOS 리포지토리 : http://mirror.centos.org/centos/7/extras/x86_64/Packages/ 
- rpm 파일 내려 받아 설치 하기 : container-selinux-2.42-1.gitad8f0f7.el7.noarch.rpm 패키지 설치 
(2.42 상위 버전은 추가적으로 의존성 패키지를 설치해야 되서 2.42 버전을 설치함)

```bash
# sudo yum install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.42-1.gitad8f0f7.el7.noarch.rpm
```

다시 Docker를 설치합니다. 

```bash
# yum install docker-ce
```

이제 Docker CE버전이 설치됐습니다. 다음 명령어를 사용해  설치 여부를 확인하빈다. 

```bash
# yum list installed | grep docker-ce
docker-ce.x86_64                      18.06.1.ce-3.el7               @docker-ce-stable

```

#### 도커 서비스 시작하기

Docker 서비스를 시작합니다. 

```bash
# systemctl start docker
```

Docker 서비스를 시작 여부를 확인합니다.  

```bash
# systemctl status docker
```

다음 명령어를 사용해 서비스 자동실행설정을 하면 머신 재기동시 Docker 서비스가 자동으로 시작합니다.

```bash
# systemctl enable docker
```

다음 명령어를 사용해 Docker 아

```bash
# docker info
```

### 도커 컨테이너 기본 사용법
공개되어 있는 도커 이미지를 사용해 컨테이너의 사용법을 알아봅니다. 
우선 공개된 도커 이미지를 내려받아 컨테이너를 기동하고 기동 여부가 확인되면 컨테이너를 정지, 삭제, 도커 이미지 삭제를 하여 기본적인 도커 이미지와 컨테이너의 라이프사이클을 살펴봅니다. 

#### 초기 상태 확인
다음 명령어로 로컬 도커 이미지의 현황을 확이할 수 있습니다.
```bash
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```
현재는 헤더만 출력되고 아무것도 나타나지 않습니다. 
[Doker Hub](https://hub.docker.com/explore/)라고 하는 공개된 Docker 이미지를 관리하는 저장서에서 내려받거나 자체 도커 이미지를 작성한 현황이 나타납니다. 

다음 명령어를 사용하면 로컬 환경의 도커 컨테이너의 현황을 확일 할 수 있습니다. (도커 컨테이너란 도커 이미지를 기반으로 기동된 컨테이너를 말합니다)
```bash
# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
아직 컨테니너를 기동하지 않았으므로 아무것도 표시되지 않습니다. 

#### hello-world 이미지를 사용해 컨테이너를 기동하기
다음 명령어를 사용해 공개되어 있는 hello-world 라는 도커 이미지를 내려받아 컨테이너를 기동시켜봅니다. 
```bash
# docker run hello-world
Unable to find image 'hello-world:latest' locally ・・・①
latest: Pulling from library/hello-world          ・・・②
9db2ca6ccae0: Pull complete
Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc
Status: Downloaded newer image for hello-world:latest

Hello from Docker!   ・・・③
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

```

결과를 보니 이미지를 다운로드 하고 'Hello form Docker!'와 추가적인 메시지가 출력된 것을 확인할 수 있습니다. 

'docker run' 명령어는 도커 이미지를 사용해 컨테이너를 기동하는 명령어입니다. 예제에서는 'hello-world'라는 도커 이미지를 사용해 컨테이너를 기동하고 있습니다. 
하지만 ①에서 출력된 결과를 보면 최초 로컬 환경에는 도커 이미지가 없었습니다. 따라서 Docker Hub를 검색하여 해당 이미지가 있으면 이미지를 내려받습니다. 그것이 ② 입니다.
hello-world 이미지 다운로드를 완료하면 해당 이미지를 사용해 컨테이너를 기동합니다. 이 hello-world 이미지는 컨테이너를 기동하면 컨테이너 내에서 ③에 해당하는 이미지를 출력하는 도커 이미지입니다. 

다음 명령어를 실행해 도커 이미지를 내려받았는지 확인합니다. 
```bash
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              2cb0d9787c4d        6 weeks ago         1.85kB

```
이번 출력 결과에는 hello-world라는 항목이 보입니다. 이것은 조금 전에 docker run 명령어를 실행해 Docker Hub로 부터 다운로드한 도커 이미지입니다. 이번 예제 docker run 명령어를 실행해 도커 이미지를 다운로드하고 컨테이너를 기동했지만 아래 명령어를 사용해 도커 이미지만 다운로드할 수도 있습니다. 
```bash
# docker pull hello-world
```
다음오로 컨테이너의 상태를 확인해 봅시다. 

```bash
# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                   PORTS               NAMES
1500a8204af5        hello-world         "/hello"            2 hours ago         Exited (0) 2 hours ago                       competent_haibt
```
출력 결과를 보면 왼쪽부터 컨테이너ID, 사용한 도커 이미지, 컨테이너 기동 시에 컨테이너 내에서 시행되는 명령어, 컨테이너를 생성한 시점, 컨테이너 상태(기동중, 정지중 등), 포트 포워드 설정, 컨테이너 이름입니다. 
COMMAND 항목의 '/hello'라는 파일로 ③의 내용을 출력합니다. 
컨테이너 이름은 특별히 지정하지 않았다면 자동으로 설정됩니다. 

또한 컨테이너 상태는 'Exited'로 되어 있어 정지된 상태입니다. (기동중에는 'Up'으로 표시됩니다)
컨테이너는 1개의 프로세스는 실행되고 있어야 하며 이 'hello-world' 이미지에서는 메시지를 출력하기만 하는 프로세스로 프로세스가 종료되는 시점에 컨테이너는 종료됩니다. 


#### 도커 컨테이너, 이미지 삭제하기
이번에는 생성한 도커 컨테이너와 내려받은 도커 이미지를 삭제해 봅시다.
다음 명령어를 사용해 hello-world 컨테이너를 삭자할 수 있습니다. 

```bash
# docker rm <Container Name> (ex. competent_haibt)
or
# docker rm <Container ID> (ex. 1500a8204af5)
```

다음 명령어를 실행해 컨테이너가 삭제된 것을 확인합니다. 

```bash
# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

컨테이너를 삭제하더라도 도커 이미지는 삭제되지 않습니다. 도커 이미지 삭제는 다음 명령어를 사용합니다.

```bash
# docker rmi <Docker Image Name> (ex. hello-world)
or
# docker rmi <Image ID> (ex. 2cb0d9787c4d)
```

다음 명령어를 실행해 이미지가 삭제된 것을 확인합니다. 

```bash
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

#### Nginx 이미지를 사용해 컨테이너 시작하기
다음으로 Nginx 이미지를 사용해봅니다. 
우선 Nginx의 도커 이미지를 내려받습니다. 

```bash
# docker pull nginx
```

내려받은 Nginx 이미지를 사용해 컨테이너를 실행합니다. 

```bash
# docker run -d --name nginx-container -p 8181:80 nginx
```

이제 nginx 컨테이너가 기동됐으므로 8181 포트를 외부에서 접속할 수 있도록 방화벽 설정이 되어 있으면 접속할 수 있습니다. 
여기서는 curl 명령어를 사용해 nginx 실행 여부를 확인해봅니다. 

```bash
# curl -v http://127.0.0.1:8181/

*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8181 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:8181
> User-Agent: curl/7.55.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.15.2
< Date: Wed, 29 Aug 2018 07:00:31 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 24 Jul 2018 13:02:29 GMT
< Connection: keep-alive
< ETag: "5b572365-264"
< Accept-Ranges: bytes
<
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host 127.0.0.1 left intact

```

nginx가 정상적으로 실행된 것을 확인할 수 있습니다. 

다음 컨테이너 실행 옵션입니다. 
옵션 | 설명
------------ | -------------
-d | 컨테이너를 백그라운드로 실행합니다. 이 옵션을 추가하지 않으면 컨테이너 기동 시 실행되는 명령어를 실행한 상태로 남아 있습니다. 예를 들어 명령어 콘솔 출력이 표시된 상태가 되기도 합니다. 
-name | 컨테이너 이름을 지정합니다. (지정하지 않는 경우 자동으로 이름이 지정됩니다)
-p | 호스트와 컨테이너간 포트 포워딩 설정. 기본적으로는 '-p <호스트의 포트>:<컨테이너의 포트>' 형식이며 호스트의 포트를 생략하면 자동으로 설정됩니다. 컨테이너는 도커로부터 생성되는 네트워크이기 때문에 이 옵션을 사용하지 않으면 호스트의 IP주소를 통해 컨테이너에서 사용하고 있는 포트에 접속할 수 없습니다.

또한 nginx 이미지는 컨테이너 기동 시에 nginx 프로세스가 실행되기 때문에 컨테이너도 정지하지 않고 실행된 상태가 됩니다. docker ps 명령어로 확인해 보면 STATUS가 'Up'으로 표시됩니다. 

```bash
# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
67ad9ba2b087        nginx               "nginx -g 'daemon of…"   15 minutes ago      Up 14 minutes       0.0.0.0:8181->80/tcp   nginx-container
```

컨테이너를 정리하려면 다음 명령어를 실행합니다. 

```bash
# docker stop nginx-container
nginx-container
# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
67ad9ba2b087        nginx               "nginx -g 'daemon of…"   16 minutes ago      Exited (0) 2 seconds ago                       nginx-container
```

삭제하는 방법은 hello-world와 동일합니다. 

```bash
# docker rm 67ad9ba2b087
67ad9ba2b087
# docker rmi c82521676580
...
Deleted: ...
```



