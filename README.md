# Data Science Salary Analysis
**Goal of the Project:** Examine the salary distribution of data science positions around the world and in the United States with SQL. 
## Installation
MySQL and SQL Studio

## Data Set
Attached on the code tree is a dataset called DS_Salaries. Make sure you download it. 
There is another data set called DataScience. This contains the answers after running the code.

## Hands-on

**1. Load the dataset into MySQL**

Follow these steps:
1. Create a schema and name it
2. Right-click into the tables and hit "Table Data Import Wizard"
3. Load the data from your computer
4. Apply and Finish. The data is now stored in *Tables*

**2. Purpose of the project**

Make sure you understand the goals of the project before your analysis. Write down what the goals are and what answers you want to answer. For me, they are:
1. For FT roles in the US, find the percentile of each salary depending on the experience level.
2. For FT roles in the US, What is the min-max-average and Q1, Q3 salary of each experience level?
3. For FT roles in the US, what are the job titles that get paid the highest?

### Analysis 

**Q1.Highest Salary of Each Experience Level**

In this analysis, we will see that depending on each experience level (entry, mid or senior...), what the highest salary is. Therefore, we will group the salary in the order of highest to lowest by the experience level.
```
SELECT CASE WHEN experience_level = 'EN' THEN 'Entry Level'
WHEN experience_level = 'SE' THEN 'Senior Level'
WHEN experience_level = 'EX' THEN 'Executive Level'
WHEN experience_level = 'MI' THEN 'Mid Level'
ELSE NULL END AS experience_cateogory,
FORMAT(MAX(salary_in_usd), '###, ###') AS highest_salary
FROM dssalary.ds_salaries
GROUP BY experience_level
ORDER BY highest_salary DESC
```

If you check the DataScience excel sheet, you can see that the highest salary is USD$600,000 and the experience category is Executive Level. Followed by Executive Level is Mid Level, Senior Level and Entry Level. It is quite interesting to see that mid level can get paid higher than senior level.

**Q2.What is the percentile of your salary depending on each experience level?** 

So, there are 2 ways of doing this. My first way is to divide the salary pool up into 4 quarter in each experience level. The one that get allocated in the 
first quarter will be the one with the lowest salary and the one that get allocated in the fourth quarter will be the one with the highest salary. Let's
demonstrate that:

*Q2a)*

```
SELECT salary_in_usd, experience_level,  
ROUND(ntile(4) OVER ( PARTITION BY experience_level ORDER BY salary_in_usd), 2) percentile_rank
FROM dssalary.ds_salaries
WHERE employment_type = 'FT' AND company_location = 'US'
```

If you check the answer sheet, you will see that for each experience level, salaries will be divided into 4 equal pool in the ascending order. Now, we have a second way to do this. We can find out the percentile of each salary of different experience level. For example, If a salary is at 96% percentile, that means that salary is higher than 96% of the salaries in the same experience level. We can do that as below:

*Q2b)*
```
SELECT FORMAT(salary_in_usd, '###, ###') AS salary, experience_level,  
ROUND(PERCENT_RANK() OVER (PARTITION BY experience_level ORDER BY salary_in_usd), 2) percentile_rank
FROM dssalary.ds_salaries
WHERE employment_type = 'FT' AND company_location = 'US'   
ORDER BY experience_level
```

**Q3. What are the highest paying job titles?**

If we don't take experience level into account, what are the highest paying job titles? 

```
SELECT job_title,
FORMAT(ROUND(salary_in_usd), '###,###') AS highest_salary
FROM dssalary.ds_salaries
WHERE company_location = 'US' AND employment_type = 'FT'
ORDER BY  (salary_in_usd) desc
```

The highest paying job is the Principal Data Engineer with an annual salary of around USD$600,000. Followed by Research Scientist, Financial Data Analyst, Applied ML Scientist and Data Scientist. Since this is a full list, it can be a bit long. If you are more interested in learning the Top 10 most high paying job, add "LIMIT 5" or "LIMIT 10" in the end. Note that since we don't remove duplicate job titles, one job title can be repeated over and over again (for example, the data scientist position).

**Q4. What is the minimum, average and maximum salaries of each experience level?**

Like the title, let's find out what is the min, average and maximum salary of each experience level. In order to make it easier to comprehend, let's organize the salary in the order of experience level: entry level going first, respectively followed by mid level, senior level and executive level.
```
SELECT
experience_level
FORMAT(MIN(salary_in_usd), '###,###') AS minimum_salary,
FORMAT(AVG(salary_in_usd), '###,###') AS average_salary,
FORMAT(MAX(salary_in_usd), '###,###') AS maximum_salary
FROM dssalary.ds_salaries
WHERE company_location = 'US' AND employment_type = 'FT'
GROUP BY experience_level
ORDER BY CASE 
WHEN experience_level = 'EN' THEN 0
WHEN experience_level = 'MI' THEN 1
WHEN experience_level = 'SE' THEN 2
WHEN experience_level = 'EX' THEN 3 END;
```

**Q5. What is the minimum, median, Q1, Q3 and maximum salaries of each experience level?**
One of the more complicating code of the project, let's find out the 1st quartile, median and 3rd quartile of the salaries of each experience level. In statistics, 1st quartile is defined as *the middle number between the minimum and the median* , median is the *middle number of the data set, thus 50% of the data below the point*, Q3 is the *middle value between the median and maximum value of the data set.* 

Let's go back to question 2a where we rank our salary pool in 4 equal parts with 1 being the lowest salaries and 4 being the highest. In order to find out the Q1 salary of each experience level, we need to find out the maximum salary value of those belonging to percentile 1. For example, in the DataScience Q2a, in the entry level positions, the maximum salary that belongs to percentile_rank 1 is $70,000. Therefore, USD$70,000 will be Q1. Applying the same logics, the median will be the maximum salary value of Q2 and Q3 will the maximum salary value of Q3.

In order to do, we need to create different tables and inner join them all together.

```
Select Q1.experience_level, minimum_salary, 1stquartile, median_salary, 3rdquartile, maximum_salary
FROM
(SELECT  Q.experience_level, MAX(Q.salary_in_usd) AS 1stquartile
FROM
(SELECT salary_in_usd, experience_level,  
ROUND(ntile(4) OVER ( PARTITION BY experience_level ORDER BY salary_in_usd), 2) percentile_rank
FROM dssalary.ds_salaries
WHERE employment_type = 'FT' AND company_location = 'US') as Q
WHERE percentile_rank=1
GROUP BY experience_level) as Q1
inner join
(SELECT  Q.experience_level, MAX(Q.salary_in_usd) AS 3rdquartile
FROM
(SELECT salary_in_usd, experience_level,  
ROUND(ntile(4) OVER ( PARTITION BY experience_level ORDER BY salary_in_usd), 2) percentile_rank
FROM dssalary.ds_salaries
WHERE employment_type = 'FT' AND company_location = 'US') as Q
WHERE percentile_rank=3
GROUP BY experience_level) as Q2
on Q1.experience_level = Q2.experience_level
INNER JOIN 
(SELECT Q.experience_level, MAX(Q.salary_in_usd) AS median_salary
FROM
(SELECT salary_in_usd, experience_level,  
ROUND(ntile(4) OVER ( PARTITION BY experience_level ORDER BY salary_in_usd), 2) percentile_rank
FROM dssalary.ds_salaries
WHERE employment_type = 'FT' AND company_location = 'US') as Q
WHERE percentile_rank=2
GROUP BY experience_level) as median
on median.experience_level = Q2.experience_level
INNER JOIN 
(SELECT  minimum.experience_level, MIN(minimum.salary_in_usd) AS minimum_salary, MAX(minimum.salary_in_usd) AS maximum_salary
FROM
(select salary_in_usd,  experience_level
FROM dssalary.ds_salaries
WHERE employment_type = 'FT' AND company_location = 'US') as minimum
GROUP BY experience_level) as Small
on Q2.experience_level = Small.experience_level
ORDER BY CASE
WHEN Q2.experience_level = 'EN' THEN 0
WHEN Q2.experience_level = 'MI' THEN 1
WHEN Q2.experience_level = 'SE' THEN 2
WHEN Q2.experience_level = 'EX' THEN 3 END
```
This creates a table with experience level, minimum salary, 1st quartile, median, 3rd quartile and maximum salary. 
### ACKNOLEDGEMENT
[1] Bahaa Khaled, Data Science Salaries Data Set: https://www.kaggle.com/code/bahaakhaled97/data-science-salaries-eda/input
