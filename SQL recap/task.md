# Task - Employee Records

The major corporation BusinessCorp&#8482; wants to do some analysis of varioius metrics around its employees and have contracted the job out to you. They have provided you with two SQL files, you will need to write the queries which will extract the data they are looking for.

## Setup

- Create a database to run the files against
- Use the `psql -d database -f file` command to run the SQL files
- Before starting to write the queries take some time to look at the data and figure out the relationships between the tables - maybe even draw an ERD to help.

## Tasks

### CTEs

1) Calculate the average salary of all employees

```sql
<!--SELECT
	AVG(salary) AS employee_average
FROM employees;-->
```

2) Calculate the average salary of the employees in each team (hint: you'll need to `JOIN` and `GROUP BY` here)

```sql
<!--SELECT
	departments.id,
	departments.name,
	AVG(Salary) AS department_salaryAVG
	FROM employees
	INNER JOIN departments
	ON departments.id = employees.department_id
	GROUP BY departments.id-->
```

3) Using a CTE find the ratio of each employees salary to their team average, eg. an employee earning £55000 in a team where the average is £50000 has a ratio of 1.1

```sql
<!-- we have to use CTE because we need to use the table generated in the prev question to feed into the next calculation( we want to use the department avg)

WITH department_averages (id, name, department_average) AS (
SELECT
	departments.id,
	departments.name,
	AVG(salary) AS department_average
FROM employees
INNER JOIN departments
ON employees.department_id = departments.id
GROUP BY departments.id
)

SELECT 
	employees.first_name,
	employees.last_name,
	employees.salary,
	department_averages.name,
	department_averages.department_average,
	employees.salary / department_averages.department_average AS ratio
	FROM employees
	JOIN department_averages
	ON employees.department_id = department_averages.id-->
```

4) Find the employee with the highest ratio in Argentina

```sql
<!-- just needed to manipulate the prev solution to inc a WHERE and ORDER BY DESC
WITH department_averages (id, name, department_average) AS (
SELECT
	departments.id,
	departments.name,
	AVG(salary) AS department_average
FROM employees
INNER JOIN departments
ON employees.department_id = departments.id
GROUP BY departments.id
)

SELECT 
	employees.first_name,
	employees.last_name,
	employees.salary,
	department_averages.name,
	department_averages.department_average,
	employees.salary / department_averages.department_average AS ratio
	FROM employees
	JOIN department_averages
	ON employees.department_id = department_averages.id
	WHERE employees.country = 'Argentina'
ORDER BY ratio DESC
LIMIT 1-->
```

5) **Extension:** Add a second CTE calculating the average salary for each country and add a column showing the difference between each employee's salary and their country average

```sql
<!--WITH department_averages (id, name, department_average) AS (
	SELECT
		departments.id,
		departments.name,
		AVG(salary) AS department_average
	FROM employees
	INNER JOIN departments
	ON employees.department_id = departments.id
	GROUP BY departments.id
),
country_averages (name, country_average) AS (
	SELECT
		country,
		AVG(salary)
	FROM employees
	GROUP BY country
)
SELECT
	employees.first_name,
	employees.last_name,
	employees.salary,
	department_averages.name,
	department_averages.department_average,
	employees.salary / department_averages.department_average AS ratio,
	employees.salary - country_averages.country_average AS country_difference
FROM employees
INNER JOIN department_averages
ON employees.department_id = department_averages.id
INNER JOIN country_averages
ON employees.country = country_averages.name->
```

---

### Window Functions

1) Find the running total of salary costs as the business has grown and hired more people

```sql
<!--
SELECT
	start_date,
	salary,
	SUM(SALARY) OVER (ORDER BY start_date) AS running_total_salary
FROM employees-->
```

2) Determine if any employees started on the same day (hint: some sort of ranking may be useful here)

```sql
<!-- dense ranking will show that there is 961 distinct values, where we know we have 1000 employess so we know there must be duplicates
SELECT 
	RANK() OVER (ORDER BY start_date) AS simple_ranking,
	DENSE_RANK() OVER (ORDER BY start_date) AS dense_ranking
FROM employees
ORDER BY simple_ranking DESC->
```

3) Find how many employees there are from each country

```sql
<!--SELECT
	DISTINCT(country),
	COUNT(*) OVER (PARTITION BY country)
FROM employeese-->
```

4) Show how the average salary cost for each department has changed as the number of employees has increased

```sql
<!--
SELECT
	employees.first_name,
	employees.last_name,
	departments.name,
	employees.start_date,
	employees.salary,
	AVG(employees.salary) OVER (PARTITION BY employees.department_id ORDER BY employees.start_date) AS AV_dept_salary
FROM employees
INNER JOIN departments
ON employees.department_id = departments.id-->
```

5) **Extension:** Research the `EXTRACT` function and use it in conjunction with `PARTITION` and `COUNT` to show how many employees started working for BusinessCorp&#8482; each year. If you're feeling adventurous you could further partition by month...

```sql
<!--Copy solution here-->
```

---

### Combining the two

1) Find the maximum and minimum salaries

```sql
<!--Copy solution here-->
```

2) Find the difference between the maximum and minimum salaries and each employee's own salary

```sql
<!--Copy solution here-->
```

3) Order the employees by start date. Research how to calculate the **median** salary value and the **standard deviation** in salary values and show how these change as more employees join the company

```sql
<!--Copy solution here-->
```

4) Limit this query to only Research & Development team members and show a rolling value for only the 5 most recent employees.

```sql
<!-- this means that only 5 of the most recent employees are inc in the calculation, so the first 5 employees are averagd, when the 6th joins, the 1st employee is kicked off and employees 2- are avergared and so on
SELECT
	first_name,
	last_name,
	departments.name,
	start_date,
	salary,
	STDDEV(salary) OVER (ORDER BY start_date ROWS 5 PRECEDING) AS salary_std_dev
FROM employees
INNER JOIN departments
ON employees.department_id = departments.id
WHERE departments.name = 'Research and Development';
-->
```

