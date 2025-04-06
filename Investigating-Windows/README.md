### Whats the version and year of the windows machine?
Run [Get-ComputerInfo](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-computerinfo?view=powershell-7.5) in Powershell to get information about windows machine.

![image](https://github.com/user-attachments/assets/a612208d-fb03-4dd8-bbb5-71c4bcb43517)

<pre>Windows Server 2016</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Which user logged in last?
Go to Event Viewer > Applications and Service Logs > Microsoft > Windows > User Profile Service > Operational. Look for Event ID = 2, which points to events about logon.  

![image](https://github.com/user-attachments/assets/b71b6d96-4e45-4d73-8834-047eeb7f8ef4)

<pre>Administrator</pre>

<span style="line-height:0.5;">&nbsp;</span>

### When did John log onto the system last?
Go to Event Viewer > Windows Logs > Security. In Filter Current Log, in Event ID field write 4624, which means successful logon.  

![image](https://github.com/user-attachments/assets/07489bf2-c51f-46fe-bd94-01f432db1e29)

![image](https://github.com/user-attachments/assets/395fbcc9-4b0f-4ead-8fbc-f1f1b6ab1e62)

<pre>03/02/2019 5:48:32 PM</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What IP does the system connect to when it first starts?
Go to System Information > Software Environment > Startup Programs  

![image](https://github.com/user-attachments/assets/e54eeb76-201a-42b4-9a43-df32e355e3fb)  

<pre>10.34.2.3</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What two accounts had administrative privileges (other than the Administrator user)?
Open CMD and run command:
```BASH
net localgroup administrators
```
![image](https://github.com/user-attachments/assets/d623a9e2-5d83-4e93-8c6f-608823d8e86e)

<pre>Guest,Jenny</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Whats the name of the scheduled task that is malicous.
Go to Task Schedular.  

![image](https://github.com/user-attachments/assets/56be50e2-3a3c-4b3a-a9d6-87dc473cb18b)  

<pre>Clean file system</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What file was the task trying to run daily?
![image](https://github.com/user-attachments/assets/3861617c-e8da-469c-b3b5-b958d00cf58f)  

<pre>nc.ps1</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What port did this file listen locally for?
<pre>1348</pre>

<span style="line-height:0.5;">&nbsp;</span>

### When did Jenny last logon?
![image](https://github.com/user-attachments/assets/f95c0c5b-e817-43f9-b603-ba9ef03aa279)
<pre>Never</pre>

<span style="line-height:0.5;">&nbsp;</span>

### At what date did the compromise take place?
![image](https://github.com/user-attachments/assets/29052240-b477-42b0-a7f4-8509eeb3f6b0)
<pre>03/02/2019</pre>

<span style="line-height:0.5;">&nbsp;</span>

### During the compromise, at what time did Windows first assign special privileges to a new logon?
![image](https://github.com/user-attachments/assets/cb11a9ec-5a8e-4572-91a9-6c6baa464e5a)
<pre>03/02/2019 4:04:49 PM</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What tool was used to get Windows passwords?
![image](https://github.com/user-attachments/assets/901e1232-3acd-4799-96f9-879823cd14b7)

`lsass.exe` is only created once at boot under normal conditions. If it appears multiple times or is spawned by another process it suggests tool like Mimikatz or Procdump attempted to access or dump LSASS, potentially by creating a new process targeting lsass.exe, or even duplicating it.
<pre>Mimikatz</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What was the attackers external control and command servers IP?
Go to Local Disk(C:) > Windows > System32 > drivers > etc  

![image](https://github.com/user-attachments/assets/238446bb-8d8f-42cb-a55c-22814687bc05)
![image](https://github.com/user-attachments/assets/97e7062e-d460-4d5c-882f-06f425e14fe2)  

The most important line is the one that maps google.com and www.google.com to the IP address `76.32.97.132`. This is not a reserved or private address, but a routable public IP.
<pre>76.32.97.132</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What was the extension name of the shell uploaded via the servers website?
Go to Local Disk(C:) > inetpub > wwwrootv  

![image](https://github.com/user-attachments/assets/595859fe-bdaf-405c-ad86-6416eb821459)

`C:\inetpub` is the default directory used by Microsoft’s Internet Information Services (IIS), which is the Windows web server. You can check for web hosted content, logs, or potentially scripts placed by an attacker.  

<pre>.jsp</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What was the last port the attacker opened?
The firewall rule "Allow outside connections for development..." is enabled, set to Allow, and permits connections on port 1337. Since this is not a standard system port, but a custom port opened for malicious use.  

![image](https://github.com/user-attachments/assets/c6512e6f-88de-4680-8fec-ccd13ac3860e)

<span style="line-height:0.5;">&nbsp;</span>

### Check for DNS poisoning, what site was targeted?
From the host file.
<pre>google.com</pre>pre>
