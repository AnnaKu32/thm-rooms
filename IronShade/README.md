What is the Machine ID of the machine we are investigating?
```BASH
cat /etc/machine-id
```
<img width="396" height="48" alt="image" src="https://github.com/user-attachments/assets/0c73c686-4a2d-4394-ac70-934fc6da444f" />

<span style="line-height:0.5;">&nbsp;</span>

What backdoor user account was created on the server?
```BASH
cat /etc/passwd
```
<img width="592" height="57" alt="image-1" src="https://github.com/user-attachments/assets/ec2d9b71-dba8-443f-a7fd-a38387f003d6" />

<span style="line-height:0.5;">&nbsp;</span>

What is the cronjob that was set up by the attacker for persistence?
```BASH
sudo cat /var/spool/cron/crontabs/root
```
<img width="672" height="530" alt="image-3" src="https://github.com/user-attachments/assets/9d88c463-9c0a-46c3-bf95-6abe777fda3d" />

<span style="line-height:0.5;">&nbsp;</span>

Examine the running processes on the machine. Can you identify the suspicious-looking hidden process from the backdoor account? How many processes are found to be running from the backdoor account’s directory?
```BASH
ps aux | grep mircoservice
```
<img width="910" height="84" alt="image-4" src="https://github.com/user-attachments/assets/3e4f4ecb-1bd1-445b-af92-2b81acf84298" />

<span style="line-height:0.5;">&nbsp;</span>

What is the name of the hidden file in memory from the root directory?
```BASH
ls -ld .?* 
```
<img width="492" height="72" alt="image-5" src="https://github.com/user-attachments/assets/aa0cf4c9-20f5-47f3-9e90-4034b6689da0" />

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
<img width="1172" height="127" alt="image-6" src="https://github.com/user-attachments/assets/f218b6bc-664e-43a3-9ab5-4d59a363003c" />

<span style="line-height:0.5;">&nbsp;</span>

From which IP address were multiple SSH connections observed against the suspicious backdoor account? How many failed SSH login attempts were observed on the backdoor account?
```BASH
sudo zgrep -a -i "sshd.*mircoservice" /var/log/auth.log*
```
<img width="1147" height="49" alt="image-7" src="https://github.com/user-attachments/assets/29b826b2-1bc1-4702-b8bf-4ab27c413ce2" />

<span style="line-height:0.5;">&nbsp;</span>

Which malicious package was installed on the host?
```BASH
sudo grep "install " /var/log/dpkg.log
```
<img width="908" height="177" alt="image-8" src="https://github.com/user-attachments/assets/59a44203-8efa-43b3-9561-1079984bb47e" />

<span style="line-height:0.5;">&nbsp;</span>

What is the secret code found in the metadata of the suspicious package?
```BASH
sudo grep -A20 -B5 -i "Package: pscanner" /var/lib/dpkg/status
```
<img width="758" height="266" alt="image-9" src="https://github.com/user-attachments/assets/7c507fa6-bcdd-447d-81a3-44ab9aa30615" />
