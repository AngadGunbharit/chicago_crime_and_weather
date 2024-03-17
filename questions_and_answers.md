# Chicago Crime & Weather
## Questions and Answers

**Author**: Angad Gunbharit <br />
**Email**: gunbharitangad@gmail.com <br />

**LinkedIn**: https://www.linkedin.com/in/angad-gunbharit/ <br />

:exclamation: If you find this repository helpful, please consider giving it a :star:. Thanks! :exclamation:

Question 1: List the total number of reported crimes in the year 2022?

sql
Copy code
SELECT 
    EXTRACT(YEAR FROM reported_crime_date) AS crime_year,
    COUNT(*) AS num_of_crimes
FROM 
    chicago.crimes
WHERE 
    EXTRACT(YEAR FROM reported_crime_date) = 2022
GROUP BY 
    crime_year;
Results:

crime_year	num_of_crimes
2022	62412
Question 2: What is the trend of domestic violence crimes from 2018 to 2023?

sql
Copy code
WITH domestic_violence AS (
    SELECT 
        EXTRACT(YEAR FROM reported_crime_date) AS crime_year,
        COUNT(*) AS num_of_crimes
    FROM 
        chicago.crimes
    WHERE 
        domestic = TRUE
    GROUP BY 
        crime_year
)
SELECT 
    crime_year,
    num_of_crimes,
    LAG(num_of_crimes) OVER (ORDER BY crime_year) AS prev_year_count,
    ROUND(100 * (num_of_crimes - LAG(num_of_crimes) OVER (ORDER BY crime_year)) / LAG(num_of_crimes) OVER (ORDER BY crime_year), 2) AS year_over_year_growth
FROM 
    domestic_violence;
Results:

crime_year	num_of_crimes	prev_year_count	year_over_year_growth
2018	44099	NULL	NULL
2019	43344	44099	-1.71
2020	39983	43344	-7.75
2021	45018	39983	12.59
2022	42530	45018	-5.53
2023	46834	42530	10.12
Question 3: List the top 5 crime locations with the highest number of reported crimes in 2023.

sql
Copy code
SELECT 
    location_description,
    COUNT(*) AS num_of_crimes
FROM 
    chicago.crimes
WHERE 
    EXTRACT(YEAR FROM reported_crime_date) = 2023
GROUP BY 
    location_description
ORDER BY 
    num_of_crimes DESC
LIMIT 5;
Results:

location_description	num_of_crimes
Street	27681
Apartment	18542
Residence	16643
Sidewalk	6814
Alley	5538
4. List the top 5 months with the highest number of reported thefts and their corresponding average high temperatures. Order the list by the number of thefts in descending order.

sql
Copy code
SELECT
    to_char(t1.reported_crime_date, 'Month') AS month,
    COUNT(*) AS theft_count,
    ROUND(AVG(t2.temp_high), 1) AS avg_high_temp
FROM
    chicago.crimes AS t1
JOIN 
    chicago.weather AS t2
ON 
    t1.reported_crime_date = t2.weather_date
WHERE 
    t1.crime_type = 'theft'
GROUP BY
    month
ORDER BY
    theft_count DESC
LIMIT 5;
5. Find the community with the highest percentage increase in reported crimes from 2020 to 2021. Display the community name, the percentage increase, and the actual increase in reported crimes.

sql
Copy code
WITH crime_counts AS (
    SELECT
        t2.community_name,
        COUNT(*) AS crime_count,
        EXTRACT(YEAR FROM t1.reported_crime_date) AS year
    FROM
        chicago.crimes AS t1
    JOIN 
        chicago.community AS t2
    ON 
        t1.community_id = t2.community_id
    GROUP BY
        t2.community_name,
        year
),
community_changes AS (
    SELECT
        c1.community_name,
        (c2.crime_count - c1.crime_count) AS crime_increase,
        ROUND(((c2.crime_count - c1.crime_count) * 100.0 / c1.crime_count), 2) AS percentage_increase
    FROM
        crime_counts AS c1
    JOIN
        crime_counts AS c2
    ON
        c1.community_name = c2.community_name
    WHERE
        c1.year = 2020 AND c2.year = 2021
)
SELECT
    community_name,
    percentage_increase,
    crime_increase
FROM
    community_changes
ORDER BY
    percentage_increase DESC
LIMIT 1;
6. Calculate the average number of reported crimes per month for each year from 2018 to 2023. Include the year, month, and average number of reported crimes.

sql
Copy code
SELECT
    EXTRACT(YEAR FROM reported_crime_date) AS year,
    TO_CHAR(reported_crime_date, 'Month') AS month,
    ROUND(AVG(crime_count), 2) AS avg_crimes_per_month
FROM
    chicago.crimes
GROUP BY
    year, month
ORDER BY
    year, month;
7. Determine the percentage of crimes that resulted in an arrest for each crime type. Display the crime type along with the percentage of arrests, ordered by the highest percentage first.

sql
Copy code
SELECT
    crime_type,
    ROUND(COUNT(CASE WHEN arrest THEN 1 ELSE NULL END) * 100.0 / COUNT(*), 2) AS arrest_percentage
FROM
    chicago.crimes
GROUP BY
    crime_type
ORDER BY
    arrest_percentage DESC;




