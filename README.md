# 📊 Netflix Movies and TV Shows Data Analysis using SQL

![Netflix Data Analysis](https://github.com/user-attachments/assets/172fea6a-930e-46b9-823b-eadfebc7d760)

## 📌 Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows dataset using **SQL**. The goal is to extract valuable insights, uncover trends, and answer key business questions to help inform decisions regarding content strategy and user engagement.

---

## 🎯 Objectives

- Analyze the distribution between movies and TV shows.
- Identify the most common content ratings.
- Explore content based on release year, country, and duration.
- Identify top-performing genres, directors, and actors.
- Categorize content based on descriptions using keyword analysis.

---

## 📂 Dataset

- 📌 **Source:** [Kaggle - Netflix Movies and TV Shows](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

---

## 🗃️ Schema Definition

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
🧠 Business Problems & SQL Solutions
1. Count of Movies vs TV Shows

```sql
SELECT type, COUNT(*) FROM netflix GROUP BY 1;
```

2. Most Common Rating per Content Type

```sql
WITH RatingCounts AS (
    SELECT type, rating, COUNT(*) AS rating_count
    FROM netflix GROUP BY type, rating
),
RankedRatings AS (
    SELECT *, RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT type, rating AS most_frequent_rating FROM RankedRatings WHERE rank = 1;
```

3. Movies Released in 2020

```sql
SELECT * FROM netflix WHERE release_year = 2020;
```

4. Top 5 Countries by Content

```sql
SELECT *
FROM (
    SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS country, COUNT(*) AS total_content
    FROM netflix GROUP BY 1
) t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

5. Longest Movie

```sql
SELECT * FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

6. Content Added in Last 5 Years

```sql
SELECT * FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

7. Content by Director 'Rajiv Chilaka'

```sql
SELECT * FROM (
    SELECT *, UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) t WHERE director_name = 'Rajiv Chilaka';
```

9. TV Shows with More Than 5 Seasons

```sql
SELECT * FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

10. Content Count by Genre

```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, COUNT(*) AS total_content
FROM netflix GROUP BY 1;
```

11. Top 5 Years with Highest Average Content Release in India

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(COUNT(show_id)::numeric /
          (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

12. Movies That Are Documentaries

```sql
SELECT * FROM netflix WHERE listed_in LIKE '%Documentaries';
```

13. Content Without a Director

```sql
SELECT * FROM netflix WHERE director IS NULL;
```

14. Movies with Salman Khan in Last 10 Years

```sql
SELECT * FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

15. Top 10 Indian Actors by Movie Appearances

```sql
SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor, COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

16. Categorize Content with 'Kill' or 'Violence' in Description

```sql
SELECT category, COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) categorized_content
GROUP BY category;
```

## 📈 Findings & Conclusion
Content Distribution: The dataset contains a wide mix of movies and TV shows, with a dominance of certain genres and types.

Regional Trends: India ranks among the top content-producing countries on Netflix.

Genre & Actor Analysis: Popular actors and genres emerge when filtering by region and rating.

Content Sensitivity: Text-based analysis helps to classify content as sensitive or neutral.

Useful Insights: This analysis can help media strategists, content planners, and Netflix itself to optimize their platform and improve user targeting.

---

## ✅ Tools Used
PostgreSQL / SQL

Excel (for preprocessing)

Kaggle Dataset
