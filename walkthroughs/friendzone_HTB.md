# FRIENDZONE box Walkthrough (HTB)

<!-- wp:image {"id":183} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/friendzone.jpg?w=775" alt="" class="wp-image-183"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>What's up my fellow CTF lovers. Today we are gonna do the Friendzone box from Hack The Box which has been retired recently. This box is one of the good level boxes from Easy category and is full of rabbit holes. So, let's jump in!!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>First we do a nmap scan with banner grabbing using the following command:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># nmap -sS -A 10.10.10.123</code></pre>
<!-- /wp:code -->

<!-- wp:preformatted -->
<pre class="wp-block-preformatted">Nmap scan report for 10.10.10.123
 Host is up (0.24s latency).
 PORT    STATE SERVICE     VERSION
 <b style="color:red;">21/tcp  open  ftp         vsftpd 3.0.3</b>
 <b style="color:red;">22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)</b>
 | ssh-hostkey: 
 |   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
 |   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
 |_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
 <b style="color:red;">53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)</b>
 | dns-nsid: 
 |_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
 <b style="color:red;">80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))</b>
 |_http-server-header: Apache/2.4.29 (Ubuntu)
 |_http-title: Friend Zone Escape software
 <b style="color:red;">139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)</b>
 <b style="color:red;">443/tcp open  ssl/http    Apache httpd 2.4.29</b>
 |_http-server-header: Apache/2.4.29 (Ubuntu)
 |_http-title: 404 Not Found
 | ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
 | Not valid before: 2018-10-05T21:02:30
 |_Not valid after:  2018-11-04T21:02:30
 |_ssl-date: TLS randomness does not represent time
 | tls-alpn: 
 |   http/1.1
 <b style="color:red;">445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)</b>
 Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
 Aggressive OS guesses: Linux 3.2 - 4.9 (95%), Linux 3.16 (95%), Linux 3.18 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.1 (93%), Linux 3.2 (93%), Linux 3.10 - 4.11 (93%), Linux 3.12 (93%), Linux 3.13 (93%), Linux 3.8 - 3.11 (93%)
 No exact OS matches for host (test conditions non-ideal).
 Network Distance: 2 hops
 Service Info: Hosts: FRIENDZONE, 127.0.0.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
 Host script results:
 |<em>clock-skew: mean: -1h00m00s, deviation: 1h43m54s, median: -1s |_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: , NetBIOS MAC:  (unknown) | smb-os-discovery:  |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu) |   Computer name: friendzone |   NetBIOS computer name: FRIENDZONE\x00 |   Domain name: \x00 |   FQDN: friendzone |</em>  System time: 2019-07-07T08:53:26+03:00
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
 Nmap done at Sun Jul  7 02:04:12 2019 -- 1 IP address (1 host up) scanned in 675.18 seconds</pre>
<!-- /wp:preformatted -->

<!-- wp:paragraph -->
<p>As we can we the Samba service is running on the target host. Let's start with it. First, we will use "smbmap" tool to check what SMB shares are available.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>smbmap -H 10.10.10.123</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":142} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image.png" alt="" class="wp-image-142"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>We have two interesting shares available here named "general" and "Development". Since "Development" is made writable, we can assume that it must be some important part in the exploitation process. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Let's see what's there in "general" SMB share using <strong>smbclient </strong> tool. </p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>smbclient //10.10.10.123/general</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":144} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-2.png" alt="" class="wp-image-144"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>There is a file "creds.txt" residing inside the "general" share. Let's download it and see what's in there! We can use <strong>get</strong> command to download the file as shown in the output.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The file seems to be containing credentials saying that it belongs to some "admin thing'. Now we need to find that "admin thing" out.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now, let seek inside the other SMB share "Development".</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":146} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-4.png" alt="" class="wp-image-146"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As we see, this share doesn't contain anything. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now we are done with SMB enumeration. Now looking at nmap results again, we see that the famous port 80 is open on the target. So let's fire up the browser and see what's going on.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":147} -->
<figure class="wp-block-image is-resized"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-5.png" alt="" class="wp-image-147"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>When we open the URL, this image appears at the index page. As we generally do, we start with inspecting the source code.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":148} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-6.png" alt="" class="wp-image-148"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>There isn't anything interesting in the source code. Also, running the gobuster/dirbuster/dirb didn't help. I also tried downloading the image and searching for some juice there using <strong>strings </strong>and <strong>exiftool</strong> but I didn't get anything.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>But after some time, I noticed something in the index page itself, the <strong>email</strong>:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":149} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-7.png" alt="" class="wp-image-149"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>This can be something useful to us. So I put <strong>friendzoneportal.red</strong> as a record inside my /etc/hosts file. But it didn't help. But when I removed the "portal" from the entry, it worked. Here is what I added to my /etc/hosts file:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":150} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-8.png" alt="" class="wp-image-150"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>I didn't find anything else of interest there. So, let's do some DNS enumeration as the domain service is running on port 53.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>There is a possibility that this domain we found might have subdomains. So let's search for them. We can use <strong>dig</strong> tool to find out subdomains of a domain by querying name servers of that domain:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># dig ns friendzone.red</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":152} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-9.png" alt="" class="wp-image-152"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As we can see, there are two nameservers of the domain "friendzone.red". Now, let's find out subdomains by quering the AXFR records:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># dig axfr friendzone.red @friendzone.red</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":153} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-10.png" alt="" class="wp-image-153"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>We have found three subdomains which are highlighted in the output image. Now, let's add them to /etc/hosts file and enumerate. Here is how the /etc/hosts file looks like now:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":154} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-11.png" alt="" class="wp-image-154"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>If we try enumerating those subdomains, we see that same index page is appearing to us. So, there is no fun in enumeration as we are enumerating the same page. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>But wait, we do see a HTTPS service on port 443. Let's see these subdomains using HTTPS.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Let's enumerate the <strong>administrator1.friendzone.red </strong>subdomain first. Opening the URL leads us to a login page:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":156} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-12.png" alt="" class="wp-image-156"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Since the subdomain name is <strong>administrator1</strong> , It might be that "admin THING" those credentials we obtained were about. If we try those credentials here, We get a page which tells us to go to <strong>/dashboard.php</strong>. Now, once we go to <strong>dashboard.php</strong>, we are welcomed with the following page:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":159} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-14.png" alt="" class="wp-image-159"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As it seems from the page, it has been created by an unskilled programmer! It's an indirect way of telling that the sanitation checks are absent here.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>What dashboard.php does is that it allows us to display our uploaded pictures on the page but we haven't uploaded any picture yet and there is no option in the website to upload that picture. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>You guys might be thinking about the <strong>uploads.friendzone.red</strong> but that turned out to be a rabbit hole which is fake and leads us nowhere.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Ahh! Wait! Remember that writable SMB share? Yea, that might be useful here. Let's try uploading a reverse shell on that share. We can do this simply by opening the share and uploading a reverse shell using <strong>put</strong> command.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":160} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-15.png" alt="" class="wp-image-160"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Now we have uploaded our reverse shell on the SMB share. Let's go back to the admin dashboard page and try to display it in the browser.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":161} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-16.png" alt="" class="wp-image-161"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As we can see, It's written on the page that in order to include/display an image, we need to use <strong>image_name</strong> parameter. But guess what? It was a rabbit hole that is of no use to us here. Let's use the default ones which are written on the page: <strong>image_id=a.jpg&amp;pagename=timestamp</strong>. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":162} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-17.png" alt="" class="wp-image-162"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Opening the URL with those default parameters, we come across a "Haha" image. But the strange thing here is the text which appears on the left side of the page. It says:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>Final Access timestamp is &lt;some_random_number></code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Now, look at the default parameters again and see the relation of "timestamp" output with the default parameters. There is a default parameter named <strong>pagename</strong> which is currently set to <strong>timestamp</strong>. Now think about it, what could be the case here? Maybe the <strong>pagename</strong> parameter is opening the page(or PHP file) supplied to it? </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>This inclusion activity can be confirmed by confirming the existence of <strong>timestamp.php</strong>. Let's open the <strong>/timestamp.php </strong>and check if it exists or not. Opening the /timestamp.php gave us the following output:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":163} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-18.png" alt="" class="wp-image-163"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Alright, now we are sure that <strong>pagename</strong> parameter on the <strong>dashboard.php</strong> includes PHP files. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now we could try to include our PHP reverse shell here from the SMB share but we don't know where the SMB shares are stored. So we don't know the path of our reverse shell on the target system.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>To figure this out, I did the SMB enumeration again using different tools. Turned out that If I use <strong>smbclient</strong> instead of <strong>smbmap</strong> to look for available shares, one of the shares were leaking it's path.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":164} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-19.png" alt="" class="wp-image-164"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Now we know that the SMB share named "Files" is stored in /etc/Files. Then it's quite obvious that the "Development" share might be residing in <strong>/etc/Development</strong>. <br></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Ok, Now we have got the path to our uploaded reverse shell, which is:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>/etc/Development/reverse_shell.php</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Now we can try to include it in the webpage.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Let's start a netcat listner first to receive the reverse shell.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":166} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-20.png" alt="" class="wp-image-166"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Now, let's open the Admin's dashboard page with our reverse shell path written in the pagename parameter like so:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>https://administrator1.friendzone.red?dashboard.php?image_id=a.jpg&amp;pagename=/etc/Development/reverse_shell</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Notice that we didn't wrote ".php" in the end because it will be appended by the script.<br>Now, opening this URL immediately gives us a reverse shell:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":167} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-21.png" alt="" class="wp-image-167"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As we have gained the shell now, let's go to /var/www and see what else the web page has. While enumeration, we come across a file named <strong>mysql_data.conf</strong></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":168} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-22.png" alt="" class="wp-image-168"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Opening that files gives us credentials of mysql of a user named <strong>friend</strong>. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":171} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-24.png" alt="" class="wp-image-171"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>But it turns out that we don't have permissions to run mysql. So what else can be done with these credentials? Aah! There is an SSH port running on the system. Let's use these credentials there. </p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>ssh friend@friendzone.red</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":173} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-25.png" alt="" class="wp-image-173"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Wow! It worked! Now you can grab the user.txt from /home/friend/user.txt</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Let's move to privelege escalation part now.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I searched for all SUID binaries and saw all the running processes using <strong>netstat</strong> but didn't find anything interesting. Then, I downloaded a tool named <strong>pspy</strong> which let's us monitor unusual/all processes running in the background. I downloaded it from <a href="https://github.com/DominicBreuker/pspy/releases/download/v1.0.0/pspy64">here</a> and transferred it to the target system.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now, running this tool showed a very interesting process which was based on python:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":176} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-26.png" alt="" class="wp-image-176"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As we can see, It's running a python script named <strong>reporter.py</strong> stored in <strong>/opt/server_admin/</strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Looking at the source inside there, there is no such vulnerability in the code which could be exploited. However, you may notice that it's importing the <strong>os module</strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":177} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-27.png" alt="" class="wp-image-177"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>The only way to exploit this python code is to alter the source code of python's os module. Let's see if we can do that. First we need to find where this os module is located. To do that, I ran a <strong>locate</strong> command to find the os.py file.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":178} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-28.png" alt="" class="wp-image-178"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Since target has two versions of python installed, there were two files named os.py. Since the reporter.py is being run by python2.7, we will try to play with the os.py file belonging to python2.7.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I simply wrote a reverse shell command wrapped inside <strong>os.system()</strong> and appended it to os.py. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":179} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-29.png" alt="" class="wp-image-179"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>But, before adding it to the os.py file, we need to start our netcat listner in order to grab the shell.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":180} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-30.png" alt="" class="wp-image-180"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Once the os.py file is written with our payload, within few seconds or a minute, we receive a root shell in our system.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":181} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/07/image-31.png" alt="" class="wp-image-181"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Yay! We got root! Now you can grab the flag and submit it!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>[Thanks for giving it a read! ~xscorp]</strong></p>
<!-- /wp:paragraph -->
