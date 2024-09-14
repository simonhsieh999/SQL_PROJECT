# Introduction
This project focuses on using SQL to analyze information about the data job market such as the different data job positions, the job salaries, the skills required, and the amount of remote and in-person data jobs available. The goal is to understand the data job market better and to improve my chance of landing my most ideal data job.

# Background 

This project is from Luke Barousse's course "[SQL For Data Analytics](https://www.youtube.com/watch?v=7mz73uXD9DA)", which I have enrolled in to develop my SQL skills. The data involved includes information about the different data jobs such as the job title, salary, skills required, location, and remoteness.

In this analysis, my goal is to identify through my SQL queries:

1) What are the top paying data analyst jobs?
2) What skills are required for those top paying data analyst jobs?
3) What skills are most in demand for data analysts?
4) What skills are associated with higher salaries for all data jobs?
5) What are the most optimal skills to learn for a data analyst looking to maximize job market value?

# The Analysis

There will be 4 tables involved in this project:

* job_postings_fact: Has information about different job postings such as the job title, salary, location, and remoteness.
* company_dim: Has the different company names
* skills_dim: Has the different skills and its type
* skills_job_dim: Has the different skills associated with each job

### 1) What are the top 10 paying data analyst jobs?

In order to identify the top 10 paying data analyst jobs, I did the following:

* Joined the job_postings_fact & company_dim table to get the company name associated with the job posting
* Filtered the data by data analyst job positions and positions with a mentioned salary
* Sorted the data from highest to lowest top salaries, and included only the top 10 salaries

```
SELECT
	job_id,
	job_title,
	job_location,
	job_schedule_type,
	salary_year_avg,
	job_posted_date,
	name AS company_name
FROM
	job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
	job_title_short = 'Data Analyst' AND
	job_location = 'Anywhere' AND
	salary_year_avg IS NOT NULL
ORDER BY
	salary_year_avg DESC 
LIMIT 10
```

Here was what was found as a result of the query:

* **High Salary Potential:** The top 10 salaries range from $184,000 to $650,000, which indicate high salary potential in data analytics.
* **Variety of Job Titles:** There are a variety of job titles within the top 10 salaries such as Director of Analytics, Principal Data Analyst, and Associate Director. There are also remote and hybrid options.
* **Diverse Employers:** There are diverse employers that pay the top 10 salaries such as Mantys, Meta, Smartasset, AT&T, and Pinterest Job Advertisements.

### 2) What skills are required for those top 10 paying data analyst jobs?

To identify the skills required for those top 10 paying data analyst jobs, I joined the table I created previously into the skills_dim and skills_job_dim tables. 

``` 
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
        LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst'
				AND salary_year_avg IS NOT NULL
        AND job_location = 'Anywhere'
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT
    top_paying_jobs.job_id,
    job_title,
    salary_year_avg,
    skills
FROM
    top_paying_jobs
	INNER JOIN 
        skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
	INNER JOIN 
        skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC
```

Here were the top 3 most demanded skills were as a result, and how many times the skills showed up in the top 10 paying data analyst jobs:

* **SQL:** 8 times
* **Python:** 7 times
* **Tableau:** 6 times

### 3) What skills are most in demand for data analysts?

For this query, I used the COUNT function to identify the number of times the skills showed up in each data analyst job posting, and sorted it from highest to lowest.

```
SELECT
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM
    job_postings_fact
    INNER JOIN
        skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN
        skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_postings_fact.job_title_short = 'Data Analyst'
GROUP BY
    skills_dim.skills
ORDER BY
    demand_count DESC
LIMIT 5
```
Here were the top 5 results:

skills   | demand_count
---------|-------------
sql      | 92628
excel    | 67031
python   | 57326
tableau  | 46554
power bi | 39468

* **SQL** and **Excel** shows up as the top 2 results, which shows the importance of spreadsheets and data processing used in data analysis.
* **Python**, **Tableau**, and **Power Bi** shows up as the next 3 results, which shows the importance of programming and visualizations in data storytelling and decision making.

### 4) What skills are associated with higher salaries for all data jobs?

In this query, I have identified the salaries associated with each skill, and sorted the data by the highest to lowest salaries of the top 5 skills.

```
SELECT
    skills_dim.skills AS skill, 
    ROUND(AVG(job_postings_fact.salary_year_avg),2) AS avg_salary
FROM
    job_postings_fact
	INNER JOIN
	    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	INNER JOIN
	    skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_postings_fact.job_title_short = 'Data Analyst' AND
    job_postings_fact.salary_year_avg IS NOT NULL
GROUP BY
    skills_dim.skills 
ORDER BY
    avg_salary DESC
LIMIT 5
```
Here were the results:

skills   | avg_salary
---------|-------------
svn      | 400000
solidity | 179000
couchbase| 160515
datarobot| 155485
golang   | 155000


### 5) What are the most optimal skills to learn for a data analyst looking to maximize job market value?

In this query, I have identified the top 10 skills that are both in high demand and have a high salary. I have sorted it by the highest to lowest demand count, followed by the highest to lowest average salary.

```
WITH skills_demand AS (
  SELECT
    skills_dim.skill_id,
		skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count
  FROM
    job_postings_fact
	  INNER JOIN
	    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	  INNER JOIN
	    skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
  WHERE
    job_postings_fact.job_title_short = 'Data Analyst'
		AND job_postings_fact.salary_year_avg IS NOT NULL
    AND job_postings_fact.job_work_from_home = True
  GROUP BY
    skills_dim.skill_id
),
average_salary AS (
  SELECT
    skills_job_dim.skill_id,
    AVG(job_postings_fact.salary_year_avg) AS avg_salary
  FROM
    job_postings_fact
	  INNER JOIN
	    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id 
  WHERE
    job_postings_fact.job_title_short = 'Data Analyst'
		AND job_postings_fact.salary_year_avg IS NOT NULL
    AND job_postings_fact.job_work_from_home = True
  GROUP BY
    skills_job_dim.skill_id
)

SELECT
  DISTINCT(skills_demand.skills),
  skills_demand.demand_count,
  ROUND(average_salary.avg_salary, 2) AS avg_salary --ROUND to 2 decimals 
FROM
  skills_demand
	INNER JOIN
	  average_salary ON skills_demand.skill_id = average_salary.skill_id
ORDER BY
  demand_count DESC, 
	avg_salary DESC
LIMIT 10
```
Here were the results:

skills    | demand_count | avg_salary
--------- |--------------|-----------
sql       | 398          | 97237
excel     | 256          | 87288
python    | 236          | 101397
tableau   | 230          | 99287
r         | 148          | 100498
power bi  | 110          | 97431
sas       | 63           | 98902
powerpoint| 58           | 88701
looker    | 49           | 103795
word      | 48           | 82576

Again, SQL, Excel, Python, and Tableau shows up as the most optimal skills. 


# Conclusions

From the analysis of the data job market, here were some insights that were found:

* **Skills:** The most important skill to have as a data analyst is SQL as it showed up as the most optimal skill in terms of high demand and high salary. Other important skills to have include Excel, Python, and Tableau.
* **Salaries** The top salaries are 6-figures, with the highest salary being $650,000. This shows the potential to make a 6-figure income as a data analyst.
* **Skill with the highest salary:** The skill with the highest salary includes SVN, Solidary, and Couchbase.

From doing this project, I was able to develop my SQL skills while at the same time, learn more about the data job market. I now have a better understanding of the skills that are expected for data jobs, and will focus on developing them. 
