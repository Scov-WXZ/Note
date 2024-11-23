# Redeemer

> 目标 IP: 11.11.11.12

## TASK 1

> Which TCP port is open on the machine?

- 6379

## TASK 2

> Which service is running on the port that is open on the machine?

- Redis

## TASK 3

> What type of database is Redis? Choose from the following options: (i) In-memory Database, (ii) Traditional Database

- In-memory Database

## TASK 4

> Which command-line utility is used to interact with the Redis server? Enter the program name you would enter into the terminal without any arguments.

- redis-cli

## TASK 5

> Which flag is used with the Redis command-line utility to specify the hostname?

- -h

## TASK 6

> Once connected to a Redis server, which command is used to obtain the information and statistics about the Redis server?

- info

## TASK 7

> What is the version of the Redis server being used on the target machine?

- 5.0.7

## TASK 8

> Which command is used to select the desired database in Redis?

- select

## TASK 9

> How many keys are present inside the database with index 0?

- 4

## TASK 10

> Which command is used to obtain all the keys in a database?

- keys \*

## Submit Flag

```shell
$ redis-cli -h 11.11.11.12 -p 6379
11.11.11.12:6379> keys *
1) "stor"
2) "numb"
3) "flag"
4) "temp"
(2.28s)
11.11.11.12:6379> get flag
```
