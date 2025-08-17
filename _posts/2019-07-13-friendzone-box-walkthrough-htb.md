---
layout: post
title: "FRIENDZONE Box Walkthrough (HTB)"
permalink: ctf-writeup/friendzone
tags: [CTF Walkthrough]
categories: [CTF Walkthrough]
image:
    path: /assets/img/friendzone/friendzone.jpg?w=775
---

What's up my fellow CTF lovers. Today we are gonna do the Friendzone box from Hack The Box which has been retired recently. This box is one of the good level boxes from Easy category and is full of rabbit holes. So, let's jump in!!

First we do a nmap scan with banner grabbing using the following command:
    
    
    # nmap -sS -A 10.10.10.123
    
    
    Nmap scan report for 10.10.10.123
     Host is up (0.24s latency).
     PORT    STATE SERVICE     VERSION
     **21/tcp  open  ftp         vsftpd 3.0.3**
     **22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)**
     | ssh-hostkey: 
     |   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
     |   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
     |_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
     **53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)**
     | dns-nsid: 
     |_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
     **80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))**
     |_http-server-header: Apache/2.4.29 (Ubuntu)
     |_http-title: Friend Zone Escape software
     **139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)**
     **443/tcp open  ssl/http    Apache httpd 2.4.29**
     |_http-server-header: Apache/2.4.29 (Ubuntu)
     |_http-title: 404 Not Found
     | ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
     | Not valid before: 2018-10-05T21:02:30
     |_Not valid after:  2018-11-04T21:02:30
     |_ssl-date: TLS randomness does not represent time
     | tls-alpn: 
     |   http/1.1
     **445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)**
     Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
     Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.16 (95%), Linux 3.18 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.1 (93%), Linux 3.2 (93%), Linux 3.10 - 4.11 (93%), Linux 3.12 (93%), Linux 3.13 (93%), Linux 3.8 - 3.11 (93%)
     No exact OS matches for host (test conditions non-ideal).
     Network Distance: 2 hops
     Service Info: Hosts: FRIENDZONE, 127.0.0.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
     Host script results:
     |_clock-skew: mean: -1h00m00s, deviation: 1h43m54s, median: -1s |_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: , NetBIOS MAC:  (unknown) | smb-os-discovery:  |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu) |   Computer name: friendzone |   NetBIOS computer name: FRIENDZONE\x00 |   Domain name: \x00 |   FQDN: friendzone |_  System time: 2019-07-07T08:53:26+03:00
     | smb-security-mode: 
     |   account_used: guest
     |   authentication_level: user
     |   challenge_response: supported
     |_  message_signing: disabled (dangerous, but default)
     | smb2-security-mode: 
     |   2.02: 
     |_    Message signing enabled but not required
     | smb2-time: 
     |   date: 2019-07-07 01:53:30
     |_  start_date: N/A
     TRACEROUTE (using port 53/tcp)
     HOP RTT       ADDRESS
     1   312.83 ms 10.10.12.1
     2   313.19 ms 10.10.10.123
     OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
     Nmap done at Sun Jul  7 02:04:12 2019 -- 1 IP address (1 host up) scanned in 675.18 seconds

As we can we the Samba service is running on the target host. Let's start with it. First, we will use "smbmap" tool to check what SMB shares are available.
    
    
    smbmap -H 10.10.10.123

![](/assets/img/friendzone/image.png)

We have two interesting shares available here named "general" and "Development". Since "Development" is made writable, we can assume that it must be some important part in the exploitation process. 

Let's see what's there in "general" SMB share using **smbclient ** tool. 
    
    
    smbclient //10.10.10.123/general

![](/assets/img/friendzone/image-2.png)

There is a file "creds.txt" residing inside the "general" share. Let's download it and see what's in there! We can use **get** command to download the file as shown in the output.

The file seems to be containing credentials saying that it belongs to some "admin thing'. Now we need to find that "admin thing" out.

Now, let seek inside the other SMB share "Development".

![](/assets/img/friendzone/image-4.png)

As we see, this share doesn't contain anything. 

Now we are done with SMB enumeration. Now looking at nmap results again, we see that the famous port 80 is open on the target. So let's fire up the browser and see what's going on.

![](/assets/img/friendzone/friendzone_logo.png?w=775)

When we open the URL, this image appears at the index page. As we generally do, we start with inspecting the source code.

![](/assets/img/friendzone/image-6.png)

There isn't anything interesting in the source code. Also, running the gobuster/dirbuster/dirb didn't help. I also tried downloading the image and searching for some juice there using **strings **and **exiftool** but I didn't get anything.

But after some time, I noticed something in the index page itself, the **email**:

![](/assets/img/friendzone/image-7.png)

This can be something useful to us. So I put **friendzoneportal.red** as a record inside my /etc/hosts file. But it didn't help. But when I removed the "portal" from the entry, it worked. Here is what I added to my /etc/hosts file:

![](/assets/img/friendzone/image-8.png)

I didn't find anything else of interest there. So, let's do some DNS enumeration as the domain service is running on port 53.

There is a possibility that this domain we found might have subdomains. So let's search for them. We can use **dig** tool to find out subdomains of a domain by querying name servers of that domain:
    
    
    # dig ns friendzone.red

![](/assets/img/friendzone/image-9.png)

As we can see, there are two nameservers of the domain "friendzone.red". Now, let's find out subdomains by quering the AXFR records:
    
    
    # dig axfr friendzone.red @friendzone.red

![](/assets/img/friendzone/image-10.png)

We have found three subdomains which are highlighted in the output image. Now, let's add them to /etc/hosts file and enumerate. Here is how the /etc/hosts file looks like now:

![](/assets/img/friendzone/image-11.png)

If we try enumerating those subdomains, we see that same index page is appearing to us. So, there is no fun in enumeration as we are enumerating the same page. 

But wait, we do see a HTTPS service on port 443. Let's see these subdomains using HTTPS.

Let's enumerate the **administrator1.friendzone.red **subdomain first. Opening the URL leads us to a login page:

![](/assets/img/friendzone/image-12.png)

Since the subdomain name is **administrator1** , It might be that "admin THING" those credentials we obtained were about. If we try those credentials here, We get a page which tells us to go to **/dashboard.php**. Now, once we go to **dashboard.php**, we are welcomed with the following page:

![](/assets/img/friendzone/image-14.png)

As it seems from the page, it has been created by an unskilled programmer! It's an indirect way of telling that the sanitation checks are absent here.

What dashboard.php does is that it allows us to display our uploaded pictures on the page but we haven't uploaded any picture yet and there is no option in the website to upload that picture. 

You guys might be thinking about the **uploads.friendzone.red** but that turned out to be a rabbit hole which is fake and leads us nowhere.

Ahh! Wait! Remember that writable SMB share? Yea, that might be useful here. Let's try uploading a reverse shell on that share. We can do this simply by opening the share and uploading a reverse shell using **put** command.

![](/assets/img/friendzone/image-15.png)

Now we have uploaded our reverse shell on the SMB share. Let's go back to the admin dashboard page and try to display it in the browser.

![](/assets/img/friendzone/image-16.png)

As we can see, It's written on the page that in order to include/display an image, we need to use **image_name** parameter. But guess what? It was a rabbit hole that is of no use to us here. Let's use the default ones which are written on the page: **image_id=a.jpg&pagename=timestamp**. 

![](/assets/img/friendzone/image-17.png)

Opening the URL with those default parameters, we come across a "Haha" image. But the strange thing here is the text which appears on the left side of the page. It says:
    
    
    Final Access timestamp is <some_random_number>

Now, look at the default parameters again and see the relation of "timestamp" output with the default parameters. There is a default parameter named **pagename** which is currently set to **timestamp**. Now think about it, what could be the case here? Maybe the **pagename** parameter is opening the page(or PHP file) supplied to it? 

This inclusion activity can be confirmed by confirming the existence of **timestamp.php**. Let's open the **/timestamp.php **and check if it exists or not. Opening the /timestamp.php gave us the following output:

![](/assets/img/friendzone/image-18.png)

Alright, now we are sure that **pagename** parameter on the **dashboard.php** includes PHP files. 

Now we could try to include our PHP reverse shell here from the SMB share but we don't know where the SMB shares are stored. So we don't know the path of our reverse shell on the target system.

To figure this out, I did the SMB enumeration again using different tools. Turned out that If I use **smbclient** instead of **smbmap** to look for available shares, one of the shares were leaking it's path.

![](/assets/img/friendzone/image-19.png)

Now we know that the SMB share named "Files" is stored in /etc/Files. Then it's quite obvious that the "Development" share might be residing in **/etc/Development**.   


Ok, Now we have got the path to our uploaded reverse shell, which is:
    
    
    /etc/Development/reverse_shell.php

Now we can try to include it in the webpage.

Let's start a netcat listner first to receive the reverse shell.

![](/assets/img/friendzone/image-20.png)

Now, let's open the Admin's dashboard page with our reverse shell path written in the pagename parameter like so:
    
    
    https://administrator1.friendzone.red?dashboard.php?image_id=a.jpg&pagename=/etc/Development/reverse_shell

Notice that we didn't wrote ".php" in the end because it will be appended by the script.  
Now, opening this URL immediately gives us a reverse shell:

![](/assets/img/friendzone/image-21.png)

As we have gained the shell now, let's go to /var/www and see what else the web page has. While enumeration, we come across a file named **mysql_data.conf**

![](/assets/img/friendzone/image-22.png)

Opening that files gives us credentials of mysql of a user named **friend**. 

![](/assets/img/friendzone/image-24.png)

But it turns out that we don't have permissions to run mysql. So what else can be done with these credentials? Aah! There is an SSH port running on the system. Let's use these credentials there. 
    
    
    ssh friend@friendzone.red

![](/assets/img/friendzone/image-25.png)

Wow! It worked! Now you can grab the user.txt from /home/friend/user.txt

Let's move to privelege escalation part now.

I searched for all SUID binaries and saw all the running processes using **netstat** but didn't find anything interesting. Then, I downloaded a tool named **pspy** which let's us monitor unusual/all processes running in the background. I downloaded it from [here](https://github.com/DominicBreuker/pspy/releases/download/v1.0.0/pspy64) and transferred it to the target system.

Now, running this tool showed a very interesting process which was based on python:

![](/assets/img/friendzone/image-26.png)

As we can see, It's running a python script named **reporter.py** stored in **/opt/server_admin/**.

Looking at the source inside there, there is no such vulnerability in the code which could be exploited. However, you may notice that it's importing the **os module**.

![](/assets/img/friendzone/image-27.png)

The only way to exploit this python code is to alter the source code of python's os module. Let's see if we can do that. First we need to find where this os module is located. To do that, I ran a **locate** command to find the os.py file.

![](/assets/img/friendzone/image-28.png)

Since target has two versions of python installed, there were two files named os.py. Since the reporter.py is being run by python2.7, we will try to play with the os.py file belonging to python2.7.

I simply wrote a reverse shell command wrapped inside **os.system()** and appended it to os.py. 

![](/assets/img/friendzone/image-29.png)

But, before adding it to the os.py file, we need to start our netcat listner in order to grab the shell.

![](/assets/img/friendzone/image-30.png)

Once the os.py file is written with our payload, within few seconds or a minute, we receive a root shell in our system.

![](/assets/img/friendzone/image-31.png)

Yay! We got root! Now you can grab the flag and submit it!

**[Thanks for giving it a read! ~xscorp]**
