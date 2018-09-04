
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

아파치 웹 서버(httpd) 패키지와 yumdownload 명령어를 사용하기 위해 yum-utils 패키지를 설치합니다.
```bash
# yum install httpd
```
```bash
# yum install yum-utils
```

아래의 명령어를 활용하여 createrepo.noarch 패키지를 설치합니다.

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

#### 아파치 설치 및 디렉터리 연결
```bash
# yum install httpd
```

우선 yum-util과 devicemapper 패키지를 설치합니다. 
(Amazon Linux 2는 이미 설치되어 있습니다) 

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
