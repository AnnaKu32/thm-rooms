This challenge begins with a .zip archive named commited. Upon extraction, the contents include:
- Readme.md
- main.py
- .git/ directory

![alt text](image.png)

<span style="line-height:0.5;">&nbsp;</span>

The project is described as a Python-based utility designed to help manage MySQL databases. The main.py includes logic to create a database, define a table, and populate it with a record.

Navigate to the `.git/logs/` directory and inspect the HEAD file to review the commit history: 
```BASH
cat .git/logs/HEAD
```

![alt text](image-2.png)

This shows all commits made, including branch checkouts and merges. The key here is to inspect each commit to identify if any sensitive data was ever committed and later removed.
```BASH
git show <commit_hash>
```

<span style="line-height:0.5;">&nbsp;</span>

In this commit, hardcoded credentials are visible inside main.py: 

![alt text](image-3.png)

<pre>flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}</pre>





