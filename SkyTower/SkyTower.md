# SkyTower - Writeup 

SkyTower is an intermediate-level machine from Vulnhub by Telspace.<br />
This CTF was designed by Telspace Systems for the CTF at the ITWeb Security Summit and BSidesCPT (Cape Town).<br />
Link to the machine: [https://www.vulnhub.com/entry/skytower-1,96/](https://www.vulnhub.com/entry/skytower-1,96/)
<br />

## Recon

First, let's identify our target.

![](Pics/fping.png)
<br />
<br />

## Scanning

Let's find out which services are running.

![](Pics/nmap.png)
<br />
<br />

We have a filtered SSH, a web server, and a squid proxy server.

## Enumeration

Let's browse the website.

![](Pics/web.png)
<br />
<br />

It's a login page.<br />
Let's check if it's vulnerable to SQLi by adding `'`.

![](Pics/web1.png)
<br />
<br />

After we submit, we got an error in our query. This means it's vulnerable to SQLi.<br />
Let's try another one.

![](Pics/web2.png)
<br />
<br />

Press `Login`.

![](Pics/web3.png)
<br />
<br />

As we see above, there's some kind of character filter.<br />
We could change `or` with `||`, let's try it.

![](Pics/web4.png)
<br />
<br />

Submit.

![](Pics/web5.png)
<br />
<br />

We got `John`'s ssh creds, but ssh is filtered.<br />
We have a `squid` proxy server, we could route our ssh through the proxy server.<br />
First, let's configure our `proxychains.conf` file.

![](Pics/prox.png)
<br />
<br />

Now ssh to the machine with proxy.

![](Pics/ssh.png)
<br />
<br />

The connection is closed, I don't know why but we could use `-t` to force pseudo-terminal.

![](Pics/ssh2.png)
<br />
<br />

When we check `.bashrc` file, the last line force the ssh connection to close, we could remove or change the file name and reconnect to the machine.

![](Pics/ssh3.png)
<br />
<br />

## Privelege Escalation

Checking our sudo perms, and we don't have any.

![](Pics/sudo.png)
<br />
<br />

We have two more users on this machine.

![](Pics/priv.png)
<br />
<br />

After enumerating, we found `login.php` under `/var/www`, this file contains creds for MySQL.

![](Pics/priv2.png)
<br />
<br />

Let's login.

![](Pics/sql.png)
<br />
<br />

There's a database called `SkyTech`, let's dump its contents.

![](Pics/sql2.png)
<br />
<br />

We got the passwords for the users found earlier, let's login with Sara.

![](Pics/ssh4.png)
<br />
<br />

Ok, rename `.bashrc` and log in again.


![](Pics/ssh5.png)
<br />
<br />

Checking Sara's Perms.

![](Pics/priv3.png)
<br />
<br />

We could use `cat/ls` as root with no password.<br />
Let's abuse this perms by path-traversing the directories.

![](Pics/priv4.png)
<br />
<br />

Finally, let's read flag.txt.

![](Pics/root.png)
<br />