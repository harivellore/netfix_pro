netflix movies and tv show data analyss using sql

1[NETFLIX_LOGO](https://github.com/harivellore/netfix_pro/blob/main/7124274_netflix_logo_icon.png)

OBJECTIVE--NETFLIX PROJECT

CREATE TABLE NETFLIX
(
   show_id   VARCHAR(6),
   type      VARCHAR(10),
   title     VARCHAR(150),
   director  VARCHAR(208),
   casts      VARCHAR(1000),
   country   VARCHAR(150),
   date_added  VARCHAR(50),
   release_year  INT,
   rating    VARCHAR(10),
   duration  VARCHAR(15), 
   listed_in   VARCHAR(150),	
   description  VARCHAR(250)
);

select * from netflix;


-- 15 Business Problems 

--1. Count the Number of Movies vs TV Shows

SELECT 
    type,
    COUNT(*) as total_content
FROM netflix
GROUP BY type;

--2. Find the Most Common Rating for Movies and TV Shows

SELECT 
      type,
      rating
FROM 
(
    SELECT 
        type,
        rating,
        count(*),
        RANK() OVER (PARTITION BY type ORDER by count (*) DESC) AS ranking
    FROM netflix
	group by 1,2
) as t1
where
   ranking = 1

--3. List All Movies Released in a Specific Year (e.g., 2020)

SELECT * 
FROM netflix
WHERE
   type='Movie'
   release_year = 2020;

--4. Find the Top 5 Countries with the Most Content on Netflix

SELECT 
   UNNEST(STRING_TO_ARRAY(country, ',')) AS new_country,
   COUNT(show_id) as total_content
FROM netflix
group by 1
order by 2 desc
limit 5

--5. Identify the Longest Movie

select * from netflix
where 
   type= 'Movie'
   and
   duration = (select max (duration) from netflix)

--6. Find Content Added in the Last 5 Years

select 
    *
from netflix
where 
  to_date(date_added, 'month dd, yyyy')>=current_date-interval '5 years'

--7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

select * from netflix
where director like '%Rajiv Chilaka%'

--8. List All TV Shows with More Than 5 Season

select 
    *
from netflix
where 
   type = 'TV Show'
   AND
   SPLIT_PART(duration, ' ',1)::numeric > 5

--9. Count the Number of Content Items in Each Genre

select 
   unnest(STRING_TO_ARRAY(listed_in,','))as genre,
   count(show_id) as total_content
from netflix
group by 1

--10.Find each year and the average numbers of content release in India on netflix.

SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release 
LIMIT 5;

--11. List All Movies that are Documentaries

SELECT * FROM netflix
WHERE 
   listed_in ILIKE '%documentaries';

--12. Find All Content Without a Director

SELECT * 
FROM netflix
WHERE director IS NULL;

--13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

SELECT * FROM netflix
WHERE
  casts ILIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;

--14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*) as total_content
FROM netflix
WHERE country ILIKE '%India%'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

--15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
