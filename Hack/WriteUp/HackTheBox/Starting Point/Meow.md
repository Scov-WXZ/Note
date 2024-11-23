# Meow

> Connect using OpenVPN
>
> TCP 443 -> DOWNLOAD VPN -> starting_point_username.ovpn
>
> Linux bash: openvpn starting_point_username.ovpn
>
> 目标 IP: 11.11.11.12

## TASK 1

> What does the acronym VM stand for?

- virtual machine

## TASK 2

> What tool do we use to interact with the operating system in order to issue commands via the command line, such as the one to start our VPN connection? It's also known as a console or shell.

- terminal

## TASK 3

> What service do we use to form our VPN connection into HTB labs?

- openvpn

## TASK 4

> What tool do we use to test our connection to the target with an ICMP echo request?

- ping

## TASK 5

> What is the name of the most common tool for finding open ports on a target?

- nmap

## TAKS 6

> What service do we identify on port 23/tcp during our scans?

- telnet

## TASK 7

> What username is able to log into the target over telnet with a blank password?

- root

## Submit Flag

```shell
$ openvpn starting_point_username.ovpn
```

Another Terminal

```shell
$ nmap -Pn 11.11.11.12
# 23/tcp telnet

$ telnet 11.11.11.12
Meow login: root

root@Meow:~# ls
flag.txt  snap
```
