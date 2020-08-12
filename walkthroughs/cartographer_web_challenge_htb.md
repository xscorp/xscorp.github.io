---
layout: post
---

# Cartographer[web] HTB walkthrough

<!-- wp:paragraph -->
<p>Hello everyone, today we are gonna do the Cartographer web challenge from HTB. This challenge is based on basic fuzzing. So let's jump in!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>This is the page which appears on visiting the challenge URL:<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":197,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://hackersfoodhome.files.wordpress.com/2019/10/image-1.png?w=1024" alt="" class="wp-image-197"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As can be seen, there is a login page on it. I tried very common and basic credentials like admin:admin, admin:pass etc but none of them worked.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>As brute forcing is considered as a bad approach/technique, I thought to try it in the end when I have no other option left. So I started inspecting the source code of login page to find some juice there.</p>
<!-- /wp:paragraph -->

<!-- wp:syntaxhighlighter/code {"language":"xml"} -->
<pre class="wp-block-syntaxhighlighter-code">&lt;html>
&lt;head>
    &lt;title>Cartographer - Login&lt;/title>
    &lt;link rel='stylesheet' href='style.css' type='text/css' />
&lt;/head>
&lt;body>
    &lt;div class="content-container">
        &lt;div class="blur">&lt;/div>
    &lt;/div>
    &lt;div class="loginform">
        &lt;center>
            &lt;img src="logo.png" />&lt;br>&lt;br>
            &lt;form method="post">
                &lt;input class="textbox" type="text" name="username" placeholder="Username">&lt;br>&lt;br>
                &lt;input class="textbox" type="password" name="password" placeholder="Password">&lt;br>&lt;br>
                &lt;input class="loginbutton" type="submit" value="Login">
            &lt;/form>
        &lt;/center>
    &lt;/div>
&lt;/body>
&lt;/html</pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:paragraph -->
<p>Unfortunately, there was nothing there.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>So my next step was to test for SQL injection vulnerability in those form fields. So I gave a try with basic SQLi payload (<strong>' or '1=1' #</strong>) in username field and got this weird reply:<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":203,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://hackersfoodhome.files.wordpress.com/2019/10/image-2.png?w=1024" alt="" class="wp-image-203"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p><strong>"Cartographer Is Still Under Construction!"</strong>. It was not the message itself that was interesting, but the query in URL where this page was found.<br></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":205,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://hackersfoodhome.files.wordpress.com/2019/10/image-3.png?w=664" alt="" class="wp-image-205"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Alright, now we have got an interesting page named <strong>panel.php </strong> and an interesting variable named <strong>info </strong>that has currently been set to <strong>home</strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Since that <strong>info </strong>variable is loading some page, I tried testing it for Local File Inclusion but no luck. At the end, the only option left was to fuzz the variable.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Here, we have two things to fuzz. First the variable name and second is the value of that variable. I tried fuzzing the variable name but got nothing. So now the only option left was to fuzz different values for the same variable <strong>info</strong>. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>But before fuzzing, I need a property that can differentiate the right result from other results. I opened the <strong>Burpsuite</strong> and intercepted the request with wrong value of <strong>info</strong> to see how the "not found/failed" response looks like.<br>I found that the unimportant response has a content length of 375 characters. This info can be used to filter out required results when using tools like wfuzz.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":217,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://hackersfoodhome.files.wordpress.com/2019/10/image-6.png?w=1024" alt="" class="wp-image-217"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Once I got the property(Content length) which differentiates right results from wrong results, now the next thing to look for is a good wordlist to fuzz the parameter value with.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I searched for parameter fuzzing wordlists in google and came across this wordlist from SecList: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt" target="_blank">https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt</a> . This list contains words that are generally found as parameter names or parameter values in web applications.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now we have got everything we need, it's time to fuzz the parameter value using <strong>wfuzz</strong> tool. The command I used was:</p>
<!-- /wp:paragraph -->

<!-- wp:syntaxhighlighter/code {"language":"bash","lineNumbers":false} -->
<pre class="wp-block-syntaxhighlighter-code">wfuzz -z file,params.txt -b "PHPSESSID=r2dut711gvbjmp0kn7v9ca8hf5" --hh 375 -t 60 "http://docker.hackthebox.eu:36539/panel.php?info=FUZZ"</pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:paragraph -->
<p>Command explaination:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li><strong>-z file,params.txt </strong>: Option to specify payload name. Here we have specified payload type as <strong>file</strong> and then have specified the name of the file to use for fuzzing.<br></li><li><strong>-b </strong>: Option to pass Cookies. Without specifying cookies, you will be redirected to the front login page automatically and hence will get wrong results. You can use Burpsuite to get this cookie after intercepting the requested URL from browser.<br></li><li><strong>--hh</strong> : Option to hide responses of specific length. Here, we know that unimportant response has a length of 375 characters, we don't want it to appear in results. <br></li><li><strong>-t </strong>: Option to specify number of threads to speed up the fuzzing process.<br></li><li><strong>"FUZZ"</strong> : That FUZZ string in the URL is used as a placeholder for the place that will be fuzzed by the contents of the specified wordlist.</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>The above command produces the following output:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":219,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://hackersfoodhome.files.wordpress.com/2019/10/image-8.png?w=1024" alt="" class="wp-image-219"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>As you can see, other than the known value <strong>home</strong>, <strong>flag</strong> was the only value found for the variable <strong>info</strong>. Let's use this and see what's in there.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>So, I requested this URL through browser:</p>
<!-- /wp:paragraph -->

<!-- wp:syntaxhighlighter/code {"language":"bash","lineNumbers":false} -->
<pre class="wp-block-syntaxhighlighter-code">http://docker.hackthebox.eu:36539/panel.php?info=flag</pre>
<!-- /wp:syntaxhighlighter/code -->

<!-- wp:paragraph -->
<p>It immediately showed me this output which was nothing but the challenge completion flag. Yay!</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":222,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="https://hackersfoodhome.files.wordpress.com/2019/10/image-10.png?w=1024" alt="" class="wp-image-222"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>I have hidden the flag for security reasons and also so that you can try this challenge out yourself!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Happy Hacking! :-) <br>@xscorp</p>
<!-- /wp:paragraph -->
