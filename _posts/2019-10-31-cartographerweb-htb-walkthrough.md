---
layout: post
title: "Cartographer[web] HTB walkthrough"
permalink: ctf-writeup/cartographer
postday: 2019/10/31
posttime: 23_03
tags: [CTF Walkthrough]
categories: [CTF Walkthrough]
image:
    path: /assets/img/cartographer/image-1.png?w=1024
---
Hello everyone, today we are gonna do the Cartographer web challenge from HTB. This challenge is based on basic fuzzing. So let's jump in!

This is the page which appears on visiting the challenge URL:  


![](/assets/img/cartographer/image-1.png?w=1024)

As can be seen, there is a login page on it. I tried very common and basic credentials like admin:admin, admin:pass etc but none of them worked.

As brute forcing is considered as a bad approach/technique, I thought to try it in the end when I have no other option left. So I started inspecting the source code of login page to find some juice there.
    
    
    <html>
    <head>
        <title>Cartographer - Login</title>
        <link rel='stylesheet' href='style.css' type='text/css' />
    </head>
    <body>
        <div class="content-container">
            <div class="blur"></div>
        </div>
        <div class="loginform">
            <center>
                <img src="logo.png" /><br><br>
                <form method="post">
                    <input class="textbox" type="text" name="username" placeholder="Username"><br><br>
                    <input class="textbox" type="password" name="password" placeholder="Password"><br><br>
                    <input class="loginbutton" type="submit" value="Login">
                </form>
            </center>
        </div>
    </body>
    </html

Unfortunately, there was nothing there.

So my next step was to test for SQL injection vulnerability in those form fields. So I gave a try with basic SQLi payload (**' or '1=1' #**) in username field and got this weird reply:  


![](/assets/img/cartographer/image-2.png?w=1024)

**"Cartographer Is Still Under Construction!"**. It was not the message itself that was interesting, but the query in URL where this page was found.  


![](/assets/img/cartographer/image-3.png?w=664)

Alright, now we have got an interesting page named **panel.php ** and an interesting variable named **info **that has currently been set to **home**.

Since that **info **variable is loading some page, I tried testing it for Local File Inclusion but no luck. At the end, the only option left was to fuzz the variable.

Here, we have two things to fuzz. First the variable name and second is the value of that variable. I tried fuzzing the variable name but got nothing. So now the only option left was to fuzz different values for the same variable **info**. 

But before fuzzing, I need a property that can differentiate the right result from other results. I opened the **Burpsuite** and intercepted the request with wrong value of **info** to see how the "not found/failed" response looks like.  
I found that the unimportant response has a content length of 375 characters. This info can be used to filter out required results when using tools like wfuzz.

![](/assets/img/cartographer/image-6.png?w=1024)

Once I got the property(Content length) which differentiates right results from wrong results, now the next thing to look for is a good wordlist to fuzz the parameter value with.

I searched for parameter fuzzing wordlists in google and came across this wordlist from SecList: <https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt> . This list contains words that are generally found as parameter names or parameter values in web applications.

Now we have got everything we need, it's time to fuzz the parameter value using **wfuzz** tool. The command I used was:
    
    
    wfuzz -z file,params.txt -b "PHPSESSID=r2dut711gvbjmp0kn7v9ca8hf5" --hh 375 -t 60 "http://docker.hackthebox.eu:36539/panel.php?info=FUZZ"

Command explaination:

  * **-z file,params.txt **: Option to specify payload name. Here we have specified payload type as **file** and then have specified the name of the file to use for fuzzing.  

  * **-b **: Option to pass Cookies. Without specifying cookies, you will be redirected to the front login page automatically and hence will get wrong results. You can use Burpsuite to get this cookie after intercepting the requested URL from browser.  

  * **\--hh** : Option to hide responses of specific length. Here, we know that unimportant response has a length of 375 characters, we don't want it to appear in results.   

  * **-t **: Option to specify number of threads to speed up the fuzzing process.  

  * **"FUZZ"** : That FUZZ string in the URL is used as a placeholder for the place that will be fuzzed by the contents of the specified wordlist.

The above command produces the following output:

![](/assets/img/cartographer/image-8.png?w=1024)

As you can see, other than the known value **home**, **flag** was the only value found for the variable **info**. Let's use this and see what's in there.

So, I requested this URL through browser:
    
    
    http://docker.hackthebox.eu:36539/panel.php?info=flag

It immediately showed me this output which was nothing but the challenge completion flag. Yay!

![](/assets/img/cartographer/image-10.png?w=1024)

I have hidden the flag for security reasons and also so that you can try this challenge out yourself!

Happy Hacking! :-)   
@xscorp
