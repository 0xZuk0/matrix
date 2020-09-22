---
title: Buff Writup
published: true
---

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/profile.png))

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
// -oN : Output in nmap format inside nmap folder with name buff
```

We got the nmap output. Let's examin it.
![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/nmap.png) 

## Foothold

## Lateral Movement

## Priv Esc