### Whats the version and year of the windows machine?
Run [Get-ComputerInfo](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-computerinfo?view=powershell-7.5) in Powershell to get information about windows machine.

![image](https://github.com/user-attachments/assets/a612208d-fb03-4dd8-bbb5-71c4bcb43517)

<pre>Windows Server 2016</pre>

<span style="line-height:0.5;">&nbsp;</span>

### Which user logged in last?
Go to Event Viewer -> Applications and Service Logs -> Microsoft -> Windows -> User Profile Service -> Operational. Look for Event ID = 2, which points to events about log on.  

![image](https://github.com/user-attachments/assets/b71b6d96-4e45-4d73-8834-047eeb7f8ef4)

<pre>Administrator</pre>

<span style="line-height:0.5;">&nbsp;</span>

### When did John log onto the system last?
Go to Event Viewer -> Windows Logs -> Security. In Filter Current Log, in Event ID field write 4624, which means successful logon.  

![image](https://github.com/user-attachments/assets/07489bf2-c51f-46fe-bd94-01f432db1e29)

![image](https://github.com/user-attachments/assets/395fbcc9-4b0f-4ead-8fbc-f1f1b6ab1e62)

<pre>03/02/2019 5:48:32 PM</pre>

<span style="line-height:0.5;">&nbsp;</span>

### What IP does the system connect to when it first starts?
Go to System Information -> Software Environment -> Startup Programs  

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

### 

<span style="line-height:0.5;">&nbsp;</span>

### 

<span style="line-height:0.5;">&nbsp;</span>
