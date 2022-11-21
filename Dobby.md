# Hogwarts: Dobby - Writeup 

Dobby is an Easy level machine from Vulnhub by BLY.<br />
The description says: "dobby needs to be root to help harry potter, dobby needs to be a free elf".<br />
Link to the machine: [https://www.vulnhub.com/entry/hogwarts-dobby,597/](https://www.vulnhub.com/entry/hogwarts-dobby,597/) 

<br />

## Target discovery

Let's start by identifying our target's IP address.

```
fping -agq 10.10.10.0/24

10.10.10.1
10.10.10.2
10.10.10.6
```
<br />

## Running Services

Next, scan the target for open ports, running services, and version detection.

![](Pics/nmap.png)
<br />


## Web Enumeration

When we visit the web page, we got a default Page.

![](Pics/web.png)
<br />
<br />

Inspecting the source code, we found a comment.

![](Pics/web1.png)
<br />
<br />

Let's visit `/alohomora`.

![](Pics/web2.png)
<br />
<br />

Ok, `Draco` was in `Slytherin` house, keep that in mind and let's do directory bruteforcing.

![](Pics/gob.png)
<br />
<br />

We have a `/phpinfo.php` which contains information about the PHP environment.

![](Pics/web3.png)
<br />
<br />

Let's check `/log`.

![](Pics/web4.png)
<br />
<br />

We have a password, and a new directory `/DiagonAlley`, let's check it.

![](Pics/web5.png)
<br />
<br />

Ok, it's a blog, and the first post is in `"Brainfuck" (programming language)`. Let's decode it. [https://www.dcode.fr/brainfuck-language](https://www.dcode.fr/brainfuck-language)

![](Pics/decode.png)
<br />
<br />

It gaves us `donn`. Back to the blog, and nothing useful. Let's see if there are any directories under `/DiagonAlley`.

![](Pics/gob2.png)
<br />
<br />

Let's navigate to `/wp-admin ` and try to log in.

![](Pics/login.png)
<br />
<br />

Login with `Draco` and `slytherin` as the hint says.

![](Pics/login2.png)
<br />
<br />

Under `Plugins`, `Installed Plugins`, there's a plugin called `Hello Dolly`, we need to activate it.

![](Pics/plugin1.png)
<br />
<br />

Now, let's modify the plugin with a reverse shell code. Click on `Plugin Editor`, Select `Hello Dolly` to edit, paste your reverse shell and prepare your listener.

![](Pics/plugin2.png)
<br />
<br />

Change the Port and IP address to yours, then `
Update File`.

![](Pics/shell.png)
<br />
<br />

And we got a reverse shell.

![](Pics/shell2.png)
<br />

Now, let's see what users we have in `/etc/passwd`.

![](Pics/shell3.png)
<br />
<br />

We have `dobby`. Now, let's find a way to escalate our privileges.

![](Pics/shell4.png)
<br />
<br />

We have <strong>suid</strong> on `find` and `base32`, we could abuse find to get root. [https://gtfobins.github.io/gtfobins/find/](https://gtfobins.github.io/gtfobins/find/)

![](Pics/proof.png)
<br />
<br />

Back to the `www-data` user, there's `base32` with <strong>suid</strong>. We could abuse it to read a file like `shadow`. [https://gtfobins.github.io/gtfobins/base32/](https://gtfobins.github.io/gtfobins/base32/).

![](Pics/shell5.png)
<br />
<br />

Let's crack it with `john`.

![](Pics/hash.png)
<br />
<br />

Switch to `dobby`. 

![](Pics/shell6.png)
<br />
<br />

In `dobby`'s home directory, there's `flag1.txt`.

![](Pics/flag1.png)
<br />