To identify the type of hash used to encrypt a value, you can analyze it using the 'Analyze hash' option in [CyberChief](https://gchq.github.io/CyberChef/), for example:  

![image](https://github.com/user-attachments/assets/9053fd1c-8d86-49c7-90c4-ed19d8a5aa60)

You can also decrypt simple hashes using [Crackstation](https://crackstation.net/), for example:  

![image](https://github.com/user-attachments/assets/e5a17847-a6b5-434e-9ee0-822949b8290d)

MD5:
```Bash
48bb6e862e54f2a795ffc4e541caed4d
```
<pre>
easy
</pre>

SHA1:
```Bash
48bb6e862e54f2a795ffc4e541caed4d
```
<pre>
easy
</pre>

SHA256:
```Bash
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
```
<pre>
letmein
</pre>

MD4:
```Bash
279412f945939ba78ce0758d3fd83daa
```
<pre>
Eternity22
</pre>

SHA256:
```Bash
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
```
<pre>paule</pre>

NTLM:
```Bash
1DFECA0C002AE40B8619ECF94819CC1B
```
<pre>n63umy8lkf4i</pre>

<br>
</br>

For more complex hashes use [Hashcat](https://hashcat.net/wiki/doku.php?id=example_hashes)
```Bash
hashcat -a 0 -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```
Bcrypt:
```Bash
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```
<pre>
bleh
</pre>  

<br>
</br>

For salted encrypted hashes format them as `<hash>:<salt>` in hash.txt
```Bash
 hashcat -a 0 -m 150 hash.txt /usr/share/wordlists/rockyou.txt
```
```Bash
e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme
```
<pre></pre>

<pre></pre>
<pre></pre>
