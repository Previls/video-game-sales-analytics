# Video Game Industry Analytics (BigQuery SQL & Looker Studio)

An end-to-end data analytics project analyzing global sales of 11,000+ video games to identify regional market trends, publisher performance, and the impact of critic scores.

---

## Tech Stack
- **Database:** Google BigQuery (SQL)
- **Visualization:** Looker Studio
- **Dataset:** Video Game Sales with Ratings (Kaggle)

---

##  Interactive Dashboard
 [Link to Live Looker Studio Dashboard](https://datastudio.google.com/u/0/reporting/f0d35c4f-380a-41dd-b718-b938267ff2e7/page/vLKCG)

---

##  Tasks & SQL Queries

### Task 1: Regional Preferences (NA vs JP)
- **Logic:** Used CTEs and `ROW_NUMBER()` window functions to rank and compare top genres in North America vs Japan.

```sql
WITH genre_naa AS (
    SELECT 
        genre AS genre_na,
        SUM(NA_Sales) AS na_summ,
        ROW_NUMBER() OVER (ORDER BY SUM(NA_Sales) DESC) AS row_numbbb
    FROM `united-catfish-467316-u5`.portfolio_games.video_games
    GROUP BY genre
    ORDER BY na_summ DESC
    LIMIT 3
),
genre_jpp AS (
    SELECT 
        genre AS genre_jp,
        SUM(JP_Sales) AS jp_summ,
        ROW_NUMBER() OVER (ORDER BY SUM(JP_Sales) DESC) AS row_numbbb
    FROM `united-catfish-467316-u5`.portfolio_games.video_games
    GROUP BY genre
    ORDER BY jp_summ DESC
    LIMIT 3
)
SELECT
    a.genre_na,
    a.na_summ,
    p.genre_jp,
    p.jp_summ,
    a.row_numbbb AS rank_place
FROM genre_naa a
JOIN genre_jpp p 
ON a.row_numbbb = p.row_numbbb;

```

### Task 2: Impact of Critic Scores
- **Logic:**: Segmented games into performance tiers (Хіт, Середняк, Провал) using CASE WHEN and calculated average global sales.
```sql
SELECT
    AVG(Global_Sales) AS avg_sales,
    CASE 
        WHEN Critic_Score BETWEEN 90 AND 100 THEN 'Хіт'
        WHEN Critic_Score BETWEEN 70 AND 89 THEN 'Середняк'
        ELSE 'Провал'
    END AS characteristic_of_score
FROM `united-catfish-467316-u5`.portfolio_games.video_games
WHERE Critic_Score IS NOT NULL
GROUP BY characteristic_of_score
ORDER BY avg_sales DESC;
```

### Task 3: Top Publishers & Bestsellers
- **Logic:**: Applied PARTITION BY in a window function to extract each top publisher's #1 best-selling game.

```sql
WITH publ_ AS (
    SELECT 
        Publisher,
        SUM(Global_Sales) AS suma_gl_sales
    FROM `united-catfish-467316-u5`.portfolio_games.video_games
    GROUP BY Publisher
    ORDER BY suma_gl_sales DESC 
    LIMIT 10
),
rankk AS (
    SELECT 
        u.Name,
        u.publisher,
        ROW_NUMBER() OVER (PARTITION BY Publisher ORDER BY u.Global_Sales DESC) AS game_rank
    FROM `united-catfish-467316-u5`.portfolio_games.video_games u
)
SELECT 
    s.Name,
    u.publisher,
    u.suma_gl_sales
FROM publ_ u
JOIN rankk s 
ON u.publisher = s.publisher
WHERE game_rank = 1
ORDER BY u.suma_gl_sales DESC;
```

## Key Insights
* **NA vs. JP Market Divergence:**: Action ($879M) and Sports ($684M) lead North American sales, whereas Role-Playing (RPG) completely dominates Japan with $355M — more than double its second-place genre.

* **Critic Scores Impact:**: "Hit" rated games (>90 score) average $2.83M in global sales — 3.3x higher than mid-tier games ($0.86M) and nearly 9x higher than low-rated titles.

* **Publisher Market Concentration:**: Nintendo leads total revenue with $1.79B (top title: Wii Sports), followed by Electronic Arts ($1.12B, FIFA 16) and Activision ($731M, Call of Duty: MW3).




