<h1>NullByte Walkthrough (Vulnhub)</h1>


<!-- wp:paragraph -->
<p>Hello my fellow hackers! Today we are going to do the <a href="https://www.vulnhub.com/entry/nullbyte-1,126/">NullByte</a> machine from vulnhub. I am assuming that you have already set it up on virtual box/vmware or any other virtual environment and it's running. If you haven't done it yet, I will recommend you to search Google to do that.</p>
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

<!-- wp:image {"id":12} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image.png" alt="" class="wp-image-12"/><figcaption>Output of netdiscover tool</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As you can see that it listed two connected devices. First one belongs to the router itself while the second one is of the target system. I have hidden the MAC addresses for security reasons.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now we have got the IP of the target system, now it's time for enumeration. We will do an nmap scan for port scanning. I used the following command for port scanning through nmap.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># nmap -sS -sC -sV  -A -p- 192.168.43.148 -oN nmap.txt</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>It requires a bit explanation! The <strong>-sS</strong> flag is used for <strong>TCP SYN scan</strong>. <strong>-sC</strong> is used for using <strong>default NSE scripts</strong>, <strong>-sV</strong> is used for <strong>version detection</strong>, <strong>-A </strong>for <strong>banner grabbing</strong>, <strong>-p- </strong>for <strong>scanning all ports</strong>. Following our target IP, the last flag <strong>-oN </strong> is for <strong>output in normal text form</strong> and the file name is supplied as an argument to the flag. I have marked the ports with red arrow.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":18} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/nmap.png" alt="" class="wp-image-18"/><figcaption>Output of nmap scan</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Let's first have a look at HTTP service running on the target host on port 80.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":19} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-1.png" alt="" class="wp-image-19"/><figcaption>Website running on target host</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Just like every other webpage inspection, we will first start by checking the source code of the web page to see if there is anything interesting there. Turned out there isn't anything useful.</p>
<!-- /wp:paragraph -->

<!-- wp:syntaxhighlighter/code {"language":"xml"} -->
<pre class="wp-block-syntaxhighlighter-code brush: xml; notranslate">&lt;html>
&lt;head>&lt;title>Null Byte 00 - level 1&lt;/title>&lt;/head>
&lt;body>
&lt;center>
&lt;img src="main.gif">
&lt;p> If you search for the laws of harmony, you will find knowledge. &lt;/p>
&lt;/center>	
&lt;/body>
&lt;/html>
</pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:paragraph -->
<p>After struggling for hours with it, I thought there could be something inside the <strong>main.gif</strong> file. I got this hint from the line "<em>If you search for the laws of harmony, you will find knowledge.</em>".<br>I downloaded the main.gif by issuing the following command:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>wget http://192.168.43.148/main.gif</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":21} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-2.png" alt="" class="wp-image-21"/><figcaption>Output of wget command </figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Now, to inspect the gif file, I used an amazing tool called <strong>exiftool</strong>. This tool is well known for looking at metadata of an image file. </p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># exiftool main.gif</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":22} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-3.png" alt="" class="wp-image-22"/><figcaption>Output of exiftool for main.gif inspection</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Did you find anything weird looking or unusual here. If not, then look at the highlighted line. That comment seems unusual. Just store it somewhere, we might be needing it later.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>NOTE: I haven't mentioned the directory brute force tools like dirbuster, dirb, gobuster etc because they didn't find anything special  here. </strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>After roaming here and there in the website for a while, I tested the string<strong>"kzMb5nVYJw</strong>" which we got using exiftool in URL to check if there's any page/directory named so, Luckily I succeed! We got an authentication page on it. <br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":23} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-4.png" alt="" class="wp-image-23"/><figcaption>Authentication page inside the directory</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Again a web page! Remember what was the first step to inspect it? Yea, check the source code.</p>
<!-- /wp:paragraph -->

<!-- wp:syntaxhighlighter/code {"language":"xml"} -->
<pre class="wp-block-syntaxhighlighter-code brush: xml; notranslate">&lt;center>
&lt;form method="post" action="index.php">
Key:&lt;br>
&lt;input type="password" name="key">
&lt;/form> 
&lt;/center>
&lt;!-- this form isn't connected to mysql, password ain't that complex --!></pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:paragraph -->
<p>There isn't anything special in the source code BUT there is a line which might be a hint to the attacker. Look at the line "<em>this form isn't connected to mysql, password ain't that complex</em>". If you have some experience in breaking boxes, you might be aware that reality is totally opposite of what is mentioned in source codes generally. This is done to prevent attacker from making even an attempt to connect to MySQL. But it's a clear and true hint that the password isn't that complex. This leads us to brute force attack. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I used <strong>hydra </strong>tool to brute force the form for passwords. To work with hydra, we need to tell the hydra how the request string or request parameters looks like in the HTTP request. To get that information, I used <strong>burpsuite </strong> tool to intercept HTTP requests and examined the request. I entered a random key(password) and hit Enter. The request generated was:</p>
<!-- /wp:paragraph -->

<!-- wp:html -->
<pre class="wp-block-code"><code>POST /kzMb5nVYJw/index.php HTTP/1.1
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

<p style="color:Red;">key=testKey</p></code></pre>
<!-- /wp:html -->

<!-- wp:paragraph -->
<p>I want you to notice few things here. First is that our entered key is being passed through the <strong>key</strong> variable. Since the variable is included in the request's body, we know that this form is using <strong>POST</strong> method. Now we need to pass the <strong>key</strong> variable to hydra so that it can make appropriate request to the server. Following is the <strong>hydra </strong>syntax I used for brute forcing the key.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>hydra 192.168.43.148 -V -l "" -P /home/xscorp/Desktop/NullByte/brute_pass.txt  http-post-form "/kzMb5nVYJw/index.php:key=^PASS^:invalid"</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>The <strong>-V</strong> flag is used for <strong>verbose</strong>,<br><strong>-l </strong>is for specifying the username. Since there is no username here, so we passed the ""<br><strong>-P </strong>is for specifying the password file. Here, I used a custom wordlist named <strong>brute_pass.txt</strong>. You could choose whatever you want. <br><strong>http-post-form </strong>is the name of the Hydra's module which is used for brute forcing HTTP-forms password which uses POST method to send data to server.<br>Next, the http-post-form method accepts 3 colon(":") separated values which are in the following order:<br><em>&lt;target_page_URL&gt;<strong>:</strong>&lt;paramaters&gt;<strong>:</strong>&lt;attempt_failed_string&gt;</em><br>So, here we have specified the target page URL as the index.php located in <strong>kzMb5nVYJw</strong>.<br>The <strong>key</strong> variable to be passed should be written at the second place. The <strong>^PASS^</strong> is a type of regular expression which tells the hydra to put values in place of it. Then ultimately, we need to specify the attempt failed string. This tells the hydra what a failed attempt looks like. Otherwise there's no way Hydra can identify if a password guess was correct or not. To get the attempt_failed_string, we can try a wrong attempt and check how the HTML is changing in the page. Turned out that the page displays something like "Invalid key". So, we passed the "Invalid" at third place for failed attempt identification. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":25} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-5.png" alt="" class="wp-image-25"/><figcaption>Output of hydra tool</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>So Now we have the right key <strong>elite</strong>. Entering the key in the web page will lead us to a new page.<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":27} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-6.png" alt="" class="wp-image-27"/><figcaption>Page after entering the right key</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>After passing several wrong usernames, I passed an empty string and BOOM, it showed me all the usernames.<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":28} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-7.png" alt="" class="wp-image-28"/><figcaption>Page's output after passing an empty string as username</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Hmmn, Look at those names of attributes. They look a lot like columns of a table in database. If an empty string revealed so much about it, then there are chances that it is vulnerable to SQL injection attack. Trying <strong>' </strong>and <strong>" </strong>broke the syntax and displayed the following error message.<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":29} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-8.png" alt="" class="wp-image-29"/><figcaption>Error on passing " as username</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Ok, So now there is no doubt that this is vulnerable to SQL injection attack. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I tried different payloads manually like " OR SHOW DATABASES; " but nothing helped. After almost half an hour, I realized that we need to be good enough to understand and break it. At this point, I googled for walkthrough so that I can get past it. But unfortunately, there were no walkthroughs which show it by manually passing the attack payload. So I left that too and used automated tools like <strong>sqlmap</strong> to take advantage of that SQL injection vulnerability.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The <strong>sqlmap</strong> is a powerful tool which is used for automated vulnerability scanning. I used the following command to search for the vulnerabilities and ultimately to list the existing databases:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># sqlmap --url http://192.168.43.148/kzMb5nVYJw/420search.php?usrtosearch= --dbs</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Here, <strong>--url</strong> is used for specifying the URL of the target page and <strong>--dbs</strong> is used for displaying the list of existing databases once the vulnerability is exploited.<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":30} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-9.png" alt="" class="wp-image-30"/><figcaption>Output of sqlmap</figcaption></figure>
<!-- /wp:image -->

<!-- wp:image {"id":31} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-10.png" alt="" class="wp-image-31"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Ok, now we have access to database. That <strong>seth</strong> database looks interesting. Let's check that out. The following commands lists the tables inside the "seth" database:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># sqlmap --url http://192.168.43.148/kzMb5nVYJw/420search.php?usrtosearch= -D seth --tables</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":32} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-11.png" alt="" class="wp-image-32"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Looks like we are heading the right way. Let's dump all the records inside the table <strong>users</strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># sqlmap --url http://192.168.43.148/kzMb5nVYJw/420search.php?usrtosearch= -D seth -T users --dump</code></pre>
<!-- /wp:code -->

<!-- wp:image {"id":33} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-12.png" alt="" class="wp-image-33"/><figcaption>users table dump from the database</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>The <strong>isis</strong> seems uninteresting. However <strong>ramses</strong> user has some encrypted/encoded password. Again, a little bit experienced people will know just by a sight that it's a base64 encoded string. Let's decrypt it!</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># echo "YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE=" | base64 --decode</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>The encoded string needed to be padded. So I added an extra '=' at the end. The output/decoded string is: </p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>c6d6bd7ebf806f43c76acc3681703b81</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Now, It turned out that this weird string is an md5 hash. Since I didn't know how to use tools like hashcat, I used an online md5 decoder like <a href="https://www.md5online.org/md5-decrypt.html">this</a> one to decrypt the md5 hash.<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":34} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-13.png" alt="" class="wp-image-34"/><figcaption>Decoded hash</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p><strong>omega</strong>, we have got it. But what to do with this value? Till now, all I know is that this password belongs to the user <strong>ramses</strong> but where should I use it? <br>Well, If you look carefully at the nmap result, there is a <strong>SSH</strong> service running on the target. Let's try these credentials there.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># ssh ramses@192.168.43.148 -p 777</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>In the above command, we are using the <strong>ssh</strong> client to connect to remote host with the username <strong>ramses</strong> at port 777 specified using <strong>-p</strong> option. Entering the password as <strong>omega</strong> when prompted, we got shell! Yay!</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":35} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-14.png" alt="" class="wp-image-35"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Now, It's time for privilege escalation. Listing all the directories including hidden directory reveals that there is a file named <strong>.bash_history</strong> which generally contains the recent commands used against that user's home directory. Reading it reveals the existence of a file named <strong>procwatch</strong>. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":41} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-17.png" alt="" class="wp-image-41"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>This file might be the key to root access. But first, we need to find out where it is located. To do this, we can use <strong>find </strong>command.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>ramses@NullByte:~$ find / -type f -name procwatch</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p><strong>-type f </strong>specifies that we are looking for a file and not a directory. <strong>-name procwatch </strong>tells the find tool that the name of the file to be searched is <strong>procwatch</strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":36} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-15.png" alt="" class="wp-image-36"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>We have got the location of <strong>procwatch</strong>. Let's see what's there!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Getting inside that directory, I found out that it's an executable file. And what's beautiful about it is that it's a SUID binary which means this can be executed by anyone with the privileges of the user who owns this file. And luckily, the owner of this file is <strong>root</strong>, that means if we can hijack it's execution, we can execute arbitrary code or gain a shell.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":42} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-18.png" alt="" class="wp-image-42"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Now, let's execute the <strong>procwatch </strong>and examine it's behavior.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":43} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-19.png" alt="" class="wp-image-43"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Wait! Doesn't this look familiar? That's what the <strong>ps</strong> command does in linux i.e. listing the running processes of the current session. Following is the output of <strong>ps</strong> command.  </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":44} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-20.png" alt="" class="wp-image-44"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Compare both the outputs and see the similarity. That's because <strong>procwatch </strong>is using <strong>ps </strong>from inside.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now, there is a chance that <strong>procwatch </strong>might be using relative path of the <strong>ps </strong>tool instead of absolute path. Sounds confusing? Here's how both type of paths looks like:</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Relative path:  <code>ps</code><br>Absolute path: <code>/bin/ps</code></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Relative path shouldn't be used in binaries as it can be abused using path manipulation technique. Let's check the value of <strong>PATH</strong> environment variable.</p>
<!-- /wp:paragraph -->

<!-- wp:html -->
<pre class="wp-block-code"><code>ramses@NullByte:/var/www/backup$ echo $PATH
<div>/usr/local/bin:/usr/bin:<b style="color:Red;">/bin</b>:/usr/local/games:/usr/games</div></code></pre>
<!-- /wp:html -->

<!-- wp:paragraph -->
<p>Notice that <strong>/bin</strong> there. The <strong>procwatch</strong> must be executing <strong>ps</strong> from this path. Can you think of a way of hijacking it? Yes, we can modify the <strong>PATH</strong> and add a custom path for <strong>ps</strong> so that <strong>procwatch </strong>can call our variant of <strong>ps </strong>executable. Let's add any arbitrary writable path like <strong>/tmp</strong> to the <strong>PATH</strong>.<br><code>ramses@NullByte:/var/www/backup$ export PATH=/tmp:$PATH</code></p>
<!-- /wp:paragraph -->

<!-- wp:html -->
<pre class="wp-block-code"><code>ramses@NullByte:/var/www/backup$ echo $PATH
<div><b style="color:Red;">/tmp</b>:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games</div></code></pre>
<!-- /wp:html -->

<!-- wp:paragraph -->
<p>Now, the value of <strong>PATH </strong>variable has changed. Now <strong>procwatch</strong> will look for <strong>ps</strong> in <strong>/tmp </strong>before <strong>/bin</strong>. <br>Now let's navigate to /tmp and create our own executable named <strong>ps </strong>for the <strong>procwatch</strong> to execute with root privileges.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>All we want to write in our<strong> ps </strong>file is the command for spawning a shell. Since this file will be executed with <strong>root</strong> privileges, we will gain the root shell. I wrote this in <strong>ps </strong>file to spawn a shell:<br><code>ramses@NullByte:/tmp$ echo "/bin/sh -i" &gt; ps</code><br>Here, <strong>echo </strong>command will create a file if doesn't exist already and write whatever is next into it. <strong>"/bin/sh -i" </strong>is for spawning an <strong>interactive </strong>shell. <strong>&gt; ps</strong> is used for writing this to <strong>ps </strong>file.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Once our file is written, now we need to give it global execution permission so that It can be executed by any user(root in this case). This can be done using <strong>chmod</strong>.<br><code>ramses@NullByte:/tmp$ chmod +x ./ps</code></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now, everything is set and it's showtime. It's time to execute the <strong>procwatch </strong>binary and see what happens. If everything goes well and our steps are correct, we must get root shell.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":51} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-21.png" alt="" class="wp-image-51"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Hurrey! We got root! We have finally hacked this box. Now let's get the flag located in <strong>/root </strong>directory.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":52} -->
<figure class="wp-block-image"><img src="https://hackersfoodhome.files.wordpress.com/2019/05/image-22.png" alt="" class="wp-image-52"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>I have hidden some part of the flag so that you guys can't directly see it. Try it yourself and you will learn a lot. Seeing a walkthrough is simple, but trying the box even after watching the walkthrough is not that easy. Good Luck! Happy Hacking!</p>
<!-- /wp:paragraph -->
