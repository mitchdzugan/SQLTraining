# `JOIN`
Joins allow you to pull in data from another table. This is useful when a table you are looking at has a column containing an id that represents a row in a different table.
```SQL
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

## `INNER JOIN` Examples


**Get Adam's Top 10 Movie Reviews**
```SQL
SELECT User.name as userName, Movie.name as movieName, UserMovieReview.stars, UserMovieReview.review, UserMovieReview.reviewDate FROM User
INNER JOIN UserMovieReview on UserMovieReview.userId = User.id
INNER JOIN Movie on UserMovieReview.movieId = Movie.id
WHERE User.name = 'Adam'
ORDER BY UserMovieReview.stars DESC
LIMIT 10
```
This one uses something we didn't talk about which is that you can also use `as` to rename columns not just tables in joins. Without the `as userName`/`as movieName` then those two columns show up as `name` and `name (1)` in our output. Not a big deal but can be nice to make more clear.

**Get 10 Most Recent Movie Reviews From People Keef Follows**
```SQL
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
```SQL
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
