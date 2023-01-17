# Digitalworld.local:SNAKEOIL - Writeup

SNAKEOIL is another machine from `Digitalworld.local` series from Vulnhub by Donavan.<br />

The description says: "Recently, Good Tech Inc. has decided to change their application development process. However, their applications look broken and too basic. Is this an application full of snakeoil, or are they insecure too?<br />This goes beyond PEN-200, and some web application development expertise could be helpful".<br />

Link to the machine: [https://www.vulnhub.com/entry/digitalworldlocal-snakeoil,738/](https://www.vulnhub.com/entry/digitalworldlocal-snakeoil,738/)
<br />

## Information Gathering

Let's find the target's IP address.

```bash
┌─[a7@Parrot]─[10.10.10.2]─[~/Desktop/vulnHub/Digitalworld.local:SNAKEOIL]
└──╼ $fping -agq 10.10.10.0/24
10.10.10.1
10.10.10.2
10.10.10.7
```
<br />

## Port Scan

Scan the target for open ports and running services.

![](Pics/nmap.png)
<br />
<br />

## Web Server Enumeration

First, let's visit port 80.

![](Pics/web.png)
<br />
<br />

Just a page telling us `SNAKEOIL` is set up properly, examining the source code, and nothing useful.<br />
Let's visit port 8080.

![](Pics/web2.png)
<br />
<br />

When we click on `Introduction`, we got a post, but look at the URL.

![](Pics/web3.png)
<br />
<br />

Back to the main page, click on `House Rules`.

![](Pics/web4.png)
<br />
<br />

We got username `Patrick`, and again notice the URL.<br />
Now, head back and click on `Useful Links`.

![](Pics/web5.png)
<br />
<br />

We got some information, first, it's **flask**, second, there is some kind of authentication mechanism like JWT (JSON Web Tokens), anyway, there's a link, let's visit it.

![](Pics/jwt.png)
<br />
<br />

It's about configuring **JWT**.<br />
Our next step is to launch `gobuster`.

![](Pics/dir.png)
<br />
<br />

Firstly, let's try to change the directory from `4` to `3`.

![](Pics/error.png)
<br />
<br />

This is the "404 Not Found" message in JSON format which means there is no page with that name.<br />
Moving on, let's browse `test`.

![](Pics/web6.png)
<br />
<br />

Now, open `Burp`, and visit all links on the website including those found by `gobuster` for more analysis.

![](Pics/burp.png)
<br />
<br />

Notice `create`, let's browse it.

![](Pics/web7.png)
<br />
<br />

Let's create a post.

![](Pics/web8.png)
<br />
<br />

Submit.

![](Pics/web9.png)
<br />
<br />

Click on `Edit`.

![](Pics/web10.png)
<br />
<br />

Back to `Burp`, send the interesting requests to `Repeater` for more analysis.<br />
Starting with `login`.

![](Pics/burp2.png)
<br />
<br />

The `METHOD NOT ALLOWED`, change the request method.

![](Pics/burp3.png)
<br />
<br />

We got `BAD REQUEST` which means the request sent to the server is invalid or corrupted, add `username=patrick&password=patrick`.

![](Pics/burp4.png)
<br />
<br />

Let's jump to `registration`, when we send the request it responds with `Wrong Method` so change the method.

![](Pics/burp5.png)
<br />
<br />

Bad request, obviously the method is POST and the page is `registration`, so let's add a new user.

![](Pics/burp6.png)
<br />
<br />

Check our user on `users`.

![](Pics/burp7.png)
<br />
<br />

Back to `login`, log in with the user `admin`.

![](Pics/burp8.png)
<br />
<br />

We got Access_Token! Anyway, we analyzed `users, login, registration`, we have two left, let's analyze `secret`.

![](Pics/burp9.png)
<br />
<br />

Let's send a request to `run`.

![](Pics/burp10.png)
<br />
<br />

Change the method.

![](Pics/burp11.png)
<br />
<br />

This one took me some time till I notice my mistake.

![](Pics/burp12.png)
<br />
<br />

Let's do it.

![](Pics/burp13.png)
<br />
<br />

Where the :) is my secret key.<br />
Let's visit `secret`.

![](Pics/burp14.png)
<br />
<br />

After a few tries, thinking maybe expect a cookie or token, so let's add one.

![](Pics/burp15.png)
<br />
<br />

Tried it but didn't work so I visit the link we found earlier.

![](Pics/cookie.png)
<br />
<br />

Try it again.

![](Pics/burp16.png)
<br />
<br />

We got the secret key, now let's add it in `run`.

![](Pics/burp17.png)
<br />
<br />

We got some kind of command output, we may be able to inject commands.<br />
After playing around, turns out it's `curl`.

![](Pics/burp18.png)
<br />
<br />

The `url` parameter is vulnerable to command injection, so let's send some commands.

![](Pics/burp19.png)
<br />
<br />

Tried to list files and directories.

![](Pics/burp20.png)
<br />
<br />

After browsing these files, found an interesting one.

![](Pics/burp21.png)
<br />
<br />

## Getting Access

Tried to log in with ssh with the obtained creds.

![](Pics/ssh.png)
<br />
<br />

When we check for SUDO permissions, we could run anything as root, let's get root.

![](Pics/root.png)
<br />
<br />