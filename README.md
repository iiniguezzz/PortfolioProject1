# Spotify Advanced SQL Project and Query Optimization P-6
Project Category: Advanced
[Click Here to get Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

![Spotify Logo](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_logo.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```
## Project Steps

### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

### 4. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Queries are categorized into **easy**, **medium**, and **advanced** levels to help progressively develop SQL proficiency.

#### Easy Queries
- Simple data retrieval, filtering, and basic aggregations.
  
#### Medium Queries
- More complex queries involving grouping, aggregation functions, and joins.
  
#### Advanced Queries
- Nested subqueries, window functions, CTEs, and performance optimization.

### 5. Query Optimization
In advanced stages, the focus shifts to improving query performance. Some optimization strategies include:
- **Indexing**: Adding indexes on frequently queried columns.
- **Query Execution Plan**: Using `EXPLAIN ANALYZE` to review and refine query performance.
  
---

## 15 Practice Questions

### Easy Level
1. Retrieve the names of all tracks that have more than 1 billion streams.
```sql
SELECT * FROM spotify
WHERE stream > 1000000000;
```

2. List all albums along with their respective artists.
```sql
SELECT 
DISTINCT album, artist 
FROM spotify
ORDER BY 1;
```

3. Get the total number of comments for tracks where `licensed = TRUE`.
```sql
SELECT SUM(comments) 
	as total_comments 
FROM spotify
WHERE licensed = 'True';
```

4. Find all tracks that belong to the album type `single`.
```sql
SELECT * FROM spotify
WHERE ablum_type = 'single';
```

5. Count the total number of tracks by each artist.
```sql
SELECT 
	artist, --- 1
	COUNT(*) as total_no_songs --- 2
FROM spotify
GROUP BY artist 
ORDER BY 2;
```

### Medium Level
1. Calculate the average danceability of tracks in each album.
```sql
SELECT 
	album , 
	avg(danceability) as avg_danceability
	FROM spotify
	GROUP BY 1
	ORDER BY 2 DESC;
```

2. Find the top 5 tracks with the highest energy values.
```sql
SELECT 
	track,
	MAX(energy)
FROM Spotify
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

3. List all tracks along with their views and likes where `official_video = TRUE`.
```sql
SELECT 
	track,
	SUM(VIEWS) as total_views,
	SUM(LIKES) as total_likes
FROM spotify 
WHERE official_video = 'true' 
GROUP BY 1 
ORDER BY 2 DESC
LIMIT 5;
```

4. For each album, calculate the total views of all associated tracks.
```sql
SELECT 
	album,
	track,
	SUM(views)
FROM spotify
GROUP By 1,2
ORDER BY 3 DESC;
```
5. Retrieve the track names that have been streamed on Spotify more than YouTube.
```sql
SELECT * FROM 
(SELECT 
	track,
	-- most_played_on,
	COALESCE(SUM(CASE WHEN most_played_on = 'Youtube' THEN stream END),0) as streamed_on_youtube,
	COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END),0) as streamed_on_spotify
FROM spotify
GROUP BY 1
) as t1
WHERE 
	streamed_on_spotify > streamed_on_youtube
	AND
	streamed_on_youtube <> 0;
```
### Advanced Level
1. Find the top 3 most-viewed tracks for each artist using window functions.
```sql
WITH ranking_artist
AS
(SELECT
	artist,
	track,
	SUM(views) as total_view, -- we need to use SUM() because we have to figure out the total number of views 
	DENSE_RANK() OVER(PARTITION BY artist ORDER BY SUM (views) DESC) as rank
FROM spotify
GROUP BY 1, 2 
ORDER BY 1, 3 DESC
)
SELECT * FROM ranking_artist
WHERE rank <= 3;

```

2. Write a query to find tracks where the liveness score is above the average.
```sql
 SELECT * FROM spotify;

 -- we will be using the liveness column with the AVG function 
 SELECT AVG(liveness) FROM spotify -- 0.19
 -- add a where clause to filter it so that it shows tracks whos liveness is > 0.19
 SELECT 
 	track,
	artist,
	liveness
FROM spotify
WHERE liveness > (SELECT AVG (liveness) FROM spotify)

```

3. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.**
```sql

SELECT * FROM spotify;

-- we will be using the following columns: album, energy
SELECT
	album,
	energy
FROM spotify

-- we need to find the highest and lowest energy values and group them by 1 (album),then save that data 
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energy,
FROM spotify
GROUP BY 1
)

-- now write a query that finds the difference between the highest and lowest energy and name it energy_diff

WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energy
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energy as energy_diff
FROM cte
ORDER BY 2 DESC;
```
   
4. Find tracks where the energy-to-liveness ratio is greater than 1.2.
```sql
SELECT * FROM spotify;

-- we will be using columns track, energy, artist and liveness 
SELECT 
	track,
	artist,
	energy,
	liveness,
-- now calculate the energy to liveness ratio by dividing energy by liveness and null liveness values if they are 0 to avoid errors. name this function appropriately 
	energy / NULLIF(liveness, 0) as energy_to_liveness_ratio
FROM spotify 
WHERE energy / NULLIF(liveness,0) > 1.2;
```
5. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.
```sql
SELECT * FROM spotify;

-- we will use columns track, artist, views and likes

SELECT 
	track, 
	artist,
	views,
	likes,
-- use window function SUM(likes) OVER (ORDER BY views DESC) to calculate the cumulative sum of likes. name it appropriately
	SUM(likes) OVER(ORDER BY views DESC) AS cumulative_likes
	FROM spotify
	ORDER BY views DESC;
```

Here’s an updated section for your **Spotify Advanced SQL Project and Query Optimization** README, focusing on the query optimization task you performed. You can include the specific screenshots and graphs as described.

---

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **7 ms**
        - Planning time (P.T.): **0.17 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_before_index.png)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.153 ms**
        - Planning time (P.T.): **0.152 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_explain_after_index.png)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%203.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%202.png)
      ![Performance Graph](https://github.com/najirh/najirh-Spotify-Data-Analysis-using-SQL/blob/main/spotify_graphical%20view%201.png)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Joins, Subqueries, Window Functions
- **Tools**: pgAdmin 4 (or any SQL editor), PostgreSQL (via Homebrew, Docker, or direct installation)

## How to Run the Project
1. Install PostgreSQL and pgAdmin (if not already installed).
2. Set up the database schema and tables using the provided normalization structure.
3. Insert the sample data into the respective tables.
4. Execute SQL queries to solve the listed problems.
5. Explore query optimization techniques for large datasets.

---

## Next Steps
- **Visualize the Data**: Use a data visualization tool like **Tableau** or **Power BI** to create dashboards based on the query results.
- **Expand Dataset**: Add more rows to the dataset for broader analysis and scalability testing.
- **Advanced Querying**: Dive deeper into query optimization and explore the performance of SQL queries on larger datasets.

---

## Contributing
If you would like to contribute to this project, feel free to fork the repository, submit pull requests, or raise issues.

---

## License
This project is licensed under the MIT License.
