---
title: Hackthebox Methodology
layout: post
tags: writeup hackthebox
category: hackthebox
date: 2020-02-24
---

Hello Everyone.
In this blog I would like to discuss about the methodology I use to solve HackTheBox machines. This is my personal method and not a standard way, feel free to modify the steps. Follow me on twitter for more.
Here’s my methodology

# Basics
Firstly I like to keep things clean, so I make a folder to keep all my findings and recon stuff in. I usually have a sub directory for recon and files, but that depends on the level of box. If the box is hard, there is more chance of the recon files to be large and detailed, hence keeping a different folder for recon helps.
I create a notes.txt file for keeping quick notes, and if the box is hard then I go for cherry tree to make notes.
ALWAYS MAKE NOTES..!! Even if you feel that’s stupid. But trust me, it will really help you when you hit a wall.

# Enumeration
I always scan the box by two scans : nmap connect scan on default ports and nmap tcp scan on all ports. Masscan can also be used as it’s much faster. And its output can be piped into nmap for further scanning. I also perfom a udp scan, just in case, I usually keep that scan in background.
```
nmap -sC -sV -v -oN nmap.txt <IP>
masscan -e tun0 -p1-65535 --rate=1000 <IP>
sudo nmap -sU -sV -A -T4 -v -oN udp.txt <IP>
```
After ports have been discovered, I closely look for the OS versions and the versions of services that are running on the ports.
If an http or https port is open I start a directory scan in the background, usually using gobuster(its hell fast). I use raft-medium-list.txt( available on SecLists github) file as wordlist.
```
gobuster dir -u <url> -w <path-to-wordlist> -x <extenstion> -o dirscan.txt
```
While the scan is running, I look for previously available exploits for the services that are running, using searchsploit or google.
If the machine is a windows box then I use the enum4linux tool
```
enum4linux -a <IP> | tee enum.txt
```
This script will perform basic windows recon automatically.

# Exploitation
Now this is the part where there is no singular path. Each box may have its own intended method of exploitation.
This is where practice and experience comes into play. Thorough recon of the target should give you some idea of how the box can be owned.
Sometimes previously made exploits may work for the box, sometimes they may require some tweaks according to the situation and sometimes you may have to create your own exploit. It depends on the level of the box.
Personally I would refrain from using metasploit for exploitation because you may not learn anything about why something worked and why the vulnerability actually existed. Also usually, the box makers tweak the box so as to stop ‘skids’ from using metasploit modules.

# Privilege Escalation
After exploitation we should usually have a shell on the box(mostly unprivileged).
My first approach is to get a proper tty shell on the box. I learned this from ippsec. He explained this right here .
After that I drop a script like linenum or linpeas on the box and run it to find any low hanging fruits or privesc ways.
To drop the script on the box you can either create a http server and download the script or you can simply copy paste the script on the machine.
Look for any privesc points
If the script results don’t give you much, try looking manually. Don’t leave any stone unturned. It gets daunting but once you find the weak link then the path would be clear.
(Optional) When you hit a wall, I recommend taking a break. Get back to the problem after sometime with a fresh mind. If you are still stuck try looking for hints in the hack the box forum. Afterall your purpose is to learn, so there is no harm in occasional seeking help from the forum. Just don’t make it regular otherwise you will be dependent on other people.

# Post exploitation
Once you get your root and user flag, delete all/any scripts or files you created on the box.
This is kinda obvious but “Don’t share the flags”.

# Conclusion
So this was my methodology for attempting hackthebox machines. I hope you learned something from it. All feedbacks are appreciated
Happy Hacking ..!!