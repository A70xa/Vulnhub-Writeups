# Breach:1 - Writeup 

Breach1 is Beginner to Intermediate-level machine from Vulnhub by mrb3n.<br />
Breach1 is the first in a multi-part series.<br />
The description says: "Solving will take a combination of solid information gathering and persistence. Leave no stone unturned".<br />
Link to the machine: [https://www.vulnhub.com/entry/breach-1,152/](https://www.vulnhub.com/entry/breach-1,152/)
<br />

## Identify The Target

Let's begin with the target's IP address.

![](Pics/ping.png)
<br />
<br />

## Ports Scanning

Let's scan our target.

![](Pics/nmap.png)
<br />
<br />

We got all 65535 ports opened. Let's concentrate on port "80".

![](Pics/nmap2.png)
<br />
<br />

## WebServer Enumeration

Let's visit the webpage.

![](Pics/web.png)
<br />
<br />

We got two usernames, write them down, and inspect the source code.

![](Pics/web2.png)
<br />
<br />

We found a base64 encoded hash, the `/images` directory, and another webpage, let's first decode the hash.

![](Pics/decode.png)
<br />
<br />

Looks like it's **Peter Gibbons**'s username and password, we noticed that name on the webpage! Anyway, run `gobuster` and we got `/images` directory, so let's visit it.

![](Pics/web3.png)
<br />
<br />

Let's visit `/initech.html`.

![](Pics/web4.png)
<br />
<br />

Inspecting the source code.

![](Pics/web5.png)
<br />
<br />

We got another comment, and two links, let's browse them.

![](Pics/web6.png)
<br />
<br />

Inspect the source code.

![](Pics/web7.png)
<br />
<br />

Nothing to see here! Let's download the picture in the link for more examination if that what he meant.

![](Pics/pic.png)
<br />
<br />

We got a comment!! Anyway, moving on, we saw an `Employee portal`, let's visit it.

![](Pics/web8.png)
<br />
<br />

Let's try Peter's creds we found earlier.

![](Pics/web9.png)
<br />
<br />

There are 3 emails in Peter's inbox, let's read them.

![](Pics/web10.png)
<br />
<br />

We saw the name `bill` in the picture we downloaded, moving to the next email.

![](Pics/web11.png)
<br />
<br />

This one is interesting, we have a new username, some SSL cert, and a file to download, let's download it.

![](Pics/file.png)
<br />
<br />

Use the `file` command on it.

![](Pics/file2.png)
<br />
<br />

It's my first time dealing with something like this, so let's google it.

![](Pics/key.png)
<br />
<br />

Ok, there are keys stored in this file. Moving on and digging around I found something.<br />
Click on Peter Gibsons Profile, then click on Content.

![](Pics/web12.png)
<br />
<br />

Click on `SSL implementation test capture`.

![](Pics/web13.png)
<br />
<br />

Let's download that pcap file.

![](Pics/file3.png)
<br />
<br />

Open it with **WireShark**.

![](Pics/shark.png)
<br />
<br />

Notice that it's all encrypted, we can't read anything, so basically it's useless until we decrypt it.

![](Pics/shark2.png)
<br />
<br />

If we go back to the webpage, Peter was talking about (alias, storepassword and keypassword), all set to 'tomcat', we have `index.keystore`, searched on google about how to use it, and found a way.<br />
> Reference: [https://dzone.com/articles/extracting-a-private-key-from-java-keystore-jks](https://dzone.com/articles/extracting-a-private-key-from-java-keystore-jks)<br />

I will use `keytool`.

![](Pics/tool.png)
<br />
<br />

Now, let's decrypt the captured file. Close the captured file, Press `Ctrl + Shift + P`, click on `Protocols` select `TLS` and click `Edit`.

![](Pics/shark3.png)
<br />
<br />

Now open the captured file, right click on any HTTP req/res and click follow TLS or HTTP stream.

![](Pics/web14.png)
<br />
<br />

As we see above, there was a request to `/_M@nag3Me/html` and the server responded with 401 code 'Unauthorized'.

![](Pics/web15.png)
<br />
<br />

In the second request, when adding the 'Authorization' parameter, the server responds with **200** code.<br />
So, let us decrypt that hash.

![](Pics/crack.png)
<br />
<br />
Looks like a username and password, let's browse `/_M@nag3Me/html`.

![](Pics/web16.png)
<br />
<br />

Ok, open Burp, and refresh `/_M@nag3Me/html`.

![](Pics/web17.png)
<br />
<br />

That solved it, when we click on "Accept The Risk", a sign-in prompt popped up.

![](Pics/web18.png)
<br />
<br />

Supply the creds we obtained by decrypting the hash and sign in.

![](Pics/web19.png)
<br />
<br />

With the upload feature, we could get a reverse shell.<br />
Let's make a java reverse shell.

![](Pics/shell.png)
<br />
<br />

Upload it.

![](Pics/shell2.png)
<br />
<br />

Click on `/exp` to trigger it.

![](Pics/shell3.png)
<br />
<br />

## Privelege Escalation

Start to enumerate for higher priv, under the `/home` directory there are two users, when we cat `/etc/passwd` we see user `blumbergh` , and his full name is **Bill Lumbergh**.

![](Pics/priv.png)
<br />
<br />

Earlier, we found a comment `coffeestains` hidden in `bill.png`, maybe it's his password, let's try it.

![](Pics/priv2.png)
<br />
<br />

Searching in our directory and nothing useful, checking our SUDO perm.

![](Pics/priv3.png)
<br />
<br />

We could run `tee` and `tidyup.sh` as root with no password.<br />
The tool `tee` can read from standard input and write to standard output or a file. The script run every 3 minutes, it scans for any content under `swingline` and deletes it.<br />

The fun part is we could use '**tee**' to read from standard input and write it to standard output or let's say (tidyup.sh).<br />
Let's create our reverse shell.

![](Pics/priv4.png)
<br />
<br />

Now, abuse `tee` to read our payload and write it to `tidyup.sh`.

![](Pics/priv5.png)
<br />
<br />

All we have to do now is wait for 3 minutes or less to get a shell.

![](Pics/root.png)
<br />
