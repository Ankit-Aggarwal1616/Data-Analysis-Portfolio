# Netflix Movies and TV Shows SQL Analysis

#### In this project, I aim to analyze various statistics related to Netflix movies and TV shows available as of July 2022. The dataset comprises over 5000 titles from Netflix's US library.

1. How many entries are there for movies, shows, actors, and directors in the database?

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

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/bb6a8bb3-fc95-41dc-b694-e0263fdbfce2)

2. What is the average IMDb rating for both shows and movies in the dataset?
```sql
SELECT
    ROUND(AVG(CASE WHEN show_type = 'SHOW' THEN imdb_score END), 2) AS avg_imdb_rating_show,
    ROUND(AVG(CASE WHEN show_type = 'MOVIE' THEN imdb_score END), 2) AS avg_imdb_rating_movie
FROM 
    titles 
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/39eece05-187a-44f1-8494-21e42948ad06)

3. Is there a notable difference in the number of IMDb votes between TV shows and movies??
```
select 
    round(avg(case when show_type like 'show' then imdb_votes end), 2) as showvotes,
    round(avg(case when show_type like 'movie' then imdb_votes end), 2) as movievotes
from
    titles
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/81a08da6-a462-475c-a221-5c09db8843c1)

4. Is there a clear relationship between the number of IMDb votes and IMDb ratings?
```sql
SELECT
    ROUND(AVG(CASE WHEN imdb_votes < 50000 then imdb_score end), 2) as rating_under_50k_votes,
    ROUND(AVG(CASE WHEN imdb_votes BETWEEN 50000 and 100000 then imdb_score end), 2) as rating_between_50k_and_100k_votes,
    ROUND(AVG(CASE WHEN imdb_votes BETWEEN 100000 and 250000 then imdb_score end), 2)as rating_between_100k_and_250k_votes,
    ROUND(AVG(CASE WHEN imdb_votes > 250000 then imdb_score end), 2) as rating_over_250k_votes
FROM
    titles
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/1c18dc94-d58a-45a2-ab84-669b71d4aeba)

5. What is the average IMDb rating for TV shows based on their episode length: under 20 minutes, between 20 and 40 minutes, and over 40 minutes?
```sql
SELECT
    ROUND(AVG(CASE WHEN runtime < 25 THEN imdb_score END), 2) AS avg_imdb_score_under_25_minutes,
    COUNT(CASE WHEN runtime < 25 THEN imdb_score END) AS num_titles_under_25_minutes,
    ROUND(AVG(CASE WHEN runtime BETWEEN 25 AND 40 THEN imdb_score END), 2) AS avg_imdb_score_between_25_and_40_minutes,
    COUNT(CASE WHEN runtime BETWEEN 25 AND 40 THEN imdb_score END) AS num_titles_between_25_and_40_minutes,
    ROUND(AVG(CASE WHEN runtime > 40 THEN imdb_score END), 2) AS avg_imdb_score_above_40_minutes,
    COUNT(CASE WHEN runtime > 40 THEN imdb_score END) AS num_titles_above_40_minutes
FROM
    titles
WHERE 
    show_type LIKE 'show';

```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/8e8a2a60-b72f-4e62-b5f8-465bb0833fea)

6. Which countries, each with over 50 productions, have the highest IMDb ratings for their movies and TV shows?
```sql
SELECT
    production_countries,
    ROUND(AVG(imdb_score), 2) as imdb_score,
    count(*) as num_titles
FROM
	titles
WHERE
    production_countries != '[]'
GROUP BY
    production_countries
HAVING
    count(*) >= 50
ORDER BY
    avg(imdb_score) desc
LIMIT 10
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/068c540a-c754-4f8e-8658-e88f6384a30e)

7. What are the top 5 highest-rated movies and and top 5 highest-rated TV shows based on IMDb ratings?
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

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/079e6c69-f638-4ad6-9ec3-9ee45d7ac691)

8. Among actors who have appeared in at least three roles, who has the highest average IMDb rating for their roles in movies and shows, and in which countries were these productions made?
```sql
SELECT
    name,
    role,
    production_countries,
    ROUND(AVG(imdb_score), 2) as imdb_score,
    COUNT(title_id) as total_appearances,
    COUNT(DISTINCT CASE when show_type like 'show' then id end) as num_tvshows,
    COUNT(DISTINCT CASE WHEN show_type like 'movie' then id end) as num_movies
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

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/f24589ba-ced4-45ce-bac1-f164a06d5141)

9. Among American directors who have directed at least three productions, who has the highest average IMDb rating for their movies and shows?
```sql
SELECT
    name,
    role,
    production_countries,
    ROUND(AVG(imdb_score), 2) as imdb_score,
    COUNT(title_id) as total_directed,
    COUNT(DISTINCT CASE when show_type like 'show' then id end) as num_tvshows,
    COUNT(DISTINCT CASE WHEN show_type like 'movie' then id end) as num_movies
FROM
    titles t JOIN credits c
ON 
    t.id = c.title_id
WHERE
    role LIKE 'director' AND production_countries LIKE '%US%'
GROUP BY
    name, role, production_countries
HAVING
    COUNT(DISTINCT title_id) >= 3
ORDER BY
    avg(imdb_score) DESC
LIMIT 10

```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/eaa44611-1cd7-4b0e-8f2c-e1d98e92b83d)

10. For each year since 2014, which year had the highest average IMDb rating for both movies and TV shows?
```sql
SELECT
    release_year,
    ROUND(AVG(imdb_score), 2) as imdb_score
FROM
    titles
WHERE
    release_year >= 2014
GROUP BY
    release_year
ORDER BY
    avg(imdb_score) DESC
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/322ec81d-ca39-48d5-b1d3-a281c519955f)

11. Which age certification categories exhibit the highest IMDb scores on average?
```
SELECT
    age_certification,
    ROUND(AVG(imdb_score), 2) as imdb_score
FROM
    titles
GROUP BY
    age_certification
ORDER BY
    avg(imdb_score) DESC
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/7ff76564-a977-4091-9727-3f1644dcbc3b)

12. Which genres demonstrate the highest average IMDb score, considering only those genres with more than 10 titles?
```sql
SELECT
    genres,
    ROUND(AVG(imdb_score), 2) as imdb_score,
    COUNT(*) as num_titles
FROM
    titles
GROUP BY
    genres
HAVING
    COUNT(*) > 10
ORDER BY
    AVG(imdb_score) desc
LIMIT 5
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/fda8007e-783b-402d-811f-e69049b83e65)

13. Which specific subsets of the music genre exhibit the highest average IMDb ratings among titles with more than 10 entries?

```
SELECT
    genres,
    ROUND(AVG(imdb_score), 2) as imdb_score
FROM
    titles
WHERE
    genres LIKE '%music%'
GROUP BY
    genres
HAVING
    count(*) > 10
ORDER BY
    avg(imdb_score) desc
LIMIT 3
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/d56067d2-be69-4952-a8b6-435b58dc724f)

14. What are the top 10 titles, including both movies and TV shows, with the highest IMDb ratings within the five most popular genres?
```sql
with cte as (
SELECT
    genres,
    avg(imdb_score),
    count(*)
FROM
    titles
GROUP BY
    genres
HAVING
    count(*) > 10
ORDER BY
    avg(imdb_score) desc
LIMIT 5
)

SELECT
    title,
    show_type,
    cte.genres,
    production_countries,
    titles.imdb_score
FROM
    cte JOIN titles ON cte.genres = titles.genres
ORDER BY
    titles.imdb_score desc
LIMIT 10
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/c1ac7cd6-5ec6-4f36-9f1a-b383b52b0237)

15. Which 5 pairs of actors have appeared together in the highest number of movies? How many movies have they appeared in together, and what is their average IMDb rating across those movies?
```sql
SELECT 
    c1.name as actor1,
    c2.name as actor2,
    count(distinct c1.title_id) num_titles_together,
    ROUND(AVG(imdb_score), 2) as average_imdb_score
FROM
    credits c1
JOIN
    credits c2 ON c1.title_id = c2.title_id AND c1.name < c2.name
JOIN
    titles t ON c1.title_id = t.id AND c2.title_id = t.id
WHERE
    c1.role like 'actor' AND c2.role LIKE 'actor'
GROUP BY
    c1.name, c2.name
ORDER BY
    count(distinct c1.title_id) desc
LIMIT 5
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/fbdc0768-1a2a-40f4-8d73-96f91f27af1d)

16. Which actors have most frequently portrayed themselves in movies and TV shows, and what is the average IMDb rating of the titles in which they appeared as themselves?
```sql
SELECT
    name,
    count(*) as num_titles,
    ROUND(AVG(imdb_score), 2) as average_imdb_score
FROM
    credits c JOIN titles t ON c.title_id = t.id
WHERE
    character_name LIKE '%self%'
GROUP BY
    name
ORDER BY
    count(*) desc
LIMIT 10
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/094d5072-25d2-4811-9245-509f26d6a9e7)

17. Which actors have portrayed themselves most successfully based on IMDb ratings? List the top 10 movies in which an actor has portrayed themselves, sorted by IMDb rating.
```sql
SELECT
    name,
    title,
    character_name,
    imdb_score
FROM
    titles t JOIN credits c ON t.id = c.title_id
WHERE
    character_name LIKE '%self%'
ORDER BY
    imdb_score desc
LIMIT 10
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/6dd169a2-191c-4363-96dc-7eeb311cd06c)

18. Which actor has portrayed the same character the most, and what is the highest-rated movie where they portrayed that character?
```sql
with cte as (
SELECT
    name,
    character_name,
    count(character_name)
FROM
    credits
GROUP BY
    name, character_name
ORDER BY
    count(character_name) desc
LIMIT 1)

SELECT 
    cte.name,
    cte.character_name,
    title,
    imdb_score
FROM
    credits c join titles t on c.title_id = t.id join cte on (c.name = cte.name and c.character_name = cte.character_name)
ORDER BY
    imdb_score desc
LIMIT 1
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/5eb51373-a1ea-4357-8024-4b2b6ba661ea)

19. Netflix is currently considering the production of an original TV show and is evaluating various factors including the location and the key personnel involved. The company is contemplating both the US and Korea as potential production sites. In terms of casting and directing, they are interested in knowing the top TV show director and the top 5 TV show actors from each country based on IMDb rating, with a requirement of having been involved in at least 2 prior Netflix roles directed or acted in.
```sql
(
    SELECT
        name,
        production_countries,
        role,
        ROUND(AVG(imdb_score), 2) AS avg_imdb_score
    FROM
        credits c
    JOIN
        titles t ON c.title_id = t.id
    WHERE
        production_countries = "['KR']" AND role LIKE 'director'
    GROUP BY
        name, production_countries, role
    HAVING
        COUNT(CASE WHEN show_type like 'show' THEN 1 end) >= 2
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
        ROUND(AVG(imdb_score), 2) AS avg_imdb_score
    FROM
        credits c
    JOIN
        titles t ON c.title_id = t.id
    WHERE
        production_countries = "['KR']" AND role LIKE 'actor'
    GROUP BY
        name, production_countries, role
    HAVING
        COUNT(CASE WHEN show_type like 'show' THEN 1 end) >= 2
    ORDER BY
        avg_imdb_score DESC
    LIMIT 5
)
UNION
(
    SELECT
        name,
        production_countries,
        role,
        ROUND(AVG(imdb_score), 2) AS avg_imdb_score
    FROM
        credits c
    JOIN
        titles t ON c.title_id = t.id
    WHERE
        production_countries = "['US']" AND role LIKE 'director'
    GROUP BY
        name, production_countries, role
    HAVING
        COUNT(CASE WHEN show_type like 'show' THEN 1 end) >= 2
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
        ROUND(AVG(imdb_score), 2) AS avg_imdb_score
    FROM
        credits c
    JOIN
        titles t ON c.title_id = t.id
    WHERE
        production_countries = "['US']" AND role LIKE 'actor'
    GROUP BY
        name, production_countries, role
    HAVING
        COUNT(CASE WHEN show_type like 'show' THEN 1 end) >= 2
    ORDER BY
        avg_imdb_score DESC
    LIMIT 5
);
```

Output:  
![image](https://github.com/Ankit-Aggarwal1616/Data-Portfolio/assets/161187358/aeb6c8d5-6674-47d6-982c-a68d88ccee91)


