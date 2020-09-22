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

## Enumeration

This is the important phase of any challange in HackTheBox. Always remember Enumeration is the key. Whenever you are stuck always get back to enumeration phase because most of the time people miss the small details. So lets begin our challange.

### Nmap Scan

So first things first we are going to scan this machine for open ports and services.

```
sudo nmap -sC -sV -O -oN nmap/buff 10.10.10.198 
// -sC : Perform nmap scan using default scripts (NOTE: You can use specific lua scripts to perform custom scans, please refer to man page)
// -sV : Try to get the version of the services running in the system (NOTE: It does not return version of unknown services)
// -O  : Operating System fingerprinting/Tries to guess which OS is running in the system
// -oA : Output in all format inside nmap folder with name buff
```

We got the nmap output. Let's examin it.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/nmap.png)

So there is only one port open which is `port 8080/tcp` which is running `Apache httpd 2.4.43`. We also get information that the server is running php whose version is `PHP/7.4.6` and `OpenSSL/1.1.1g` and this is it, There are no more ports open in system.

> NOTE: 
There is one thing i need to tell you reader. In the above nmap scan i haven't specified port range so nmap uses its default range. To Scan the complete 65535 ports you can use `-p-` flag but it will take hell lot of time believe me. There are tools which you can use to scan port in matter of seconds. You can try [`rustscan`]() or [`masscan`]().

## Foothold

## Lateral Movement

## Priv Esc