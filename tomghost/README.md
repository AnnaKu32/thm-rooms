### Find the services exposed by the machine
```BASH
nmap -sV -p- 10.10.119.52
```
![image](https://github.com/user-attachments/assets/843c0030-20f7-4aa6-9408-5b3c25261a22)

TCP port 8009 is open and running the Apache JServ Protocol (AJP) 1.3, while port 8080 is serving an Apache Tomcat 9.0.30 instance. This version of Apache Tomcat (9.0.30), when combined with an exposed AJP port is vulnerable to the [Ghostcat vulnerability - CVE-2020-193](https://www.exploit-db.com/exploits/49039)

<span style="line-height:0.5;">&nbsp;</span>


