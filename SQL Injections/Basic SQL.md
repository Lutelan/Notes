##### `SELECT` Statement
- The `SELECT` statement is used to select data from a database.
```
SELECT CustomerName, City FROM Customers
```
- Syntax
```
SELECT column1, column2, ....
FROM table_name;
```
- To return all columns use the `SELECT *` syntax
```
SELECT * FROM Customers;
```


##### `WHERE` Clause
- SQL `WHERE` Clause is used to filter records, and get records that fulfil a condition
```
SELECT * FROM Customers WHERE Counter='Mexico'
```
- Syntax
```
SELECT column1, column2, ....
FROM table_name
WHERE condition;
```



##### `AND` Operator
- The `WHERE` clause can contain one or many `AND` operators, the `AND` operator is used to filter records based on more than one condition.
```
SELECT * FROM Customers WHERE Country = 'Spain' AND ID=1
```
- Syntax
```
SELECT column1, column2, ...
FROM table_name
WHERE condition1 AND condition2 AND condition3....;
```


##### `OR` Operator
- The `WHERE` clause can contain one or more `OR` operators, it is used to filter records based on more than one condition
```
SELECT * FROM Customers WHERE Country = 'Germany' OR Country = 'Spain'
```
- Syntax
```
SELECT column1, column2, ...
FROM table_name
WHERE condition1 OR condition2, OR condition3....;
```


##### `UNION` Operator
- The `UNION` operator is used to combine the result-set of two or more `SELECT` statements.
	- Every `SELECT` statement within `UNION` must have the same number of columns.
	- The columns must also have similar data types
	- The columns in every `SELECT` statement must also be in same order
- Syntax
```
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
```
- `UNION` only gets distinct values to get duplicate values use `UNION ALL`.
```
SELECT column_name(s) FROM table1  
UNION ALL  
SELECT column_name(s) FROM table2;
```

##### `ORDER BY` Keyword
- The `ORDER BY` keyword is used to sort the result-set in ascending or descending order.
```
SELECT * FROM Products
ORDER BY Price;
```
- Syntax
```
SELECT column1, column2, ....
FROM table_name
ORDER BY column1, column2, ... ASC | DESC;
```

##### `SUBSTRING()` Function
- Used to extract characters from a string.
- Syntax
```
SUBSTRING(string, start, length)
```
- For example this extracts the first 3 characters starting from 1
```
SELECT SUBSTRING('SQL STUFF', 1, 3) AS ExtractString
```


##### `CASE` Expression
- The `CASE` expression goes through conditions and returns a value when the first condition is met, if no conditions are true `NULL` is returned, or the `ELSE` clause result is returned.
- Syntax
```
CASE
	WHEN condition1 THEN result1
	WHEN condition2 THEN result2
	WHEN condition3 THEN result3
	ELSE result
END;
```
