# Prioritise Report 

## Summary
The TODO application is vulnerable to a blind, error-based SQL-injection that sits in its order query parameter on the front page. By switching the ORDER BY target between a valid column index (1) and an invalid one (5) inside a CASE WHEN expression, an attacker can turn the HTTP status code itself into a boolean oracle. The application always returns a 302 redirect when the expression is false and a reproducible 500 error when it is true. 

Leveraging that one-bit channel and comparing hexadecimal byte values with hex(substr(...)), it is possible to extract data from the underlying SQLite database.

<span>&nbsp;</span>

| Risk | Impact | Severity |
|------|--------|----------|
| Blind SQL Injection in `order` parameter | Full read-access to the SQLite DB | High |

<span>&nbsp;</span>

## Scope
Demo `Prioritise` TODO web-app at http://$ip. Authentication not required.

<span>&nbsp;</span>

## Methodology
- Burp Repeater        
- manual SQL fiddling  
- python brute script  

<span>&nbsp;</span>

## Environment Overview
Flask with built-in SQLite db. Db contains table todos(id,title,date) and flag table.

<span>&nbsp;</span>

## Identified Vulnerabilities
### Blind error-based SQL Injection (GET /?order)
- **Vector:** /?order=(CASE WHEN (<condition>) THEN 5 ELSE 1 END), which causes HTTP 500 for ORDER BY 5 and HTTP 302 for ORDER BY 1, creating a TRUE/FALSE oracle.

- **Proof of Concept**:   
    ```
    curl -i "http://<IP>/?order=(CASE%20WHEN%20(1=1)%20THEN%205%20ELSE%201%20END)"
    curl -i "http://<IP>/?order=(CASE%20WHEN%20(1=0)%20THEN%205%20ELSE%201%20END)"
    ```

- **Impact:** Attacker can read arbitrary data, including the flag, by probing byte-by-byte with hex(substr(<query>,pos,1))='61' and watching the status code. 

<span>&nbsp;</span>


## Walktrough
Visiting `/` shows a plain TODO list

![image](https://github.com/user-attachments/assets/f4acaa1f-85f9-44a4-ba54-efbf72954727)

Try to add new item and catch request in Burp to see how request looks.

![image-1](https://github.com/user-attachments/assets/1cce8111-7cb1-4900-9502-724bb947e2e1)

Manually replacing the value with `5` makes the server respond with `HTTP 500`, while returning it to a valid value such as `1` restores the usual `302 redirect`.

![image-7](https://github.com/user-attachments/assets/4d7f83fd-9b66-4731-80f5-c60fa43d1563)

With the oracle in place we can interrogate the SQLite database one byte at a time. SQLite comparisons are normally case-insensitive, so instead of comparing characters directly we compare their hexadecimal codes. For one byte the payload looks like this:
```
/?order=(CASE WHEN (hex(substr((SELECT group_concat(name) FROM sqlite_master WHERE type='table' AND name NOT LIKE 'sqlite_%'),1,1))='66') THEN 5 ELSE 1 END)
```

Wrapping this in `CASE WHEN` gives a yes/no for `?order=(CASE WHEN <condition> THEN 5 ELSE 1 END)`. By asking is the next byte of hex equal to X and watching for 500 or 302 HTTP, a script brute-forces table names, column names, and finally the flag. 

![image-9](https://github.com/user-attachments/assets/3d6a738f-afe7-4aac-8140-0617b43b97ad)


