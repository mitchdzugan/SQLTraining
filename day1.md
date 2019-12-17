# FUNDEMENTALS

In SQL, data is stored in TABLEs that are made up of COLUMNs and ROWs. A row represents one entity within the table whereas a column is a specific property of that data. IE:

**User**

| id | name | email |
|---|--------|------|
| 1 | Mitch | mdzugan@smash.gg |
| 2 | Reb | reb@smash.gg|

We would call this the User TABLE. It has COLUMNs for id, name, and email. And it has 2 ROWs, one for Reb and one for Mitch.

# SELECT
`SELECT *` gets all columns and rows from a table
```
SELECT * FROM Movie
SELECT * FROM User
SELECT * FROM UserFollowing
SELECT * FROM UserMovieReview
```
SQL keywords are not case sensitive
```
SeLEcT * frOm mOViE
```	


You can list specific columns to control the amount/order of the columns in your result.
compare:
```
SELECT * FROM Movie
```
with:
```
SELECT name, starring, genre from Movie
```			

# Column Modifiers
* `count(...)`: counts the number of rows in a response.
* `sum(...)`/`avg(...)`/`min(...)`/`max(...)`: all work the same way as `count` but do different operations on the given column across all of the rows in the response.
* `distinct(...)` filters rows that contain duplicate values of the supplied column

```
SELECT count(id) from Movie
```
```
SELECT count(movieId) from UserMovieReview
```
```
SELECT movieId from UserMovieReview
```
```
SELECT distinct(movieId) from UserMovieReview
```
```
SELECT count(distinct(movieId)) from UserMovieReview
```
Should be able to understand the output of each of those. Also we didnt really talk about this since we discussed `where` after but can combine these with `where` which is often useful way to end a query. IE "Get youngest/oldest player at a certain tourney", "Get the average smash.gg registration date of users who have run a tourney with more than 1000 entrants". All these would probably do a bunch of `join`s to gather data from a bunch of tables, have some `where`s to filter to the specific cases the question cares about, and then use `max`/`min`/`avg` to get the final answer.

# WHERE
`where` lets you filter the rows of a table based on certain conditions of a column.
```
SELECT * FROM Movie
WHERE genre = 'Comic Book'
```
* `or` accepts either condition, broadens results
* `and` must satisfy both conditions, narrows results
```
SELECT * FROM Movie
WHERE genre = 'Comic Book'
OR genre = 'Kids'
```
```
SELECT * FROM Movie
WHERE genre = 'Comic Book'
AND starring = 'Robert Downey Jr.'
```
`not` negates a condition. You can use `(`parenthesis`)` to combine many conditions.
```
SELECT * FROM Movie
WHERE year > 2000
AND (
	genre = 'Drama'
	OR (NOT starring = 'Bradley Cooper')
)
```
You can use `IN` to accept any of several possible values. IE:
```
SELECT * FROM Movie WHERE genre = 'Drama' OR genre = 'Kids'
```
Is the same as:
```
SELECT * FROM Movie WHERE genre in ('Drama', 'Kids')
```

# ORDER BY
`ORDER BY` orders the rows in a result according to the value in the given column.
```
select * from Movie ORDER BY year
```
`ASC`/`DESC` control whether the order is ascending or descending (`ASC` is assumed by default)

```
select * from Movie ORDER BY year DESC
```
```
select * from Movie ORDER BY year ASC
```
# LIMIT
`LIMIT` (or `TOP` in some other implementations of SQL) restricts the amount of rows in the result. **Typically used in combination with `ORDER BY`** to get the top/bottom N results of a question. IE:

**What are the 5 oldest movies in our database?**
```
select * from Movie ORDER BY year ASC limit 5
```

# JOIN
Joins allow you to pull in data from another table. This is useful when a table you are looking at has a column containing an id that represents a row in a different table.
```
SELECT * FROM UserMovieReview
INNER JOIN User ON User.id = UserMovieReview.userId
```
Couple things to remember:
a) We do not need to include anything about the information we want to use from the other table in our `join` statement. We provide the details on how the 2 tables are connected, and then all information from that table becomes free for us to use.
b) On either side of the `=` sign we must have
*  A column from the table we are `join`ing
*  A column from one of the other tables we have in our query

Helpful to think about what it means when those 2 columns contain the same value for a row in each table. IE in the above example? What does it mean to have a `UserMovieReview.userId` equal to a `User.id` well that means the review was written by that user. I think asking yourself this question will be useful for you because a couple times you tried to do: `INNER JOIN User ON User.name = UserMovieReview.userId` and so by asking what it means for `User.name` to be the same as a `UserMovieReview.userId` we see right away that's impossible because a number can't be equal to a name.

We talked mostly about `INNER JOIN`s. [Here's](https://stackoverflow.com/a/26760807) a good refernce for other joins but we can talk more about them another time.

# INNER JOIN Examples


**Get Adam's Top 10 Movie Reviews**
```
SELECT User.name as userName, Movie.name as movieName, UserMovieReview.stars, UserMovieReview.review, UserMovieReview.reviewDate FROM User
INNER JOIN UserMovieReview on UserMovieReview.userId = User.id
INNER JOIN Movie on UserMovieReview.movieId = Movie.id
WHERE User.name = 'Adam'
ORDER BY UserMovieReview.stars DESC
LIMIT 10
```
This one uses something we didn't talk about which is that you can also use `as` to rename columns not just tables in joins. Without the `as userName`/`as movieName` then those two columns show up as `name` and `name (1)` in our output. Not a big deal but can be nice to make more clear.

**Get 10 Most Recent Movie Reviews From People Keef Follows**
```
SELECT User.name as userName,
       UserFollowed.name as followedUser,
       Movie.name as movieName,
       UserMovieReview.stars,
       UserMovieReview.review,
       UserMovieReview.reviewDate
       FROM User
INNER JOIN UserFollowing on UserFollowing.userId = User.id
INNER JOIN UserMovieReview on UserMovieReview.userId = UserFollowing.followingUserId
INNER JOIN User as UserFollowed on UserMovieReview.userId = UserFollowed.id
INNER JOIN Movie on UserMovieReview.movieId = Movie.id
WHERE User.name = 'Keef'
ORDER BY UserMovieReview.reviewDate DESC
LIMIT 10
```

**Get 10 Most Recent Movie Reviews From People Following Keef**
```
SELECT User.name as userName,
       UserFollower.name as followerUser,
       Movie.name as movieName,
       UserMovieReview.stars,
       UserMovieReview.review,
       UserMovieReview.reviewDate
       FROM User
INNER JOIN UserFollowing on UserFollowing.followingUserId = User.id
INNER JOIN UserMovieReview on UserMovieReview.userId = UserFollowing.userId
INNER JOIN User as UserFollower on UserMovieReview.userId = UserFollower.id
INNER JOIN Movie on UserMovieReview.movieId = Movie.id
WHERE User.name = 'Keef'
ORDER BY UserMovieReview.reviewDate DESC
LIMIT 10
```
I think if you can really understand the differences between those 2 queries, and how their results are different, then you are basically SQL master lol. To understand the difference it results btw it probably helps to know that in the database, Keith follows everyone but me and Justin do not follow him.

test
