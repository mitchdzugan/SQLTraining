# `GROUP BY`
`GROUP BY` is useful when you have a multiple rows with the same value in a certain column but you would like to have one row for each of those values in the output. In doing so you can use `aggregate` functions to reduce all of the values that a column has in its grouping into a single value.

For example lets say we want a list of movies along side their average review score. We will start by doing an `INNER JOIN` to get data about both users and their reviews:

```SQL
SELECT * FROM Movie
INNER JOIN UserMovieReview on Movie.Id = UserMovieReview.movieId
ORDER BY Movie.id /* ORDER BY only used to help show groups */
```
| id | name | starring | genre | year | userId | movieid | stars | review | reviewDate |
|---|--------|------|------|------|------|------|------|------|------|
| 1 | Jurassic World | Chris Pratt | Sci-Fi | 2015 | 2 | 1 | 0 | trash | 2019-09-21 00:00:00.000
| 1 | Jurassic World | Chris Pratt | Sci-Fi | 2015 | 5 | 1 | 1 | bro chris pratt sux! | 2019-11-12 00:00:00.000
| 2 | Avengers: Endgame | Robert Downey Jr. | Comic Book | 2019 | 2 | 2 | 8 | fun | 2019-09-22 00:00:00.000
| 2 | Avengers: Endgame | Robert Downey Jr. | Comic Book | 2019 | 5 | 2 | 5 | i didnt see any others but this made no sense.. | 2019-11-13 00:00:00.000
3 | Finding Dory | Ellen Degeneres | Kids | 2016 | 1 | 3 | 0 | terrible | 2017-01-14 00:00:00.000
3 | Finding Dory | Ellen Degeneres | Kids | 2016 | 5 | 3 | 6 | i dont like fish.. | 2019-11-14 00:00:00.000
......

We can see at this point that we have all of the data we need. We have all of the movies and their reviews, but we have a row for every individual movie review and rating instead of a row for each movie and the average rating. This is where we can use `GROUP BY`.

```SQL
SELECT Movie.id, Movie.name, AVG(UserMovieReview.stars) FROM Movie
INNER JOIN UserMovieReview on Movie.Id = UserMovieReview.movieId
GROUP BY Movie.id
ORDER BY AVG(UserMovieReview.stars) DESC
```
This will give us the results we expect.
## Aggregates
Above we used the `AVG` aggregate to get an average of all of the star values in a group with the same movieId. We also have the following aggregates available to us:
* `count(...)`: gets the number of rows in the group
* `sum(...)`: gets the sum of all the values for column `...` in the group
* `avg(...)`: gets the average of all the values for column `...` in the group
* `min(...)`: gets the minimum of all values for column `...` in the group
* `max(...)`: gets the maximum of all values for column `...` in the group

Additionally instead of putting a column in any of the `...`s you can put `distinct(...)` where `...` is a column. Each aggregate function would work the same way but it would ignore duplicate values within a group. This is most useful for `count` when you want to get a count of the number of unique instances of something but potentially has applications for `average`/`sum` if you only wanted to do those calculations on the unique values though I can't think of a practical example and have never done so.

Also note that in our first lesson we saw you can use `aggregate` functions outside of `group by`s and in that case, instead of reducing groupings into a single value for each grouped row, they reduce the entire output into a single row value. ie:
```sql
SELECT count(*) FROM User
```
simply returns one row with the count of users in the database.

## `HAVING`
Once we have grouped and aggregated our data into useful values, we may want to do additional filtering of the output. For example, in our above average review score query, we may want to filter out all movies that haven't been reviewed by at least 3 users. We can use `HAVING` to do that like so:
```SQL
SELECT Movie.id, Movie.name, AVG(UserMovieReview.stars) FROM Movie
INNER JOIN UserMovieReview on Movie.Id = UserMovieReview.movieId
GROUP BY Movie.id
HAVING count(UserMovieReview.userId) > 2 /* <-- key addition */
ORDER BY AVG(UserMovieReview.stars) DESC
```
Note that we could not do this using `WHERE` because the `WHERE` filtering happens before the `GROUP BY` so we would be unable to check the condition since the rows have yet to be grouped. This means that **Aggregate functions can NEVER be used in a `WHERE` clause**. Similarly, because `HAVING` always operates on the result of a `GROUP BY`, this means that **Usages of `HAVING` must ALWAYS be preceded by a `GROUP BY`** 

That doesn't mean that `WHERE`s are not useful to us in `GROUP BY` queries though, they are used to filter out data that we do not want to use in our `GROUP BY` at all. For example, continuing further with the same query as above, lets say we want the same thing but we know that Adam is known to be a biassed reviewer and so we want the same results but with his reviews completely removed from the output. As in his reviews should neither count towards the average we are calculating, nor reaching the 2 threshold review count. We could do that like so:
```SQL
SELECT Movie.id, Movie.name, AVG(UserMovieReview.stars) FROM Movie
INNER JOIN UserMovieReview on Movie.Id = UserMovieReview.movieId
INNER JOIN User on User.Id = UserMovieReview.userId
WHERE User.name != 'Adam' /* <-- key addition */
GROUP BY Movie.id
HAVING count(UserMovieReview.userId) > 2
ORDER BY AVG(UserMovieReview.stars) DESC
```

SQL's syntax generally helps us remember this by the order of the syntax. The one tricky part is that even though the `_COLUMN_LIST_` in `SELECT _COLUMN_LIST_ FROM ...` is the first thing we write in our query, we are able to use aggregates there. Looking at the above query again for example:

```SQL
SELECT Movie.id, Movie.name, AVG(UserMovieReview.stars)   /* <- CAN use aggregates */
	FROM Movie
INNER JOIN UserMovieReview on
	Movie.Id = UserMovieReview.movieId                /* <- CANNOT use aggregates */
INNER JOIN User on User.Id = UserMovieReview.userId       /* <- CANNOT use aggregates */
WHERE User.name != 'Adam'                                 /* <- CANNOT use aggregates */
GROUP BY Movie.id
HAVING count(UserMovieReview.userId) > 2                  /* <- CAN use aggregates */
ORDER BY AVG(UserMovieReview.stars) DESC                  /* <- CAN use aggregates */
```
I sometimes think of the processing order as happening like below. (Don't think this is necessary to understand but might help you internalize when and where you can use aggregates)
```
FROM/JOINS => WHERE => GROUP BY => HAVING => ORDER BY/SELECT
               |                    |              |
               |                    |              - can use aggregates in ORDER BY/SELECT
               |                    - can use aggregates to filter results after grouping
               - can filter data before it is grouped.
               - cannot use aggregates though because
               - data has yet to be grouped
```
## `GROUP BY` multiple
So far we have only been grouping on only one column, just as we saw with `ORDER BY` though, we can group by multiple columns if we want. For example if we wanted to know the average score each user gives to each of the starring actors/actresses in our database we would do:
```SQL
SELECT User.Name, Movie.starring, AVG (UserMovieReview.stars) FROM User
INNER JOIN UserMovieReview on User.Id = UserMovieReview.UserId
INNER JOIN Movie on Movie.Id = UserMovieReview.MovieId
GROUP BY User.Id, Movie.starring
```

Here we are grouping on User.Id and Movie.starring, this will cause there to be one row in our output corresponding to all unique combinations of user and actor/actress.

Something to keep in mind is that adding additional `GROUP BY` columns to a query will always either add new rows or keep the same amount. (Adding Movie.starring to the `GROUP BY` in the above query could have had the same number of rows in the output as the same query with only `GROUP BY User.Id` if all users had only reviewed movies by 1 actor/actress).

**note**: the order of columns in a `GROUP BY` does not matter.

## TIP!
The words "each"/"by" in a question often signify the need for `GROUP BY`. For example rereading the above question, it was "we want to know the average score **each user** gives to **each of the starring actors/actresses** in our database" and our resulting query ended up including `GROUP BY User.Id, Movie.starring` which matches the bold nicely.
