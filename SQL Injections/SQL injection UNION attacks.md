When an application is vulnerable to SQL injection and results of the query are returned within application's responses, you can use the `UNION`keyword to get data from other tables in the data base. 
This is a SQL injection UNION attack.

- The `UNION` keyword enables you to execute one or more additional `SELECT` queries and append results to original query. Ex:-
```
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

This SQL query returns a single result set with two columns, containing values from columns `a` and `b` in `table1` and columns `c` and `d` in `table2`.

For a `UNION` query to work two key requirements must be met:
- The individual queries must return the same number of columns.
- The data types in each column must be compatible between the individual queries.

Fulfilling these requirements involves finding out:
- How many columns are being returned from the original query.
- Which columns returned from the original query are of suitable data type to hold results from injected query


__Determining the number of columns required__
###### Method 1(`ORDER BY`)
One method to determine number of columns is injecting a series of `ORDER BY` clauses and incrementing the column index till an error occurs. For example, if the injection point is a quoted string within the `WHERE` clause, you would submit
```
' ORDER BY 1-- 
' ORDER BY 2--  
' ORDER BY 3--
etc.
```
- Keep incrementing the column number to order by till you see an error such as
```
The ORDER BY position number 3 is out of range of the number of items in the select list.
```
- Or just a genuine error this tells you the number of columns in the result set
- The resulting SQL query by the above injections are
```
SELECT * FROM table WHERE value=''ORDER BY 1--' AND condiion
```




###### Method 2(`UNION SELECT`)
- The second method involves submitting a series of `UNION SELECT` payloads specifying a different number of null values.
- We keep incrementing the number of `NULL` as follows
```
' UNION SELECT NULL--
' UNION SELECT NULL, NULL--
' UNION SELECT NULL, NULL, NULL--
etc.
```
- If the number of nulls does not match the number of columns the database returns a error such as
```
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
```
- Or some other kind of generic error, in the worst case their is no change in the response.
- In Burp suite modify the vulnerable query as follows
```
'+UNION+SELECT+NULL,NULL,NULL--
```
- As with the `ORDER BY` technique, the application might actually return the database error in its HTTP response, but may return a generic error or simply return no results. When the number of nulls matches the number of columns, the database returns an additional row in the result set, containing null values in each column.
- The effect on the HTTP response depends on the application's code. If you are lucky, you will see some additional content within the response, such as an extra row on an HTML table. Otherwise, the null values might trigger a different error, such as a `NullPointerException`. In the worst case, the response might look the same as a response caused by an incorrect number of nulls. This would make this method ineffective.

###### __Data Base Specific Syntax__
- On Oracle databases every `SELECT` query must use the `FROM` keyword and specify a valid table. There is a built-in table on Oracle called `dual` which can be used for his purpose. So injected queries on Oracle would look like his
```
' UNION SELECT NULL FROM DUAL--
```
###### __Finding columns with a useful data type__
A SQL UNION injection enables you to retrieve results from an injected query. The interesting data is usually a string.

After determining the number of columns you can probe each column to test whether it can hold string data. You can submit a series of `UNION SELECT` payloads as follows
```
' UNION SELECT 'a',NULL,NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,NULL,'a',NULL--
```
- If the data type is not compatible with string data the injected query will cause a database error such as
```
Conversion failed when converting the varchar value 'a' to data type int.
```
 - If an error does not occur and the application's response contains some additional information, then the relevant column is suitable for string data
 - This helps us identify the column which stores string data.


__Using SQL injection UNION attack to retrieve interesting Information__
- After determining the number of columns in returned by the original query and found the columns which can hold string data, you are in position to retrieve string data.
- Suppose that 
	- The original query returns 2 columns both of which can hold string data
	- The injection point is a quoted string in the `WHERE` clause
	- The database contains a table called `users` with the columns `username` and `password`.
- In this example we can retrieve the contents of `users` table by submitting the input
```
' UNION SELECT username, password FROM users--
```
- To do this attack you need to know that there is a table called `users` with the columns `username` and `password`, without this we couldn't do this attack.
- It is possible to examine the database structure and find the tables and columns contained.

__Retrieving multiple values within a single column__
- In some cases the query in the previous example may only return a single column, we can retrieve multiple values together within this column by concatenating the values together
```
' UNION SELECT username || '~' || password FROM users--
```
This is oracle syntax for all look at [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- This uses the double-pipe `||` which is for string concatenation in Oracle.
- The injected query concatenates together the values of `username` and `password` separated by a `~` character. 
- The result from the query contains all the usernames and passwords
```
... 
administrator~s3cure 
wiener~peter 
carlos~montoya 
...
```
