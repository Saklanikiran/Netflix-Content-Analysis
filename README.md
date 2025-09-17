# Netflix Movies and TV Shows Data Analysis using SQL

<img width="2226" height="678" alt="Image" src="https://github.com/user-attachments/assets/0a5d1b99-ef4c-4a78-871a-be97b9b9dd38" />

## Overview
This project analyzes Netflix’s catalog of movies and TV shows to uncover key insights and trends. Using exploratory data analysis, it solves 15 business problems such as identifying content distribution, most common ratings, top countries, popular actors/directors, and genre-based patterns. It also categorizes content based on keywords like kill and violence to assess quality labeling. The goal is to provide data-driven insights into Netflix’s content strategy and audience preferences.


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

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY type;
```
<img width="155" height="70" alt="Image" src="https://github.com/user-attachments/assets/8d8ca278-fc78-410a-bd2f-a8645c471dae" />


**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```
<img width="223" height="74" alt="Image" src="https://github.com/user-attachments/assets/2736c648-8b4b-4a7e-9f47-327f6471a752" />

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT show_id, type, title, country, date_added, release_year, rating, duration, listed_in 
FROM netflix
WHERE release_year = 2020;
```

<img width="1065" height="194" alt="Image" src="https://github.com/user-attachments/assets/544017c9-68c9-4c4c-9ca4-b32f71f38e0a" />



**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT TRIM(JSON_UNQUOTE(js.country)) AS country, COUNT(*) AS total_content
FROM netflix
JOIN JSON_TABLE(
    CONCAT('["', REPLACE(country, ',', '","'), '"]'),
    "$[*]" COLUMNS (country JSON PATH "$")
) AS js
WHERE js.country IS NOT NULL AND TRIM(js.country) != ''
GROUP BY country
ORDER BY total_content DESC
LIMIT 5;
```

<img width="202" height="109" alt="Image" src="https://github.com/user-attachments/assets/2ad14143-3613-4b13-9ad2-8b0415531c53" />


**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
SELECT show_id, type, title, director, country, date_added, release_year, rating, listed_in
FROM netflix
WHERE type = 'Movie'
ORDER BY CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) DESC;

```
<img width="1155" height="186" alt="Image" src="https://github.com/user-attachments/assets/4fbe363a-e9fa-41bb-9472-6e5057c70dbc" />




**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) DESC;
```
<img width="1138" height="204" alt="Image" src="https://github.com/user-attachments/assets/f7b4efe3-1d86-45a5-8153-c83b33f8c69d" />


**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT *
FROM netflix
WHERE director LIKE '%Rajiv Chilaka%';
```
<img width="799" height="94" alt="Image" src="https://github.com/user-attachments/assets/1ea14159-5de5-4568-b54d-3a6d1b746b0a" />


**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT show_id, type, title, director, country, date_added, release_year, rating, listed_in
FROM netflix
WHERE type = 'TV Show'
  AND CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) > 5;
```
<img width="957" height="151" alt="Image" src="https://github.com/user-attachments/assets/14d0d7ed-59d4-4eb3-8d47-e5cd24cb6e82" />


**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT TRIM(genre) AS genre, COUNT(*) AS total_content
FROM netflix
JOIN JSON_TABLE(
    CONCAT('["', REPLACE(listed_in, ',', '","'), '"]'),
    "$[*]" COLUMNS (genre VARCHAR(255) PATH "$")
) AS g
GROUP BY genre;

```
<img width="1149" height="185" alt="Image" src="https://github.com/user-attachments/assets/06d08823-a043-4725-810d-4d8b70971152" />


**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in India on netflix. 
return top 5 year with highest avg content release!

```sql
SELECT release_year,
       COUNT(show_id) AS total_release,
       ROUND(COUNT(show_id) / (SELECT COUNT(show_id) FROM netflix WHERE country = 'India') * 100, 2) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY release_year
ORDER BY avg_release DESC
LIMIT 5;

```
<img width="275" height="132" alt="Image" src="https://github.com/user-attachments/assets/22799f5d-53bb-4f3f-9f8c-51389b62c456" />


**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
SELECT *
FROM netflix
WHERE listed_in LIKE '%Documentaries%';

```
<img width="1157" height="205" alt="Image" src="https://github.com/user-attachments/assets/67f09ce0-9285-48df-9be0-e8d575238a46" />


**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
SELECT *
FROM netflix
WHERE director IS NULL OR director = '';
```
<img width="1149" height="164" alt="Image" src="https://github.com/user-attachments/assets/f30d4280-7d34-4206-b011-076d33cb3c84" />


**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```sql
SELECT *
FROM netflix
WHERE cast LIKE '%Salman Khan%'
  AND release_year > YEAR(CURDATE()) - 10;
```
<img width="883" height="152" alt="Image" src="https://github.com/user-attachments/assets/02f3c359-51b8-475d-b098-b735760f2495" />


**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
SELECT TRIM(actor) AS actor, COUNT(*) AS total_movies
FROM netflix
JOIN JSON_TABLE(
    CONCAT('["', REPLACE(cast, ',', '","'), '"]'),
    "$[*]" COLUMNS (actor VARCHAR(255) PATH "$")
) AS c
WHERE country = 'India'
GROUP BY actor
ORDER BY total_movies DESC
LIMIT 10;

```
<img width="206" height="226" alt="Image" src="https://github.com/user-attachments/assets/c5bebfbe-2ba6-4378-96e3-d870b45fcc07" />


**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql
SELECT category, type, COUNT(*) AS content_count
FROM (
    SELECT *,
           CASE 
               WHEN LOWER(description) LIKE '%kill%' OR LOWER(description) LIKE '%violence%' THEN 'Bad'
               ELSE 'Good'
           END AS category
    FROM netflix
) AS categorized_content
GROUP BY category, type
ORDER BY type;


```
<img width="242" height="142" alt="Image" src="https://github.com/user-attachments/assets/b579a52f-c5ad-4f3d-af9e-7dcf0ba879bd" />


**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
