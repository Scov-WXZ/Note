# Fawn

> 目标 IP: 11.11.11.12

## TAKS 1

> What does the 3-letter acronym FTP stand for?

- File Transfer Protocol

## TASK 2

> Which port does the FTP service listen on usually?

- 21

## TASK 3

> FTP sends data in the clear, without any encryption. What acronym is used for a later protocol designed to provide similar functionality to FTP but securely, as an extension of the SSH protocol?

- SFTP

## TASK 4

> What is the command we can use to send an ICMP echo request to test our connection to the target?

- ping

## TASK 5

> From your scans, what version is FTP running on the target?

- vsftpd 3.0.3

## TAKS 6

> From your scans, what OS type is running on the target?

- Unix

## TAKS 7

> What is the command we need to run in order to display the 'ftp' client help menu?

- ftp -h

## TAKS 8

> What is username that is used over FTP when you want to log in without having an account?

- anonymous

## TAKS 9

> What is the response code we get for the FTP message 'Login successful'?

- 230

## TAKS 10

> There are a couple of commands we can use to list the files and directories available on the FTP server. One is dir. What is the other that is a common way to list files on a Linux system.

- ls

## TAKS 11

> What is the command used to download the file we found on the FTP server?

- get

## Submit Flag

```shell
$ nmap -sV -Pn 11.11.11.12
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
Service Info: OS: Unix

$ ftp 11.11.11.12
Connected to 11.11.11.12.
220 (vsFTPd 3.0.3)
Name (11.11.11.12:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||61039|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
ftp> get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||24987|)
150 Opening BINARY mode data connection for flag.txt (32 bytes).
100% |***************|    32        0.11 KiB/s    00:00 ETA
226 Transfer complete.
32 bytes received in 00:01 (0.01 KiB/s)
ftp> exit
221 Goodbye.
```
