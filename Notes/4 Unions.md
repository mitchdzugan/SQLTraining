# `UNION`
Union's are pretty simple, the syntax is just
```SQL
_QUERY_1_
UNION
_QUERY_2_
```
This will run `_QUERY_1_` and then run  `_QUERY_2_` and finally combine both of their lists of results into one larger set of results. For example using the `Chinook_sqlite` database:
```SQL
SELECT FirstName, LastName FROM Customer
UNION
SELECT FirstName, LastName FROM Employee
```
Will get you the list of names for both Employees and Customers. The downside to unions is once they queries have been unioned we have no way of knowing which query a given row came from (ie we don't know whether a given person is an Employee or Customer). Because of this I would say `UNION` is usually the last part of a query, where you glue together multiple cases that satisfy the answer to a question that can not be gotten all together with one query.

One thing that I forgot to mention when we first talked about this is that neither `_QUERY_1_` nor `_QUERY_2_` can use an `ORDER BY`, instead you put the `ORDER BY` at the end and it acts on the `UNION`'ed results. IE we could do:
```SQL
SELECT FirstName, LastName FROM Customer
UNION
SELECT FirstName, LastName FROM Employee
ORDER BY LastName, FirstName
```
and it would order the entire output alphatbetically by last name regardless of whether the names are for Customers or Employees.