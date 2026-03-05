# SQL Analysis: Books Adapted Into Movies

## Introduction
This project analyzes a curated dataset of books that have been adapted into movies, exploring ratings, popularity, and engagement metrics. Using SQL, queries are applied to extract actionable insights about:
- Highly rated and popular books
- Top authors by book count and reader engagement
- Overhyped and underrated books
- Popularity distributions

## Dataset

Books that have been adapted into films with reader ratings and popularity metrics.

Source: https://huggingface.co/datasets/reinashi/best_movie_adaptations

## Dataset Description

The dataset contains books that have been adapted into movies, along with ratings and engagement metrics from Goodreads and a movie adaptation list.

| Column Name   | Description |
|---------------|-------------|
| Name          | Title of the book |
| Author        | Author of the book |
| Avg Rating    | Average Goodreads rating (0-5) |
| Rating Count  | Number of ratings received on Goodreads |
| Score         | Goodreads list score (based on adaptation list ranking) |
| Vote Count    | Number of votes received on the adaptation list |

## Key Questions
- Which books are most loved by readers?
- Which authors dominate movie adaptations?
- Are highly rated books also the most popular?
- Are there books that are widely read but poorly rated?

## SQL Techniques Used
- Filtering
- Aggregations
- Grouping
- CASE statements
- Ranking logic
- Data quality checks

## Data Cleaning & Challenges

The raw CSV contained commas inside book titles, escaped with a backslash `\`, e.g.:
The Lord of the Rings (Middle Earth\, #2-4)

Standard CSV import tools split these titles incorrectly into multiple columns.  

**Solution:**

1. Replaced `\,` with a temporary placeholder `;` across all rows:
The Lord of the Rings (Middle Earth; #2-4)
2. Imported the CSV into SQL successfully.
3. Restored the original commas inside book titles:
```sql
UPDATE books
SET name = REPLACE(name, ';', ',');
```
4. Cast numeric columns (Avg Rating, Rating Count, Score, Vote Count) to proper numeric types.
5. Removed rows with missing or zero ratings to ensure meaningful analysis.

## Analytical Goals Section
- Identify books with highest ratings, popularity, and engagement.
- Find top authors by total ratings and average ratings.
- Discover overhyped books: widely read but low-rated.
- Highlight hidden gems: highly rated but less popular books.
- Explore author-level rankings and book percentile metrics using advanced SQL queries.

## Queries
```sql
SELECT * from books LIMIT 10; -- check samples of books
update books set name = REPLACE(name,';',','); -- update the values to previous version with commas.
SELECT name, author, avg_rating as "Average Rating" from books order by avg_rating limit 10; -- Books with the highest rating
SELECT name, author, rating_count as "Rating Count" FROM books order by rating_count LIMIT 10; -- Books with the highest popularity
SELECT author, count(author) as "Number of Books" from books group by author ORDER by COUNT(author) desc limit 10; -- Authors with the highest number of books
SELECT MIN(avg_rating), MAX(avg_rating), MIN(rating_count), MAX(rating_count) from books; -- Find range of avg_rating and rating_counts
SELECT name, author, avg_rating as "Average Rating", rating_count AS "Rating Count" from books WHERE avg_rating>4.5 AND rating_count > 100000 order by rating_count desc; --Books with the highest rating and popularitybooks
SELECT AVG(avg_rating) as "Average Rating of Books" from books; -- Average Rating across all books
SELECT author, count(*) as "Books Written", SUM(rating_count) as "Total Rating Count" from books GROUP by author ORDER BY SUM(rating_count) DESC limit 10; --Top authors by total reader engagement
SELECT COUNT(*) FROM books WHERE avg_rating=0 and rating_count=0; -- Identifying rows with empty values.

SELECT name, author, avg_rating, vote_count, (avg_rating*vote_count) AS "Engagement Score", RANK() OVER (ORDER by (avg_rating*vote_count) DESC) as "Engagement Rank" from books limit 10; -- Books by weighted engagement
SELECT name, author, avg_rating, rating_count, RANK() OVER (ORDER BY rating_count Desc, avg_rating asc) as hype_rank from books where avg_rating<4.0 and avg_rating!=0 LIMIT 10; -- Overhyped books with high popularity but low ratings
SELECT name, author, avg_rating, rating_count, RANK() OVER (ORDER BY avg_rating desc, rating_count ASC) as hype_rank from books where avg_rating > 4.5 and rating_count < 50000 and rating_count!=0 limit 10; -- Underrated books with high ratings but low popularity
SELECT CASE 
        WHEN (rating_count < 50000 and rating_count!=0) THEN 'Low Popularity'
        WHEN rating_count < 200000 THEN 'Medium Popularity'
        WHEN rating_count < 1000000 THEN 'High Popularity'
        ELSE 'Very High Popularity'
        END AS popularity_bin, AVG(avg_rating) AS avg_rating, COUNT(*) AS book_count FROM books GROUP BY popularity_bin ORDER BY book_count DESC; --Popularity Counts of Books
```

## Key Insights
1. Highest Rated Books

| Name                                                         | Author         | Avg Rating |
| ------------------------------------------------------------ | -------------- | ---------- |
| Harry Potter and the Deathly Hallows (Harry Potter, #7)      | J.K. Rowling   | 4.62       |
| Harry Potter and the Prisoner of Azkaban (Harry Potter, #3)  | J.K. Rowling   | 4.58       |
| Harry Potter and the Half-Blood Prince (Harry Potter, #6)    | J.K. Rowling   | 4.58       |
| Harry Potter and the Goblet of Fire (Harry Potter, #4)       | J.K. Rowling   | 4.57       |
| The Return of the King (The Lord of the Rings, #3)           | J.R.R. Tolkien | 4.57       |
| The Lord of the Rings (Middle Earth, #2-4)                   | J.R.R. Tolkien | 4.53       |
| Harry Potter and the Order of the Phoenix (Harry Potter, #5) | J.K. Rowling   | 4.50       |
| The Two Towers (The Lord of the Rings, #2)                   | J.R.R. Tolkien | 4.49       |
| Red Dragon and The Silence of the Lambs                      | Thomas Harris  | 4.48       |
| Harry Potter and the Sorcerer's Stone (Harry Potter, #1)     | J.K. Rowling   | 4.47       |

2. Most Popular Books

| Name                                                        | Author              | Rating Count |
| ----------------------------------------------------------- | ------------------- | ------------ |
| Harry Potter and the Sorcerer's Stone (Harry Potter, #1)    | J.K. Rowling        | 10,507,368   |
| The Hunger Games (The Hunger Games, #1)                     | Suzanne Collins     | 9,042,730    |
| Twilight                                                    | Stephenie Meyer     | 6,824,643    |
| To Kill a Mockingbird                                       | Harper Lee          | 6,382,892    |
| The Great Gatsby                                            | F. Scott Fitzgerald | 5,446,055    |
| Pride and Prejudice                                         | Jane Austen         | 4,411,252    |
| Harry Potter and the Prisoner of Azkaban (Harry Potter, #3) | J.K. Rowling        | 4,408,422    |
| The Hobbit (The Lord of the Rings, #0)                      | J.R.R. Tolkien      | 4,169,898    |
| Harry Potter and the Chamber of Secrets (Harry Potter, #2)  | J.K. Rowling        | 4,121,668    |
| Harry Potter and the Goblet of Fire (Harry Potter, #4)      | J.K. Rowling        | 3,867,270    |

3. Top Authors by Number of Adapted Books

| Author              | Number of Books |
| ------------------- | --------------- |
| Stephen King        | 13              |
| J.K. Rowling        | 7               |
| William Shakespeare | 5               |
| J.R.R. Tolkien      | 5               |
| Thomas Harris       | 4               |
| Stephenie Meyer     | 4               |
| Nicholas Sparks     | 4               |
| Jane Austen         | 4               |
| E.M. Forster        | 4               |
| Agatha Christie     | 4               |

4. Overhyped Books

| Name              | Author              | Avg Rating | Rating Count |
| ----------------- | ------------------- | ---------- | ------------ |
| Twilight          | Stephenie Meyer     | 3.66       | 6,824,643    |
| The Great Gatsby  | F. Scott Fitzgerald | 3.93       | 5,446,055    |
| Lord of the Flies | William Golding     | 3.70       | 2,999,384    |
| Romeo and Juliet  | William Shakespeare | 3.74       | 2,674,988    |
| Fahrenheit 451    | Ray Bradbury        | 3.96       | 2,540,936    |

5. Underrated Books

| Name              | Author              | Avg Rating | Rating Count |
| ----------------- | ------------------- | ---------- | ------------ |
| Twilight          | Stephenie Meyer     | 3.66       | 6,824,643    |
| The Great Gatsby  | F. Scott Fitzgerald | 3.93       | 5,446,055    |
| Lord of the Flies | William Golding     | 3.70       | 2,999,384    |
| Romeo and Juliet  | William Shakespeare | 3.74       | 2,674,988    |
| Fahrenheit 451    | Ray Bradbury        | 3.96       | 2,540,936    |

6. Engagement Score

| Name                                                     | Author            | Avg Rating | Vote Count | Engagement Score |
| -------------------------------------------------------- | ----------------- | ---------- | ---------- | ---------------- |
| The Lord of the Rings (Middle Earth, #2-4)               | J.R.R. Tolkien    | 4.53       | 89         | 403.17           |
| Harry Potter and the Sorcerer's Stone (Harry Potter, #1) | J.K. Rowling      | 4.47       | 77         | 344.19           |
| Gone with the Wind                                       | Margaret Mitchell | 4.31       | 50         | 215.50           |
| Harry Potter and the Deathly Hallows (Harry Potter, #7)  | J.K. Rowling      | 4.62       | 46         | 212.52           |
| The Godfather (The Godfather, #1)                        | Mario Puzo        | 4.39       | 48         | 210.72           |

7. Popularity Distribution

| Popularity Bin       | Avg Rating | Book Count |
| -------------------- | ---------- | ---------- |
| Medium Popularity    | 1.77       | 166        |
| Low Popularity       | 3.91       | 160        |
| High Popularity      | 4.10       | 63         |
| Very High Popularity | 4.16       | 45         |

## Skills Demonstrated
- Data cleaning & preprocessing
- SQL query writing: filtering, aggregation, grouping
- Window functions, ranking, conditional aggregation
- Analytical interpretation for decision-making
