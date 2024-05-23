# Analyzing Unicorn Companies- A SQL Project
![Analyzing Unicorn Companies](Analyzing%20Unicorn%20Companies.png)

Did you know that the average return from investing in stocks is [10% per year](https://www.nerdwallet.com/article/investing/average-stock-market-return) (not accounting for inflation)? But who wants to be average?! 

You have been asked to support an investment firm by analyzing trends in high-growth companies. They are interested in understanding which industries are producing the highest valuations and the rate at which new high-value companies are emerging. Providing them with this information gives them a competitive insight as to industry trends and how they should structure their portfolio looking forward.

You have been given access to their `unicorns` database, which contains the following tables:

### dates
| Column       | Description                                  |
|------------- |--------------------------------------------- |
| `company_id`   | A unique ID for the company.                 |
| `date_joined` | The date that the company became a unicorn.  |
| `year_founded` | The year that the company was founded.       |

### funding
| Column           | Description                                  |
|----------------- |--------------------------------------------- |
| `company_id`       | A unique ID for the company.                 |
| `valuation`        | Company value in US dollars.                 |
| `funding`          | The amount of funding raised in US dollars.  |
| `select_investors` | A list of key investors in the company.      |

### industries
| Column       | Description                                  |
|------------- |--------------------------------------------- |
| `company_id`   | A unique ID for the company.                 |
| `industry`     | The industry that the company operates in.   |

### companies
| Column       | Description                                       |
|------------- |-------------------------------------------------- |
| `company_id`   | A unique ID for the company.                      |
| `company`      | The name of the company.                          |
| `city`         | The city where the company is headquartered.      |
| `country`      | The country where the company is headquartered.   |
| `continent`    | The continent where the company is headquartered. |

## The desired outcome: 
Generate a comprehensive analysis highlighting the top three industries with the highest number of unicorn companies and their average valuation in billions over the recent years (2022, 2021, and 2020). The results should be ordered by the most recent year and then by the highest number of unicorn companies.

Our query should return a table in the following format:
| industry  | year | num\_unicorns       | average\_valuation\_billions |
| --------- | ---- | ------------------- | ---------------------------- |
| industry1 | 2021 |        ---          |             ---              |
| industry2 | 2020 |        ---          |             ---              |
| industry3 | 2019 |        ---          |             ---              |
| industry1 | 2021 |        ---          |             ---              |
| industry2 | 2020 |        ---          |             ---              |
| industry3 | 2019 |        ---          |             ---              |
| industry1 | 2021 |        ---          |             ---              |
| industry2 | 2020 |        ---          |             ---              |
| industry3 | 2019 |        ---          |             ---              |

Where `industry1`, `industry2`, and `industry3` are the three top-performing industries.

## Approach:
<ol>
<li><b>CTE 1:</b> Identifying the top three performing industries by evaluating the number of new unicorn companies established over the recent years (2022, 2021, and 2020)
</li>
<li><b>CTE 2:</b> Calculating the number of unicorns and their average valuation, grouped by year and industry.
</li>
<li><b>Combining the two CTEs</b>, rounding off the average valuation (in billions) to two decimal places and finally grouping the result by industry, year and number of unicorns. 
</li>

### The SQL query:
~~~sql
WITH top_industries AS (
	SELECT 
    	i.industry, 
        COUNT(i.*)
    FROM industries AS i
    JOIN dates AS d
        ON i.company_id = d.company_id
    WHERE DATE_PART('year', d.date_joined) IN ('2020', '2021', '2022')
    GROUP BY industry
    ORDER BY count DESC
    LIMIT 3
),

yearly_ranks AS 
(
    SELECT 
    	COUNT(i.*) AS num_unicorns,
        i.industry,
        DATE_PART('year', d.date_joined) AS year,
        AVG(f.valuation) AS average_valuation
    FROM industries AS i
    JOIN dates AS d
        ON i.company_id = d.company_id
    JOIN funding AS f
        ON d.company_id = f.company_id
    GROUP BY industry, year
)

SELECT 
	industry,
    year,
    num_unicorns,
    ROUND(AVG(average_valuation / 1000000000), 2) AS average_valuation_billions
FROM yearly_ranks
WHERE year in ('2020', '2021', '2022')
	AND industry in (SELECT industry FROM top_industries)
GROUP BY industry, year, num_unicorns
ORDER BY year DESC, num_unicorns DESC
~~~

### The output table:
| industry                        | year | num_unicorns | average_valuation_billions |
|---------------------------------|------|--------------|----------------------------|
| Fintech                         | 2022 | 31           | 1.71                       |
| Internet software & services    | 2022 | 28           | 2.00                       |
| E-commerce & direct-to-consumer | 2022 | 7            | 1.57                       |
| Fintech                         | 2021 | 138          | 2.75                       |
| Internet software & services    | 2021 | 119          | 2.15                       |
| Internet software & services	  | 2020 | 20	          | 4.35                       | 
| E-commerce & direct-to-consumer	| 2020 | 16	          | 4                          |
| Fintech	                        | 2020 | 15	          | 4.33                       |

A graphical representation illustrating the number of unicorn companies across various industries in the years 2019, 2020, and 2021 will be as follows:
![unicorn distribution](unicron%distribution.png)

