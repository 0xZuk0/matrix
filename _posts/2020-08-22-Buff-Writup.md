---
title: Buff Writup
published: true
---

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/profile.png)

## Content
1. [Introduction](#introduction)
1. [Enumertion](#enumeration)
1. [FootHold](#foothold)
1. [Lateral Movement](#lateral-movement)
1. [Priv Esc](#priv-esc)

## Introduction

## **Enumeration**

This is the important phase of any challange in HackTheBox. Always remember Enumeration is the key. Whenever you are stuck always get back to enumeration phase because most of the time people miss the small details. So lets begin our challange.

### Nmap Scan

So first things first we are going to scan this machine for open ports and services.

`sudo nmap -sC -sV -O -oN nmap/buff 10.10.10.198`

``` 
// -sC : Perform nmap scan using default scripts (NOTE: You can use specific lua scripts to perform custom scans, please refer to man page)
// -sV : Try to get the version of the services running in the system (NOTE: It does not return version of unknown services)
// -O  : Operating System fingerprinting/Tries to guess which OS is running in the system
// -oA : Output in all format inside nmap folder with name buff
```

We got the nmap output. Let's examin it.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/nmap.png)

So there is only one port open which is `port 8080/tcp` which is running `Apache httpd 2.4.43` and have http-title `mrb3 Bro Hut`. We also get information that the server is running php whose version is `PHP/7.4.6` and `OpenSSL/1.1.1g` and this is it, There are no more ports open in system.

> **NOTE**: 
There is one thing i need to tell you reader. In the above nmap scan i haven't specified port range so nmap uses its default range. To Scan the complete 65535 ports you can use `-p-` flag but it will take hell lot of time believe me. There are tools which you can use to scan port in matter of seconds. You can try [`rustscan`]() or [`masscan`]().

### Banner Grabbing

Let's take a look at banner. Most of the time you can get lots and lots of information just from banner grabbing. For example which server it is running, Which PHP version it is running, Content-Length and list goes on. Don't ever underestimate banner grabbing because i have faced some challanges where banner grabbing gives you the information of vulnerable service running in the system. So lets begin.

I am using telnet for banner grabbing. You can use NetCat for this purpose if you like. You can also use nmap to grab banner.

First we are going to connect to port 8080 of the machine.

`telnet 10.10.10.198 8080`

when it is connected we have to send request with HEAD method to to get the banner

`HEAD / HTTP/1.1`

`Host: 10.10.10.198` and press enter

You will get output like the image.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/banner.png)

Its giving us same info which nmap gave us. There are some extra things like Cookie, Cache-Control etc. etc.. They are not looking that intresting to us. So let's move ahead.

## Foothold

### Visit to port 8080

So we have seen in the nmap scan that port 8080/tcp is open which is running apace server. So let's take a look. Visiting to the webpage we see the following webpage.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/home.png)

The first thing which i do is check if `robots.txt` is present or not. Unfortunately we are out of luck this time. Second thing i try to look at the source code and look for anything fishy, but again i didn't find anything. Looking here and there we found something intresting in the contact page.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/contact.png)

`Gym Management Software 1.0` that's looking intresting. Lets google it.

### Googling

So after some Googling we found this

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/exploit.png)

So we have an Unauthenticated Remote Command Execution. So lets download the exploit and try it.

### Unauthenticated Remote Command Execution

We can see that the code is written in python. So lets execute the exploit

`python exploit.py`

We get the following output

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/expbanner.png)

So we have to pass the WEBAPP_URL to the exploit to work. So let's do it

`python exploit.py 'http://10.10.10.198/8080/'`

Andddd.... Bingo!! We got a RCE in the machine

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/postexp.png)



## Lateral Movement

## Priv Esc