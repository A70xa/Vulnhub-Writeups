# HarryPotter: Nagini - Writeup 

Nagini is the second of three HarryPotter VM series from Vulnhub by Mansoor R.<br />
There are three Horcruxes hidden inside this machine.<br />
Link to the machine: [https://www.vulnhub.com/entry/harrypotter-nagini,689/](https://www.vulnhub.com/entry/harrypotter-nagini,689/) 

<br />

## Target discovery

The first step is to identify the target's IP address.

```
fping -agq 10.10.10.0/24

10.10.10.1
10.10.10.2
10.10.10.8
10.10.10.9
10.10.10.10
```
<br />

## Port scanning

Next, scan the target for open ports and running services.

![](Pics/nmap.png)

There's two open ports `SSH` and `HTTP`.

<br />

## Enumeration

Like the previous one, when we visit the web page a nice image shows, and nothing useful in the source code.

![](Pics/web.png)
<br />
 
So, let's `gobuster` the directories.

![](Pics/go.png)
<br />

We have `/Joomla` and `/note.txt`, let's see the content of `note.txt` first.
<br />

![](Pics/web2.png)
<br />
<br />
Searched for HTTP3 on google and found some info.

![](Pics/web3.png)
<br/>

And a tool in Github for this task. [https://github.com/cloudflare/quiche](https://github.com/cloudflare/quiche)
<br />
Let's follow the steps. I'll clone, and build it on kali machine.

![](Pics/http.png)
<br />
![](Pics/http2.png)
<br />
![](Pics/http3.png)
<br />
![](Pics/http4.png)
<br />
![](Pics/http5.png)
<br />

Add `https://quic.nagini.hogwarts` to /etc/hosts. And now we're ready to visit the site with http3.

![](Pics/http6.png)
<br />

We got `/internalResourceFeTcher.php`, let's navigate to it.

![](Pics/http7.png)
<br />

It's a php file with SSRF. Type `file:///etc/passwd`.

![](Pics/http8.png)
<br />

Nice, I tried a few things but nothing helped. Back to `/joomla` and scan it with `joomscan --url http://10.10.10.10/joomla -ec`

![](Pics/joom.png)
<br />

We have some interesting directories and a `/configuration.php.bak`, download it.

![](Pics/conf.png)
<br />

cat `configuration.php.bak`

![](Pics/db.png)
<br />

And then I stuck :), for hours tried directory bruteforcing and default creds on `/administrator` but nothing worked, asked friends for help, and a hint came out: download this tool and learn about it: [https://github.com/tarunkant/Gopherus](https://github.com/tarunkant/Gopherus)

clone, install, and run `gopherus --exploit mysql`.<br />
Enter the username we found in the config file and the query syntax is: `show databases;`.

![](Pics/db2.png)
<br />

Copy the link and paste it into `internalResourceFeTcher.php?url=<here>`<br />
Refresh the page multiple times

![](Pics/db3.png)
<br />

There is a db called `joomla`. Now type `use joomla; show tables;`

![](Pics/db4.png)
<br />

Now we have the db name and the table. let's dump its contents. `use joomla; select * from joomla_users;`

![](Pics/db5.png)
<br />

Great, We got the username and hashed password, tried to decode it but failed. The second solution is to change it with my hashed password `Password123`.

![](Pics/db6.png)
<br />

Copy, paste the link, and refresh the page.

![](Pics/db7.png)
<br />
<br />

If we go back to  `joomscan` result, there is `/joomla/administrator`. Let us visit it.

![](Pics/site.png)
<br />
<br />

Login with the Username: `site_admin` and the Password: `Password123`.

![](Pics/site2.png)
<br />

I searched on google for a way to get a reverse shell.
> Referense [https://www.hackingarticles.in/joomla-reverse-shell/](https://www.hackingarticles.in/joomla-reverse-shell/)

So let's do it. Click on `Extensions` then `Templates`.

![](Pics/site3.png)
<br />
<br />
We will use `Beez3`, Click `Details and Files`, and Click on `index.php`.

![](Pics/site4.png)
<br />

Now, replace the original `index.php` with  `/usr/share/webshells/php/php-reverse-shell.php`.
<h4>Remember to change the Port and IP</h4>

![](Pics/site5.png)
<br />
<br />

Press save, open your listener, back to `joomla`, and press template preview.

![](Pics/site6.png)
<br />
<br />
<br />
<br />
<br />
![](Pics/shell3.png)
<br />

Now we're in. I found two users in `home` directory (Snape and Hermoine), listing user snape's directory, and found `.creds.txt` contains a password. I tried to ssh snape with the obtained password.

![](Pics/shell3.png)
<br />

Now, heading to Hermoine's directory and finding the first Horcrux(flag), We don't have permission to open it, but there's a SUID binary that belongs to her, it just copies a file to another location. So, I copied the `Horcrux2.txt` to `flag.txt`, now we can open it.

![](Pics/shell4.png)
<br />

## Privilege Escalation

The user Snape doesn't have priv, but maybe Hermoine does. We can do a little trick by generating a new ssh key and copying it to Hermoine's `.ssh` directory. In that way, we could ssh Hermoine.

![](Pics/shell5.png)
<br />

SSH to Hermoine.

![](Pics/shell6.png)
<br />

Now, let's upload `linpeas` and `pspy`.

![](Pics/priv.png)
<br />

But, nothing useful. Start digging more. Under `/var/www/html` there's the second Horcrux(flag).

![](Pics/priv2.png)
<br />

Now we have 2 Horcruxes. let's get root.
<br />

## Root Access

In Hermoine's directory, there's `.mozilla` directory contains credentials. there is a tool called `firepwd` which allows us to recover the credentials.

![](Pics/root.png)
<br />

We have the root's password, let's ssh.

![](Pics/root2.png)
<br />
