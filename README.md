# SQL-13
CREATE TABLE categories (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    parent_id INT REFERENCES categories(id)
);

INSERT INTO categories VALUES
(1, 'Eshittirish jihozlari', NULL),
(2, 'Audio', 1),
(3, 'Video', 1),
(4, 'Quloqchinlar', 2),
(5, 'Kolonkalar', 2);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    salary INT
);

INSERT INTO employees VALUES
(101, 'Ali', 'IT', 5000),
(102, 'Vali', 'IT', 6000),
(103, 'Sami', 'IT', 6000),
(104, 'Olim', 'HR', 4000),
(105, 'Zuhra', 'HR', 4500),
(106, 'Karim', 'Sales', 3500);

CREATE TABLE sales (
    sale_date DATE,
    amount INT
);

INSERT INTO sales VALUES
('2026-01-01', 100),
('2026-01-02', 150),
('2026-01-03', 120),
('2026-01-04', 200),
('2026-01-05', 250);

SELECT name, salary,
       (SELECT AVG(salary) FROM employees WHERE department = 'IT') AS it_avg_salary
FROM employees
WHERE department = 'IT';

SELECT name, department 
FROM employees 
WHERE department IN (SELECT DISTINCT department FROM employees WHERE salary > 4000);

SELECT e.name, e.department 
FROM employees e
WHERE EXISTS (
    SELECT 1 FROM sales s WHERE s.amount > e.salary * 0.05 
);

WITH total_it_budget AS (
    SELECT SUM(salary) as it_sum FROM employees WHERE department = 'IT'
),
total_hr_budget AS (
    SELECT SUM(salary) as hr_sum FROM employees WHERE department = 'HR'
)
SELECT 
    'IT vs HR Taqqoslovi' AS hisobot_turi,
    it_sum AS it_budjeti,
    hr_sum AS hr_budjeti,
    (it_sum - hr_sum) AS farq
FROM total_it_budget, total_hr_budget;

WITH RECURSIVE generate_numbers AS (
    SELECT 1 AS num
    UNION ALL
    SELECT num + 1 
    FROM generate_numbers 
    WHERE num < 10
)
SELECT num FROM generate_numbers;

WITH RECURSIVE category_tree AS (
    SELECT id, name, parent_id, CAST(name AS TEXT) AS path
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    SELECT c.id, c.name, c.parent_id, CAST(ct.path || ' -> ' || c.name AS TEXT)
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT id, name, path FROM category_tree;

WITH ranked_employees AS (
    SELECT 
        department,
        name,
        salary,
        ROW_NUMBER() OVER(PARTITION BY department ORDER BY salary DESC) as rn,
        RANK() OVER(PARTITION BY department ORDER BY salary DESC) as rnk,
        DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) as drnk
    FROM employees
)
SELECT * FROM ranked_employees;

WITH ranked_employees AS (
    SELECT department, name, salary,
           ROW_NUMBER() OVER(PARTITION BY department ORDER BY salary DESC) as rn
    FROM employees
)
SELECT department, name, salary 
FROM ranked_employees 
WHERE rn = 1;

SELECT 
    sale_date,
    amount,
    
    LAG(amount, 1) OVER(ORDER BY sale_date) AS previous_day_amount,
    
    LEAD(amount, 1) OVER(ORDER BY sale_date) AS next_day_amount,
    
    SUM(amount) OVER(ORDER BY sale_date) AS cumulative_sum,
    
    AVG(amount) OVER(ORDER BY sale_date ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) AS moving_avg,
    
    ROUND((amount * 100.0) / SUM(amount) OVER(), 2) AS sale_percentage_of_total

FROM sales;
