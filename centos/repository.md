
## YUM 리포지토리 만들기 
Amazon 클라우드 상의 소프트웨어 배포를 위한 YUM Repository를 구축하는 방법을 설명합니다.
Yum은 레드햇 계열의 리눅스에서 소프트웨어를 배포하기 위한 방법으로 rpmbuild를 활용하여 구축을 할 경우 손쉽게 대상 서버들로 배포하는 것이 가능합니다.

### YUM repository 구축을 위한 createrepo 패키지 설치
Repository 구축은 createrepo 명령을 활용하여 만들 수 있습니다. 일반적으로 설치된 패키지에는 createrepo 패키지가 설치되어 있지 않으므로 해당 패키지를 설치 후 시스템을 구축해야 합니다.

#### 설치 환경 확인 
AWS에 Yum 리포지토리를 구성할 예정이며 설치 환경 정보는 다음과 같습니다. 

```bash
# cat /etc/*-release | uniq
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
VERSION_ID="2"
PRETTY_NAME="Amazon Linux 2"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o:amazon:amazon_linux:2"
HOME_URL="https://amazonlinux.com/"
Amazon Linux 2

# rpm -qa *-release
system-release-2-4.amzn2.x86_64

```

#### createrepo 패키지 설치

아래의 명령어를 활용하여 createrepo 패키지를 설치합니다.

```bash
# yum install createrepo
# createrepo --version
createrepo 0.9.9
```
#### yum-util 설치

yumdownload 명령어를 사용하기 위해 yum-utils 패키지를 설치합니다.

```bash
# yum install yum-utils
```


#### 아파치 웹 서버(httpd) 설치 및 기동
아파치를 설치합니다. 
```bash
# yum install httpd
```

아파치를 시작합니다. 
```bash
# systemctl start httpd
```

정상 서비스 여부를 확인합니다 
```bash
# lsof -i:80 -P
COMMAND  PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
httpd   5014   root    4u  IPv6  23534      0t0  TCP *:80 (LISTEN)
httpd   5047 apache    4u  IPv6  23534      0t0  TCP *:80 (LISTEN)
httpd   5049 apache    4u  IPv6  23534      0t0  TCP *:80 (LISTEN)
httpd   5050 apache    4u  IPv6  23534      0t0  TCP *:80 (LISTEN)
httpd   5051 apache    4u  IPv6  23534      0t0  TCP *:80 (LISTEN)
httpd   5052 apache    4u  IPv6  23534      0t0  TCP *:80 (LISTEN)

```

리포지토리 용도의 디렉터리를 생성합니다. 
```bash
# mkdir /var/www/html/repo
```

리포지토리를 생성합니다. 
```bash
]# createrepo --database /var/www/html/repo/
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete

```

리포지토리 디렉터리로 이동하고 repodata 디렉터리가 생성된 것을 확인합니다. 
```bash
# cd /var/www/html/repo/
# pwd
/var/www/html/repo
# ls -F
repodata/
```

테스트 용도 rpm 패키지를 다운로드 합니다. 
```bash
# yumdownloader --disablerepo=* --enablerepo=base bc
```




아래의 명령어를 활용하여 createrepo 패키지를 설치합니다.

```bash
# yum install createrepo
# createrepo --version
createrepo 0.9.9
```

### Repository 디렉토리 생성 및 아파치 구성
리포지토리로 사용될 디렉토리를 생성하고, 아파치를 구성하는 작업을 수행해야 합니다. 
본 문서에서 생성되는 디렉토리를 /opt/repo 라는 디렉토리를 사용합니다. 
웹 서버의 경우 아파치에 기본적으로 설치되는 패키지를 yum 명령어를 활용하여 설치합니다. 

#### Yum repository를 위한 디렉토리 생성
아래의 명령을 수행하여 디렉토리를 생성합니다
```bash
# mkdir -p /opt/repo
```

#### repo 파일 생성 
```bash
# yum install httpd
```

