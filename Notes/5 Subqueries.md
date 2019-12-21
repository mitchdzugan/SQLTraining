# Subqueries
So far we have seen always used TABLEs as the source of new data for our `SELECT * FROM TABLE` as well as our `INNER JOIN ON TABLE` but we can also notice that the output of any one query can also be thought of as a TABLE itself because like tables they are just a list of rows that all have columns of the same type. Because of this we can actually use entirely different queries themselves in those positions instead of tables. In doing so, the inner query is called a `subquery`.

This is mostly useful in cases where we want to `SELECT FROM` or `JOIN ON` a subset of data that can not be described using simply `WHERE` (IE _usually_ needs a `GROUP BY` + `HAVING`).

For example consider the following 2 cases using the `Chinook_sqlite` database. First off, I would like to know the average song length for each album on albums that are not by AC/DC. That one is relatively straightforward and can be queried like so:
```SQL
SELECT Album.title, Artist.Name, AVG(Track.Milliseconds) FROM Track
INNER JOIN Album on Track.AlbumId = Album.AlbumId
INNER JOIN Artist on Album.ArtistId = Artist.ArtistId
WHERE Artist.Name != 'AC/DC'
GROUP BY Track.AlbumID
```
But now let's say that I would like to know the average song length for each album on albums made by an artists that have made more than 5 albums. This seems pretty similar to the last query because we are just filtering certain artists out of our results but what we'll see is that it is simply impossible to encode the condition "have made more than 5 albums" in a simple `WHERE` clause. This might jump out to you because in the `GROUP BY` section we showed that `WHERE` clauses **cannot** use aggregates and we would need to use the aggregate `count` to get that information about the artist.

We are still stuck though because in order to get the number of albums that an artist has made, we would need to group on Artist.name or Artist.ArtistId. This would not work because we are already grouping on Track.AlbumId and as we also saw in the `GROUP BY` section, adding additional column to our `GROUP BY` will expand the number of rows in our output and we already had the correct amount.

The real sticking point here is that we don't actually want to `JOIN` on the entire Artist table, we actually want to `JOIN` on the Artist table with all Artists that have less then 6 Albums removed. This query is straightforward enough:
```SQL
SELECT Artist.ArtistId, Artist.Name, count(Album.albumId) FROM Album
INNER JOIN Artist on Album.ArtistId = Artist.ArtistId
GROUP BY Album.ArtistId
HAVING count(Album.AlbumId) > 5
```
With this, we can simply swap out our `INNER JOIN` on the Artist table with an `INNER JOIN` to this query and we will get exactly what we want! So this is our final query:
```SQL
SELECT Album.title, ManyAlbumArtist.Name, AVG(Track.Milliseconds) FROM Track
INNER JOIN Album on Track.AlbumId = Album.AlbumId
INNER JOIN
	(
		SELECT Artist.ArtistId, Artist.Name, count(Album.albumId) FROM Album
		INNER JOIN Artist on Album.ArtistId = Artist.ArtistId
		GROUP BY Album.ArtistId
		HAVING count(Album.AlbumId) > 5
	) as ManyAlbumArtist on Album.ArtistId = ManyAlbumArtist.ArtistId
GROUP BY Track.AlbumID
ORDER BY ManyAlbumArtist.Name /* <- ORDER BY to show all have more than 5 albums */
```

Note the syntax here. Anywhere that you use `_TABLE_` you can replace `_TABLE_` with `( _QUERY_ ) as _SUB_QUERY_NAME_` and then can use `_SUB_QUERY_NAME_` to refer to columns from that subquery.



