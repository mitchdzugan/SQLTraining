# FUNDEMENTALS

In SQL, data is stored in TABLEs that are made up of COLUMNs and ROWs. A row represents one entity within the table whereas a column is a specific property of that data. IE:

**User**

| id | name | email |
|---|--------|------|
| 1 | Mitch | mdzugan@smash.gg |
| 2 | Reb | reb@smash.gg|

We would call this the User TABLE. It has COLUMNs for id, name, and email. And it has 2 ROWs, one for Reb and one for Mitch.

## `SELECT`
`SELECT *` gets all columns and rows from a table
```SQL
SELECT * FROM Movie
SELECT * FROM User
SELECT * FROM UserFollowing
SELECT * FROM UserMovieReview
```
SQL keywords are not case sensitive
```SQL
SeLEcT * frOm mOViE
```	


You can list specific columns to control the amount/order of the columns in your result.
compare:
```SQL
SELECT * FROM Movie
```
with:
```SQL
SELECT name, starring, genre from Movie
```			

## Column Modifiers
* `count(...)`: counts the number of rows in a response.
* `sum(...)`/`avg(...)`/`min(...)`/`max(...)`: all work the same way as `count` but do different operations on the given column across all of the rows in the response.
* `distinct(...)` filters rows that contain duplicate values of the supplied column

```SQL
SELECT count(id) from Movie
```
```SQL
SELECT count(movieId) from UserMovieReview
```
```SQL
SELECT movieId from UserMovieReview
```
```SQL
SELECT distinct(movieId) from UserMovieReview
```
```SQL
SELECT count(distinct(movieId)) from UserMovieReview
```
Should be able to understand the output of each of those. Also we didnt really talk about this since we discussed `where` after but can combine these with `where` which is often useful way to end a query. IE "Get youngest/oldest player at a certain tourney", "Get the average smash.gg registration date of users who have run a tourney with more than 1000 entrants". All these would probably do a bunch of `join`s to gather data from a bunch of tables, have some `where`s to filter to the specific cases the question cares about, and then use `max`/`min`/`avg` to get the final answer.

## `WHERE`
`where` lets you filter the rows of a table based on certain conditions of a column.
```SQL
SELECT * FROM Movie
WHERE genre = 'Comic Book'
```
* `or` accepts either condition, broadens results
* `and` must satisfy both conditions, narrows results
```SQL
SELECT * FROM Movie
WHERE genre = 'Comic Book'
OR genre = 'Kids'
```
```SQL
SELECT * FROM Movie
WHERE genre = 'Comic Book'
AND starring = 'Robert Downey Jr.'
```
`not` negates a condition. You can use `(`parenthesis`)` to combine many conditions.
```SQL
SELECT * FROM Movie
WHERE year > 2000
AND (
	genre = 'Drama'
	OR (NOT starring = 'Bradley Cooper')
)
```
You can use `IN` to accept any of several possible values. IE:
```SQL
SELECT * FROM Movie WHERE genre = 'Drama' OR genre = 'Kids'
```
Is the same as:
```SQL
SELECT * FROM Movie WHERE genre in ('Drama', 'Kids')
```

## `ORDER BY`
`ORDER BY` orders the rows in a result according to the value in the given column.
```SQL
select * from Movie ORDER BY year
```
`ASC`/`DESC` control whether the order is ascending or descending (`ASC` is assumed by default)

```SQL
select * from Movie ORDER BY year DESC
```
```SQL
select * from Movie ORDER BY year ASC
```
You can list multiple columns in an `ORDER BY` and it will use the latter columns to break ties in the earlier columns. For example in the `Chinook_Sqlite` database:

```SQL
SELECT FirstName, LastName FROM Customer
ORDER BY FirstName, LastName
```
This will order the results alphabetically by FirstName but for people who have the same FirstName (IE "Luis Rojas" and "Luis Goncalves", they will be sorted between each other by LastName.

## `LIMIT`
`LIMIT` (or `TOP` in some other implementations of SQL) restricts the amount of rows in the result. **Typically used in combination with `ORDER BY`** to get the top/bottom N results of a question. IE:

**What are the 5 oldest movies in our database?**
```SQL
select * from Movie ORDER BY year ASC limit 5
```
