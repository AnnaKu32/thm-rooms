### To identify the type of hash used to encrypt a value, you can analyze it using the 'Analyse hash' option in [CyberChief](https://gchq.github.io/CyberChef/) 

![image](https://github.com/user-attachments/assets/9053fd1c-8d86-49c7-90c4-ed19d8a5aa60)

<span style="line-height:0.5;">&nbsp;</span>

### You can decrypt simple hashes using [Crackstation](https://crackstation.net/) 

![image](https://github.com/user-attachments/assets/e5a17847-a6b5-434e-9ee0-822949b8290d)

<span style="line-height:0.5;">&nbsp;</span>

MD5:
```Bash
48bb6e862e54f2a795ffc4e541caed4d
```
<pre>easy</pre>

<span style="line-height:0.5;">&nbsp;</span>

SHA1:
```Bash
48bb6e862e54f2a795ffc4e541caed4d
```
<pre>easy</pre>

<span style="line-height:0.5;">&nbsp;</span>

SHA256:
```Bash
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
```
<pre>letmein</pre>

<span style="line-height:0.5;">&nbsp;</span>

MD4:
```Bash
279412f945939ba78ce0758d3fd83daa
```
<pre>Eternity22</pre> 

<span style="line-height:0.5;">&nbsp;</span>

SHA256:
```Bash
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
```
<pre>paule</pre>

<span style="line-height:0.5;">&nbsp;</span>

NTLM:
```Bash
1DFECA0C002AE40B8619ECF94819CC1B
```
<pre>n63umy8lkf4i</pre>

<span style="line-height:0.5;">&nbsp;</span>

### For more complex hashes use [Hashcat](https://hashcat.net/wiki/doku.php?id=example_hashes)  

Bcrypt:  

![image](https://github.com/user-attachments/assets/689f64ae-1cbc-493c-908a-e6c69b82c297)
```Bash
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```
```Bash
hashcat -a 0 -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```
<pre>bleh</pre>  

<span style="line-height:0.5;">&nbsp;</span>

### For salted encrypted hashes, format them as `hash:salt` in hash.txt  

HMAC-SHA1:  

![image](https://github.com/user-attachments/assets/956ef83b-fd3c-4d66-bfb4-2ed00736518e)
```Bash
hashcat -a 0 -m 160 hash.txt /usr/share/wordlists/rockyou.txt
```
```Bash
e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme
```
![image](https://github.com/user-attachments/assets/f7faa470-e12e-4d8e-85d8-841654b84f6e)
<pre>481616481616</pre>

<span style="line-height:0.5;">&nbsp;</span>

SHA512crypt:  

![image](https://github.com/user-attachments/assets/524e7019-0077-484c-a949-e41582085132)
```Bash
hashcat -a 0 -m 1800 hash.txt /usr/share/wordlists/rockyou.txt
```
```Bash
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
```
<pre>waka99</pre>
