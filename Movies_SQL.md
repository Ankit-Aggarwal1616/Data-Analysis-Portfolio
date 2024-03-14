How many entries are there for movies, shows, actors, and directors in the database?

```sql
SELECT 
    COUNT(DISTINCT CASE WHEN role like 'ACTOR' THEN person_id END) AS actor_count,
    COUNT(DISTINCT CASE WHEN role like 'DIRECTOR' THEN person_id END) AS director_count,
    COUNT(DISTINCT CASE WHEN show_type like 'SHOW' THEN id END) AS show_count,
    COUNT(DISTINCT CASE WHEN show_type like 'MOVIE' THEN id END) AS movie_count
FROM 
	titles t 
LEFT JOIN 
	credits c ON c.title_id = t.id;
```

What is the average IMDb rating for both shows and movies in the dataset?
```sql
SELECT
	AVG(CASE WHEN show_type = 'SHOW' THEN imdb_score END) AS avg_imdb_rating_show,
    AVG(CASE WHEN show_type = 'MOVIE' THEN imdb_score END) AS avg_imdb_rating_movie
FROM 
	titles 
```
What are the top 5 highest-rated movies and and top 5 rated TV shows based on IMDb ratings?
```sql
SELECT
    title,
    imdb_score,
    show_type
FROM
    (SELECT
        title,
        imdb_score,
        'TV Show' AS show_type
    FROM
        titles
    WHERE
        show_type = 'SHOW'
    ORDER BY
        imdb_score DESC
    LIMIT 5) AS tv_shows
UNION ALL
SELECT
    title,
    imdb_score,
    show_type
FROM
    (SELECT
        title,
        imdb_score,
        'Movie' AS show_type
    FROM
        titles
    WHERE
        show_type = 'MOVIE'
    ORDER BY
        imdb_score DESC
    LIMIT 5) AS movies;
```

Among actors, who has the highest average IMDb rating for their roles in movies and shows?
```sql
SELECT
	name,
    role,
    production_countries,
    avg(imdb_score),
    COUNT(title_id),
    COUNT(DISTINCT Case when show_type like 'show' then id end) as numtvshows,
    COUNT(DISTINCT CASE WHEN show_type like 'movie' then id end) as nummovies
FROM
	titles t JOIN credits c
ON 
	t.id = c.title_id
WHERE
	role like 'actor'
GROUP BY
	name, role, production_countries
HAVING
	COUNT(DISTINCT title_id) >= 3
ORDER BY
	avg(imdb_score) DESC
LIMIT 10
```
Which directors have the highest average IMDb ratings for the movies and shows they've directed in the United States?
```sql
SELECT
	name,
    role,
    production_countries,
    avg(imdb_score),
    COUNT(title_id),
    COUNT(DISTINCT Case when show_type like 'show' then id end) as numtvshows,
    COUNT(DISTINCT CASE WHEN show_type like 'movie' then id end) as nummovies
FROM
	titles t JOIN credits c
ON 
	t.id = c.title_id
WHERE
	role like 'director' AND production_countries like '%US%'
GROUP BY
	name, role, production_countries
HAVING
	COUNT(DISTINCT title_id) >= 3
ORDER BY
	avg(imdb_score) DESC
LIMIT 10

```

For each year since 2010, which year has the highest average TMDB rating for movies and TV shows?
```sql
SELECT
	release_year,
    AVG(tmdb_score)
FROM titles
WHERE release_year >= 2010
GROUP BY release_year
ORDER BY avg(tmdb_score) DESC


Is there a correlation between different movie age certifications and IMDb ratings?

SELECT
	age_certification,
    AVG(imdb_score)
FROM titles
GROUP BY age_certification
ORDER BY avg(imdb_score) DESC
```

Which genres have the highest average TMDB popularity?
```sql
SELECT
	genres,
    avg(imdb_score),
    count(*)
FROM
titles
group by genres
HAVING count(*) > 10
order by avg(imdb_score) desc
LIMIT 5

SELECT
	genres,
    avg(imdb_score),
    count(*)
FROM
titles
WHERE genres like '%horror%'
group by genres
HAVING count(*) > 10
order by avg(imdb_score) desc
LIMIT 5
```
What are the top 3 highest-rated movies in the 5 most popular genres based on IMDb ratings?
```sql
with cte as 
	(SELECT
	genres,
    avg(imdb_score),
    count(*)
FROM
titles
WHERE genres != '[]'
group by genres
HAVING count(*) > 10
order by avg(imdb_score) desc
LIMIT 5)

SELECT
	title,
    cte.genres,
    production_countries,
    avg(titles.imdb_score)
FROM cte join titles on cte.genres = titles.genres
GROUP BY title, cte.genres, production_countries
ORDER BY avg(titles.imdb_score) desc
LIMIT 10
```

Which 5 pairs of actors have appeared together in the most movies, and what are those movies?
```sql
select 
	c1.name as name1,
    c2.name as name2,
    count(distinct c1.title_id),
    avg(imdb_score)
from credits c1
join credits c2 on c1.title_id = c2.title_id and c1.name < c2.name
join titles t on c1.title_id = t.id and c2.title_id = t.id
group by c1.name, c2.name
order by count(distinct c1.title_id) desc
limit 5

Do IMDb votes differ significantly between TV shows and movies?

select 
	round(avg(case when show_type like 'show' then imdb_votes end), 2) as showvotes,
    round(avg(case when show_type like 'movie' then imdb_votes end), 2) as movievotes
from
titles
```
Is there a clear relationship between the number of IMDb votes and IMDb ratings?
```sql
SELECT
	AVG(CASE WHEN imdb_votes < 50000 then imdb_score end) as lessfifty,
    AVG(CASE WHEN imdb_votes BETWEEN 50000 and 100000 then imdb_score end) as betweenfiftyhundred,
    AVG(CASE WHEN imdb_votes BETWEEN 100000 and 250000 then imdb_score end) as between100250,
    AVG(CASE WHEN imdb_votes > 250000 then imdb_score end) as over250
FROM
	titles
```
What is the average IMDb rating for TV shows based on their duration: under 20 minutes, between 20 and 40 minutes, and over 40 minutes?
```sql
SELECT
	AVG(CASE WHEN runtime < 25 then imdb_score end) as less25avg,
    COUNT(CASE WHEN runtime < 25 then imdb_score end) as less25count,
    AVG(CASE WHEN runtime between 25 and 40 then imdb_score end) as between2540avg,
    COUNT(CASE WHEN runtime between 25 and 40 then imdb_score end) as between2540count,
    AVG(CASE when runtime > 40 then imdb_score end) as over40avg,
    COUNT(CASE when runtime > 40 then imdb_score end) as over40count
FROM
	titles
WHERE 
	show_type like 'show'
```
Which countries produce movies and TV shows with the highest IMDb ratings?
```sql
SELECT
	production_countries,
    AVG(imdb_score),
    count(*)
FROM
	titles
WHERE production_countries != '[]'
GROUP BY production_countries
HAVING count(*) >= 50
ORDER BY avg(imdb_score) desc
LIMIT 20
```
Which actors have most frequently portrayed themselves in movies and TV shows?
```sql
SELECT
	name,
    title,
    character_name,
    imdb_score
FROM
	titles t join credits c on t.id = c.title_id
WHERE character_name like '%self%'
ORDER BY imdb_score desc
LIMIT 20

SELECT
	name,
    count(*),
    AVG(imdb_score)
FROM
	credits c join titles t on c.title_id = t.id
WHERE character_name like '%self%'
GROUP BY name
ORDER BY count(*) desc
LIMIT 20
```
Who are the most popular director and actor for the top 5 countries represented in the dataset?
```sql
(
    SELECT
        name,
        production_countries,
        role,
        AVG(imdb_score) AS avg_imdb_score
    FROM
        credits c
    JOIN
        titles t ON c.title_id = t.id
    WHERE
        production_countries = "['KR']" AND role LIKE 'director'
    GROUP BY
        name, production_countries, role
    HAVING
        COUNT(*) >= 5
    ORDER BY
        avg_imdb_score DESC
    LIMIT 1
)
UNION
(
    SELECT
        name,
        production_countries,
        role,
        AVG(imdb_score) AS avg_imdb_score
    FROM
        credits c
    JOIN
        titles t ON c.title_id = t.id
    WHERE
        production_countries = "['KR']" AND role LIKE 'actor'
    GROUP BY
        name, production_countries, role
    HAVING
        COUNT(*) >= 5
    ORDER BY
        avg_imdb_score DESC
    LIMIT 1
)
UNION
(
    SELECT
        name,
        production_countries,
        role,
        AVG(imdb_score) AS avg_imdb_score
    FROM
        credits c
    JOIN
        titles t ON c.title_id = t.id
    WHERE
        production_countries = "['US']" AND role LIKE 'director'
    GROUP BY
        name, production_countries, role
    HAVING
        COUNT(*) >= 5
    ORDER BY
        avg_imdb_score DESC
    LIMIT 1
)
UNION
(
    SELECT
        name,
        production_countries,
        role,
        AVG(imdb_score) AS avg_imdb_score
    FROM
        credits c
    JOIN
        titles t ON c.title_id = t.id
    WHERE
        production_countries = "['US']" AND role LIKE 'actor'
    GROUP BY
        name, production_countries, role
    HAVING
        COUNT(*) >= 5
    ORDER BY
        avg_imdb_score DESC
    LIMIT 1
);
```

Which actor has portrayed the same character the most, and what is the highest-rated movie where they portrayed that character?
```sql
with cte as (
SELECT
	name,
    character_name,
    count(character_name)
from credits
WHERE character_name != ''
GROUP BY name, character_name
order by count(character_name) desc
limit 1)

select 
cte.name,
cte.character_name,
title,
imdb_score
from credits c join titles t on c.title_id = t.id join cte on (c.name = cte.name and c.character_name = cte.character_name)
order by imdb_score desc
limit 1
```
