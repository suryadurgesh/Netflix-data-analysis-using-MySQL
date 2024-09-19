# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/suryadurgesh/Netflix-data-analysis-using-MySQL/blob/046f20d935fcd63b6c74c018bac4035ecf5ce35c/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```
## Business Problems and Solutions
/*
1. Count the number of Movies vs TV Shows
*/

SELECT
	type,
    COUNT(*) AS total_content
FROM netflix
GROUP BY type;

/*
2. Find the most common rating for movies and TV shows
*/

SELECT
	type,
    rating
FROM
(
SELECT
	type,
	rating,
	COUNT(*),
    RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
FROM netflix
GROUP BY 1, 2
) as t1
WHERE 
	ranking = 1;
    
/*
3. List all movies released in a specific year (e.g., 2020)
*/

SELECT * FROM netflix
WHERE
	type = 'Movie'
    AND
    release_year = 2020;
    
/*
4. Find the top 5 countries with the most content on Netflix
*/

SELECT 
	country,
    COUNT(show_id) as total_content
FROM netflix
GROUP BY 1;

SELECT 
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(country, ',', n.n), ',', -1)) AS new_country, 
    COUNT(show_id) AS total_content
FROM 
    netflix
JOIN 
    (SELECT 1 n UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) n
ON n.n <= 1 + LENGTH(country) - LENGTH(REPLACE(country, ',', ''))
GROUP BY new_country
ORDER BY total_content DESC
LIMIT 5;

/*
5. Identify the longest movie
*/

SELECT * FROM netflix
WHERE
	type = 'Movie'
	AND
	duration = (SELECT MAX(duration) FROM netflix);
    
/*
6. Find content added in the last 5 years
*/

SELECT *
FROM netflix
WHERE STR_TO_DATE(date_added, '%M %d, %Y') >= DATE_SUB(CURDATE(), INTERVAL 5 YEAR);

/*
7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
*/

SELECT *
FROM netflix
WHERE FIND_IN_SET('Rajiv Chilaka', director) > 0;

/*
8. List all TV shows with more than 5 seasons
*/

SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) > 5;

/*
9. Count the number of content items in each genre
*/

SELECT 
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(listed_in, ',', n.n), ',', -1)) AS genre,
    COUNT(*) AS total_content
FROM 
    netflix
JOIN 
    (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) n(n)
ON n.n <= 1 + LENGTH(listed_in) - LENGTH(REPLACE(listed_in, ',', ''))
GROUP BY genre;

/*
10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!
*/

SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id) / (
            SELECT COUNT(show_id) 
            FROM netflix 
            WHERE country = 'India'
        ) * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;

/*
11. List all movies that are documentaries
*/

SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';

/*
12. Find all content without a director
*/

SELECT * 
FROM netflix
WHERE director is NULL ;

/*
13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
*/

SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > YEAR(CURDATE()) - 10;

/*
14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
*/

SELECT 
    TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(casts, ',', n.n), ',', -1)) AS actor,
    COUNT(*) AS total_appearances
FROM netflix
JOIN (SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5) n(n)
WHERE country = 'India'
GROUP BY actor
ORDER BY total_appearances DESC
LIMIT 10;

/*
15.
Categorize the content based on the presence of the keywords 'kill' and 'violence' in 
the description field. Label content containing these keywords as 'Bad' and all other 
content as 'Good'. Count how many items fall into each category.
*/

SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN LOWER(description) LIKE '%kill%' OR LOWER(description) LIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
