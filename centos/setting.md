
## CentOS 기본 설정  

### 보안 설정

#### 원격 root 로그인 접속 제한하기 
기본적인 ssh 설정에는 root 로그인이 허용되며 원격에서 root 로그인을 허용하면 악의적인 목적으로 root 계정으로 ssh 접속을 시도할 수 있다.
따라서 기본적으로 사용자 IP 기반으로 접속을 제한하거나 root 계정에 대한 원격 접속을 제한해야 한다. 

다음은 서버에서 root 계정으로 ssh 로그인을 제안하는 방법이다. 
```bash
# vi /etc/ssh/sshd_config
...
#PermitRootLogin yes
...
(수정)
...
PermitRootLogin no
...

```


#### sudo 권한 설정(sudo user 생성하기) 
CentOS는 기본적으로 sudo 권한은 root만 가지고 있다. 
따라서 root 계정을 사용하지 못하게 하면서 root 권한을 주기 위해서는 sudo 명령어를 사용할 수 있는 권한을 부여하면 된다. 

기본적으로 sudo 설정파일의 권한은 440으로 되어 있다. 
```bash
# ls -al /etc/sudoers
-r--r-----. 1 root root 4000 Jan 15  2014 /etc/sudoers
```
CentOS에서는 기본적으로 wheel group 멤버가 sudo 권한을 갖고 있다. 
그러므로 사용자(username 계정)를 wheel group에 추가해 sudo 권한을 가질 수 있다.
또한 NOPASSWD를 설정해 패스워드 입력을 생략할 수 있다.
```bash
# usermod -aG whell username

#vi /etc/sudoers
...
## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
...
(수정)
...
## Same thing without a password
%wheel        ALL=(ALL)       NOPASSWD: ALL
...

```
다른 방법으로는 sudo 설정파일에 사용자를 추가하거나 그룹에 권한을 줄 수 있다.  
```bash
#vi /etc/sudoers
...
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
...
(수정)
...
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
opdc    ALL=(ALL)       ALL
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
%opdc   ALL=(ALL)       ALL
## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
opdc    ALL=(ALL)       NOPASSWD: ALL
...

```

#### SSH 서비스 포트 변경하기 

SSH는 기본 포트로 22번 포트를 사용하고 있으며 서버 보안을 위해 포트를 변경해서 사용할 수 있습니다. 
SSH 포트 변경 후에는 방화벽 등으로 접속이 안될 수 있으므로 주의하기 바랍니다. 

다음과 같이 현재 사용하고 있는 SSH 포트를 확인할 수 있습니다. 
```bash
# netstat -anp | grep LISTEN | grep sshd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2483/sshd
tcp6       0      0 :::22                   :::*                    LISTEN      2483/sshd

# cat /etc/ssh/sshd_config | egrep ^\#?Port
#Port 22
```

다음과 같이 SSH 포트를 변경할 수 있습니다.

```bash
# vi /etc/ssh/sshd_config
...
#Port 22
...
(수정)
...
#Port 22
Port 2221
...
```
변경된 설정 반영하려면 sshd를 재시작해야 합니다. 
```bash
# service sshd restart
Redirecting to /bin/systemctl restart  sshd.service

# netstat -anp | grep LISTEN | grep sshd
tcp        0      0 0.0.0.0:2221            0.0.0.0:*               LISTEN      27414/sshd
tcp6       0      0 :::2221                 :::*                    LISTEN      27414/sshd

```

#### CentOS7 방화벽 관리하기 (firewalld) 

```bash
# firewall-cmd --state
running

```


```bash
# firewall-cmd --permanent --zone=public --add-port=8080/tcp
# firewall-cmd --reload
```

```bash
# cat /etc/firewalld/zones/public.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="dhcpv6-client"/>
  <service name="ssh"/>
  <port protocol="tcp" port="2221"/>
  <port protocol="tcp" port="80"/>
  <port protocol="tcp" port="6100"/>
  <port protocol="tcp" port="7700"/>
</zone>

```



