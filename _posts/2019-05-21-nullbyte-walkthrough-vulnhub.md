---
layout: post
title: "NullByte Walkthrough (Vulnhub)"
permalink: ctf-writeup/nullbyte
author: xscorp
tags: [CTF Walkthrough]
categories: [CTF Walkthrough]
image:
    path: /assets/img/nk/hacked-1.png?w=775
---
Hello my fellow hackers! Today we are going to do the [NullByte](https://www.vulnhub.com/entry/nullbyte-1,126/) machine from vulnhub. I am assuming that you have already set it up on virtual box/vmware or any other virtual environment and it's running. If you haven't done it yet, I will recommend you to search Google to do that.

First of all, before testing anything, we must be knowing the IP of the target machine. To obtain that, we need to use the **Netdiscover** tool. The netdiscover tool can scan IP range for connected hosts. I have used class C addresses for my host and guest machines, so the IP will be of the form 192.168.xx.xx

To pass the IP range/network IP to netdiscover, we can use **-r** option.
    
    
    # netdiscover -r 192.168.43.0/24

![](/assets/img/nk/image.png)Output of netdiscover tool

As you can see that it listed two connected devices. First one belongs to the router itself while the second one is of the target system. I have hidden the MAC addresses for security reasons.

Now we have got the IP of the target system, now it's time for enumeration. We will do an nmap scan for port scanning. I used the following command for port scanning through nmap.
    
    
    # nmap -sS -sC -sV  -A -p- 192.168.43.148 -oN nmap.txt

It requires a bit explanation! The **-sS** flag is used for **TCP SYN scan**. **-sC** is used for using **default NSE scripts**, **-sV** is used for **version detection**, **-A **for **banner grabbing**, **-p- **for **scanning all ports**. Following our target IP, the last flag **-oN ** is for **output in normal text form** and the file name is supplied as an argument to the flag. I have marked the ports with red arrow.

![](/assets/img/nk/nmap.png)Output of nmap scan

Let's first have a look at HTTP service running on the target host on port 80.

![](/assets/img/nk/image-1.png)Website running on target host

Just like every other webpage inspection, we will first start by checking the source code of the web page to see if there is anything interesting there. Turned out there isn't anything useful.
    
    
    <html>
    <head><title>Null Byte 00 - level 1</title></head>
    <body>
    <center>
    <img src="main.gif">
    <p> If you search for the laws of harmony, you will find knowledge. </p>
    </center>	
    </body>
    </html>
    

After struggling for hours with it, I thought there could be something inside the **main.gif** file. I got this hint from the line "_If you search for the laws of harmony, you will find knowledge._".  
I downloaded the main.gif by issuing the following command:
    
    
    wget http://192.168.43.148/main.gif

![](/assets/img/nk/image-2.png)Output of wget command 

Now, to inspect the gif file, I used an amazing tool called **exiftool**. This tool is well known for looking at metadata of an image file. 
    
    
    # exiftool main.gif

![](/assets/img/nk/image-3.png)Output of exiftool for main.gif inspection

Did you find anything weird looking or unusual here. If not, then look at the highlighted line. That comment seems unusual. Just store it somewhere, we might be needing it later.

**NOTE: I haven't mentioned the directory brute force tools like dirbuster, dirb, gobuster etc because they didn't find anything special here. **

After roaming here and there in the website for a while, I tested the string**"kzMb5nVYJw**" which we got using exiftool in URL to check if there's any page/directory named so, Luckily I succeed! We got an authentication page on it.   


![](/assets/img/nk/image-4.png)Authentication page inside the directory

Again a web page! Remember what was the first step to inspect it? Yea, check the source code.
    
    
    <center>
    <form method="post" action="index.php">
    Key:<br>
    <input type="password" name="key">
    </form> 
    </center>
    <!-- this form isn't connected to mysql, password ain't that complex --!>

There isn't anything special in the source code BUT there is a line which might be a hint to the attacker. Look at the line "_this form isn't connected to mysql, password ain't that complex_". If you have some experience in breaking boxes, you might be aware that reality is totally opposite of what is mentioned in source codes generally. This is done to prevent attacker from making even an attempt to connect to MySQL. But it's a clear and true hint that the password isn't that complex. This leads us to brute force attack. 

I used **hydra **tool to brute force the form for passwords. To work with hydra, we need to tell the hydra how the request string or request parameters looks like in the HTTP request. To get that information, I used **burpsuite ** tool to intercept HTTP requests and examined the request. I entered a random key(password) and hit Enter. The request generated was:
    
    
    POST /kzMb5nVYJw/index.php HTTP/1.1
    Host: 192.168.43.148
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:65.0) Gecko/20100101 Firefox/65.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: http://192.168.43.148/kzMb5nVYJw/
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 11
    Connection: close
    Upgrade-Insecure-Requests: 1
    DNT: 1
    
    
    
    key=testKey

I want you to notice few things here. First is that our entered key is being passed through the **key** variable. Since the variable is included in the request's body, we know that this form is using **POST** method. Now we need to pass the **key** variable to hydra so that it can make appropriate request to the server. Following is the **hydra **syntax I used for brute forcing the key.
    
    
    hydra 192.168.43.148 -V -l "" -P /home/xscorp/Desktop/NullByte/brute_pass.txt  http-post-form "/kzMb5nVYJw/index.php:key=^PASS^:invalid"

The **-V** flag is used for **verbose**,  
**-l **is for specifying the username. Since there is no username here, so we passed the ""  
**-P **is for specifying the password file. Here, I used a custom wordlist named **brute_pass.txt**. You could choose whatever you want.   
**http-post-form **is the name of the Hydra's module which is used for brute forcing HTTP-forms password which uses POST method to send data to server.  
Next, the http-post-form method accepts 3 colon(":") separated values which are in the following order:  
_<target_page_URL>**:**<paramaters>**:**<attempt_failed_string>_  
So, here we have specified the target page URL as the index.php located in **kzMb5nVYJw**.  
The **key** variable to be passed should be written at the second place. The **^PASS^** is a type of regular expression which tells the hydra to put values in place of it. Then ultimately, we need to specify the attempt failed string. This tells the hydra what a failed attempt looks like. Otherwise there's no way Hydra can identify if a password guess was correct or not. To get the attempt_failed_string, we can try a wrong attempt and check how the HTML is changing in the page. Turned out that the page displays something like "Invalid key". So, we passed the "Invalid" at third place for failed attempt identification. 

![](/assets/img/nk/image-5.png)Output of hydra tool

So Now we have the right key **elite**. Entering the key in the web page will lead us to a new page.  


![](/assets/img/nk/image-6.png)Page after entering the right key

After passing several wrong usernames, I passed an empty string and BOOM, it showed me all the usernames.  


![](/assets/img/nk/image-7.png)Page's output after passing an empty string as username

Hmmn, Look at those names of attributes. They look a lot like columns of a table in database. If an empty string revealed so much about it, then there are chances that it is vulnerable to SQL injection attack. Trying **' **and **" **broke the syntax and displayed the following error message.  


![](/assets/img/nk/image-8.png)Error on passing " as username

Ok, So now there is no doubt that this is vulnerable to SQL injection attack. 

I tried different payloads manually like " OR SHOW DATABASES; " but nothing helped. After almost half an hour, I realized that we need to be good enough to understand and break it. At this point, I googled for walkthrough so that I can get past it. But unfortunately, there were no walkthroughs which show it by manually passing the attack payload. So I left that too and used automated tools like **sqlmap** to take advantage of that SQL injection vulnerability.

The **sqlmap** is a powerful tool which is used for automated vulnerability scanning. I used the following command to search for the vulnerabilities and ultimately to list the existing databases:
    
    
    # sqlmap --url http://192.168.43.148/kzMb5nVYJw/420search.php?usrtosearch= --dbs

Here, **\--url** is used for specifying the URL of the target page and **\--dbs** is used for displaying the list of existing databases once the vulnerability is exploited.  


![](/assets/img/nk/image-9.png)Output of sqlmap ![](/assets/img/nk/image-10.png)

Ok, now we have access to database. That **seth** database looks interesting. Let's check that out. The following commands lists the tables inside the "seth" database:
    
    
    # sqlmap --url http://192.168.43.148/kzMb5nVYJw/420search.php?usrtosearch= -D seth --tables

![](/assets/img/nk/image-11.png)

Looks like we are heading the right way. Let's dump all the records inside the table **users**.
    
    
    # sqlmap --url http://192.168.43.148/kzMb5nVYJw/420search.php?usrtosearch= -D seth -T users --dump

![](/assets/img/nk/image-12.png)users table dump from the database

The **isis** seems uninteresting. However **ramses** user has some encrypted/encoded password. Again, a little bit experienced people will know just by a sight that it's a base64 encoded string. Let's decrypt it!
    
    
    # echo "YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE=" | base64 --decode

The encoded string needed to be padded. So I added an extra '=' at the end. The output/decoded string is: 
    
    
    c6d6bd7ebf806f43c76acc3681703b81

Now, It turned out that this weird string is an md5 hash. Since I didn't know how to use tools like hashcat, I used an online md5 decoder like [this](https://www.md5online.org/md5-decrypt.html) one to decrypt the md5 hash.  


![](/assets/img/nk/image-13.png)Decoded hash

**omega**, we have got it. But what to do with this value? Till now, all I know is that this password belongs to the user **ramses** but where should I use it?   
Well, If you look carefully at the nmap result, there is a **SSH** service running on the target. Let's try these credentials there.
    
    
    # ssh ramses@192.168.43.148 -p 777

In the above command, we are using the **ssh** client to connect to remote host with the username **ramses** at port 777 specified using **-p** option. Entering the password as **omega** when prompted, we got shell! Yay!

![](/assets/img/nk/image-14.png)

Now, It's time for privilege escalation. Listing all the directories including hidden directory reveals that there is a file named **.bash_history** which generally contains the recent commands used against that user's home directory. Reading it reveals the existence of a file named **procwatch**. 

![](/assets/img/nk/image-17.png)

This file might be the key to root access. But first, we need to find out where it is located. To do this, we can use **find **command.
    
    
    ramses@NullByte:~$ find / -type f -name procwatch

**-type f **specifies that we are looking for a file and not a directory. **-name procwatch **tells the find tool that the name of the file to be searched is **procwatch**.

![](/assets/img/nk/image-15.png)

We have got the location of **procwatch**. Let's see what's there!

Getting inside that directory, I found out that it's an executable file. And what's beautiful about it is that it's a SUID binary which means this can be executed by anyone with the privileges of the user who owns this file. And luckily, the owner of this file is **root**, that means if we can hijack it's execution, we can execute arbitrary code or gain a shell.

![](/assets/img/nk/image-18.png)

Now, let's execute the **procwatch **and examine it's behavior.

![](/assets/img/nk/image-19.png)

Wait! Doesn't this look familiar? That's what the **ps** command does in linux i.e. listing the running processes of the current session. Following is the output of **ps** command. 

![](/assets/img/nk/image-20.png)

Compare both the outputs and see the similarity. That's because **procwatch **is using **ps **from inside.

Now, there is a chance that **procwatch **might be using relative path of the **ps **tool instead of absolute path. Sounds confusing? Here's how both type of paths looks like:

Relative path: `ps`  
Absolute path: `/bin/ps`

Relative path shouldn't be used in binaries as it can be abused using path manipulation technique. Let's check the value of **PATH** environment variable.
    
    
    ramses@NullByte:/var/www/backup$ echo $PATH
    
    
    /usr/local/bin:/usr/bin:**/bin**:/usr/local/games:/usr/games

Notice that **/bin** there. The **procwatch** must be executing **ps** from this path. Can you think of a way of hijacking it? Yes, we can modify the **PATH** and add a custom path for **ps** so that **procwatch **can call our variant of **ps **executable. Let's add any arbitrary writable path like **/tmp** to the **PATH**.  
`ramses@NullByte:/var/www/backup$ export PATH=/tmp:$PATH`
    
    
    ramses@NullByte:/var/www/backup$ echo $PATH
    
    
    **/tmp**:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

Now, the value of **PATH **variable has changed. Now **procwatch** will look for **ps** in **/tmp **before **/bin**.   
Now let's navigate to /tmp and create our own executable named **ps **for the **procwatch** to execute with root privileges.

All we want to write in our** ps **file is the command for spawning a shell. Since this file will be executed with **root** privileges, we will gain the root shell. I wrote this in **ps **file to spawn a shell:  
`ramses@NullByte:/tmp$ echo "/bin/sh -i" > ps`  
Here, **echo **command will create a file if doesn't exist already and write whatever is next into it. **"/bin/sh -i" **is for spawning an **interactive **shell. **> ps** is used for writing this to **ps **file.

Once our file is written, now we need to give it global execution permission so that It can be executed by any user(root in this case). This can be done using **chmod**.  
`ramses@NullByte:/tmp$ chmod +x ./ps`

Now, everything is set and it's showtime. It's time to execute the **procwatch **binary and see what happens. If everything goes well and our steps are correct, we must get root shell.

![](/assets/img/nk/image-21.png)

Hurrey! We got root! We have finally hacked this box. Now let's get the flag located in **/root **directory.

![](/assets/img/nk/image-22.png)

I have hidden some part of the flag so that you guys can't directly see it. Try it yourself and you will learn a lot. Seeing a walkthrough is simple, but trying the box even after watching the walkthrough is not that easy. Good Luck! Happy Hacking!
