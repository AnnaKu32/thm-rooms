What is the Machine ID of the machine we are investigating?
```BASH
cat /etc/machine-id
```
![alt text](image.png)

<span style="line-height:0.5;">&nbsp;</span>

What backdoor user account was created on the server?
```BASH
cat /etc/passwd
```
![alt text](image-1.png)

<span style="line-height:0.5;">&nbsp;</span>

What is the cronjob that was set up by the attacker for persistence?
```BASH
sudo cat /var/spool/cron/crontabs/root
```
![alt text](image-3.png)
<span style="line-height:0.5;">&nbsp;</span>

Examine the running processes on the machine. Can you identify the suspicious-looking hidden process from the backdoor account? How many processes are found to be running from the backdoor account’s directory?
```BASH
ps aux | grep mircoservice
```
![alt text](image-4.png)
<span style="line-height:0.5;">&nbsp;</span>

What is the name of the hidden file in memory from the root directory?
```BASH
ls -ld .?* 
```
![alt text](image-5.png)
<span style="line-height:0.5;">&nbsp;</span>

What suspicious services were installed on the server? Format is service a, service b in alphabetical order.
```BASH
ls /sys/fs/cgroup/systemd/system.slice/
```
<span style="line-height:0.5;">&nbsp;</span>

Examine the logs; when was the backdoor account created on this infected system?
```BASH
sudo grep -a -i useradd /var/log/auth.log
```
![alt text](image-6.png)
<span style="line-height:0.5;">&nbsp;</span>

From which IP address were multiple SSH connections observed against the suspicious backdoor account? How many failed SSH login attempts were observed on the backdoor account?
```BASH
sudo zgrep -a -i "sshd.*mircoservice" /var/log/auth.log*
```
![alt text](image-7.png)
<span style="line-height:0.5;">&nbsp;</span>



Which malicious package was installed on the host?
```BASH
sudo grep "install " /var/log/dpkg.log
```
![alt text](image-8.png)
<span style="line-height:0.5;">&nbsp;</span>

What is the secret code found in the metadata of the suspicious package?
```BASH
sudo grep -A20 -B5 -i "Package: pscanner" /var/lib/dpkg/status
```
![alt text](image-9.png)