# Hack-Me-Please Writeups

"Hack Me Please" is an easy machine from Vulnhub by Saket Sourav.

This is an OSCP-like machine, so we don't need any bruteforcing.

Tested on Virtualbox.

Link to the machine: [https://www.vulnhub.com/entry/hack-me-please-1,731/](https://www.vulnhub.com/entry/hack-me-please-1,731/)
<br />
<br />
## Identify the target

I'm using this machine in host-only network.

First, we need to identify the machine's IP address.

`fping -agq 10.10.10.0/24`

![](pics/fping.png)
<br />
<br />
## Scan Open Ports
Scan the target for open ports and running services.

```
nmap -sV -sC -oN nmap.log 10.10.10.10
```
![](pics/nmap.png)

We have Apache and MySQL.
<br />
<br />
## Examine Web Server
We have a default website, inspect the source code, nothing interesting.

![](pics/web1.png)
<br />
<br />
## Enumerate the directories
Run Gobuster for directories enumeration.

![](pics/go1.png)
<br />
<br />
When we visit any directory, We got a forbidden message.

After houre diggin around, i checked the website's source code, and there is `js/main.js`.

Opened it and nothing special, but strange comment.

![](pics/js-main.png)
<br />
After searching around, It's DMS (Document Management System).

> Reference: [https://www.seeddms.org/index.php?id=2](https://www.seeddms.org/index.php?id=2)

> Exploit: [https://www.exploit-db.com/exploits/47022](https://www.exploit-db.com/exploits/47022)

> and a repository: [https://sourceforge.net/p/seeddms/code/ci/5.1.22/tree/](https://sourceforge.net/p/seeddms/code/ci/5.1.22/tree/)
<br />

Ok, checking the path `http://10.10.10.10/seeddms51x/seedms-5.1.22/` and a login panel appears.

![](pics/login1.png)
<br />
<br />
The description said that brute force is not required. So, We have to keep digging.
<br />
Back to the repository, there is `/conf` file.

![](pics/seeddms.png)
<br />
<br />
navigates to `http://10.10.10.10/seeddms51x/conf/`

![](pics/conf.png)
<br />
<br />
Forbidden message. There is a `.htaccess` file that restricts directory browsing.

![](pics/seeddms2.png)
<br />
<br />
Let's visit this path `http://10.10.10.10/seeddms51x/conf/settings.xml` and finally, database credentials.

![](pics/dbcred.png)
<br />
<br />
Connect to the database with mysql tool or mycli.
<br />
`mysql -u <username> -p <password> -h <db_ip>`
<br />
There is a database called `"seeddms"`, we used it, and then we looked for the tables.

![](pics/sql5.png)
<br />

And we found many tables, there is two interesting tables `users` and `tblUsers`, Let's see their contents.

![](pics/sql2.png)

![](pics/sql3.png)
<br />

From the first table, We have a username and password, and the second there is admin and hash, I failed to decrypt it.
<br />
So, Why don't we change the admin's hash.

![](pics/sql4.png)
<br />

Back to the login panel. Login with the new Password we changed.

![](pics/login.png)
<br />
<br />
## Reverse Shell
Now we loged in, there is `add document` feature. That means we could upload a file like reverse shell.

![](pics/rshell.png)
<br />

Let's upload a reverse shell.
<br />

Find a shell online: [revshells](https://www.revshells.com/) or [pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
<br />

In kali there's some shells by default `/usr/share/webshells/`

![](pics/phpshell.png)

![](pics/rshell2.png)

![](pics/rshell3.png)
<br />

When We click on the file we see ID.

![](pics/rshell4.png)
<br />

Back to the Exploit: [exploit-db](https://www.exploit-db.com/exploits/47022)

![](pics/edb.png)
<br />

our reverse shell is ready, We are listening on port 5555, we had our reverse shell file ID.
all we need is to visit the link, in my case `http://10.10.10.10/seeddms51x/data/1048576/4/1.php`
<br />

We got a dummy shell.

![](pics/shell.png)
<br />

Upgrade the shell: [https://infosecwriteups.com/pimp-my-shell-5-ways-to-upgrade-a-netcat-shell-ecd551a180d2](https://infosecwriteups.com/pimp-my-shell-5-ways-to-upgrade-a-netcat-shell-ecd551a180d2)
<br />

![](pics/shell2.png)
<br />
<br />
Now we have intelligent shell, let's find any user.
<br />
![](pics/shell3.png)
<br />
<br />
We already got `saket's password` from the database.
<br />
![](pics/user.png)
<br />
<br />

Now, checked the sudo permissions.
<br />
![](pics/user2.png)
<br />
<br />
The user can run any command as sudo. 
<br />
switch to root.
<br />
![](pics/root.png)
