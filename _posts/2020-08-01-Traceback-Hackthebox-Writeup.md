---
title: Traceback Hackthebox Writeup
layout: post
category: hackthebox
tags: hackthebox writeup
date: 2020-08-01
---

Hello Everyone, today we’ll be doing traceback from hackthebox.
It was a relatively easy box as compared to other ‘easy’ boxes on HackTheBox.
So let’s get started…

# About
* Name: TraceBack
* IP: 10.10.10.181
* Box: Linux
* Level: Easy
* Released: 14 March 2020

# Enumeration
1. Firstly we run a nmap connect scan and a nmap all port scan on the box.
```
nmap -sC -sV -oN nmapScan.txt -v 10.10.10.181
nmap -sC -sV -oN allPortScan.txt -v -p- 10.10.10.181
```
2. Nmap output reveals the following ports open.
3. We take a look at the http port on the site
4. Looking at the source code of the html page we get an intresting comment.
5. After this it required some sweet OSINT skills. I searched the user on github and found a repo named ‘Webshell’, with its description matching the comment we found.
6. Then I created a wordlist with the names of these webshell.
7. Performing a gobuster scan on the site using the created wordlist.
```
gobuster dir -u http://10.10.10.181/ -w <wordlist> -o dirscan.txt
```
8. We find a file ‘http://10.10.10.181/smevk.php’, opening the url we are prompted for username and password. I tried ‘admin:admin’ and it worked.
9. This was a really on the webshell we see an option to upload a file, so we modify our php reverse shell file to match our ip address and port.
10. We start a listening on the port.
11. We access the url and get the shell back.

# Exploitation (User)
1. We get a shell as user webadmin. Visiting its home directory we find note.txt
2. Finding lua sudo privesc on GTFObins we execute the following command.
```
sudo -u sysadmin /home/sysadmin/luvit -e ‘os.execute(“/bin/bash”)’
```
3. We immediately get a shell as user sysadmin now.
4. We visit the home directory of sysadmin and find ‘user.txt’ file
```
cat /home/sysadmin/user.txt
```

# Exploitation (Root)
1. On getting the syadmin shell, I run linpeas script to quickly enumerate the box as sysadmin user.
2. Found a cron job executed by root.
3. Downloaded pspy32 and executed it on the box to closely observe the cronjobs.
4. It look like the root user copies the updateusermotd. file
5. We take a look at the file and find that its writeable
6. Now this part has to be very quick as we will only have 30 seconds to edit the file before the cron job replaces it again . Edit the file using the following command.
```
echo “cat /root/root.txt” >> 00header
```
7. Login using ssh and in the banner we find our root flag.
8. I Hope you understood the methodology of pwning this box. Until next time.
