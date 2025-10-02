# sqlAssignment 1

USE tech_company;

SELECT employee_number
FROM employees
WHERE employee_name = 'MARTIN';

SELECT *
FROM employees
WHERE salary > 1500;

SELECT employee_name
FROM employees
WHERE job_title = 'SALESMAN'
  AND salary > 1300;

SELECT employee_name
FROM employees
WHERE job_title <> 'SALESMAN';

SELECT employee_name,
       salary,
       salary * 0.90 AS salary_after_10pct_deduction
FROM employees
WHERE job_title = 'CLERK';

SELECT employee_name
FROM employees
WHERE hire_date < '1981-05-01';

SELECT *
FROM employees
ORDER BY salary DESC;

SELECT *
FROM departments
ORDER BY office_location ASC;

SELECT department_name
FROM departments
WHERE office_location = 'NEW YORK';

SELECT *
FROM employees
WHERE employee_name LIKE 'J%S';

SELECT *
FROM employees
WHERE employee_name LIKE 'J%S'
  AND job_title = 'MANAGER';

SELECT department_number,
       COUNT(*) AS num_employees
FROM employees
GROUP BY department_number
ORDER BY department_number;

SELECT d.department_number,
       d.department_name,
       COUNT(e.employee_number) AS num_employees
FROM departments d
LEFT JOIN employees e
  ON e.department_number = d.department_number
GROUP BY d.department_number, d.department_name
ORDER BY d.department_number;


# sqlAssignment 2

SELECT COUNT(*) AS total_employees
FROM employees;

SELECT SUM(salary) AS total_salary
FROM employees;

SELECT AVG(salary) AS avg_salary_dept20
FROM employees
WHERE department_number = 20;

SELECT DISTINCT job_title
FROM employees;

SELECT d.department_number,
       d.department_name,
       COUNT(e.employee_number) AS num_employees
FROM departments d
LEFT JOIN employees e
  ON e.department_number = d.department_number
GROUP BY d.department_number, d.department_name
ORDER BY d.department_number;

SELECT department_number,
       MAX(salary) AS max_salary
FROM employees
GROUP BY department_number
ORDER BY max_salary DESC;

SELECT SUM(salary + IFNULL(commission, 0)) AS total_salary_plus_commission
FROM employees;


# sqlAssignment 3

SELECT *
FROM employees e
INNER JOIN departments d
  ON e.department_number = d.department_number;

SELECT e.employee_name, d.department_name
FROM employees e
INNER JOIN departments d
  ON e.department_number = d.department_number
ORDER BY e.employee_name ASC;

SELECT e.employee_name, d.department_name
FROM employees e
LEFT JOIN departments d
  ON e.department_number = d.department_number
ORDER BY e.employee_name;

SELECT d.department_name, COUNT(e.employee_number) AS cnt
FROM employees e
JOIN departments d
  ON d.department_number = e.department_number
GROUP BY d.department_name;

SELECT d.department_name, COUNT(e.employee_number) AS cnt
FROM employees e
RIGHT JOIN departments d
  ON d.department_number = e.department_number
GROUP BY d.department_name
ORDER BY d.department_name;

SELECT e.employee_name AS employee,
       m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
  ON e.manager_id = m.employee_number
ORDER BY e.employee_name;

SELECT e.department_number,
       COUNT(e.department_number) AS number_of_employees
FROM employees e
GROUP BY e.department_number
HAVING number_of_employees > 3
ORDER BY number_of_employees DESC;

SELECT employee_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
ORDER BY salary DESC;


# sqlAssignment 4

DROP TABLE IF EXISTS employees_leaders;
DROP TABLE IF EXISTS leaders;

CREATE TABLE leaders (
  leader_number INT PRIMARY KEY,
  leader_name   VARCHAR(50) NOT NULL
);

INSERT INTO leaders (leader_number, leader_name) VALUES
  (9001, 'ALPHA'),
  (9002, 'BETA'),
  (9003, 'GAMMA');

CREATE TABLE employees_leaders (
  employee_number INT,
  leader_number   INT,
  PRIMARY KEY (employee_number, leader_number),
  FOREIGN KEY (employee_number) REFERENCES employees(employee_number),
  FOREIGN KEY (leader_number)   REFERENCES leaders(leader_number)
);

INSERT INTO employees_leaders (employee_number, leader_number) VALUES
  (7839, 9001),
  (7698, 9001),
  (7566, 9002),
  (7782, 9003),
  (7902, 9002),
  (7788, 9002),
  (7499, 9001),
  (7844, 9001),
  (7521, 9003),
  (7654, 9003);

SELECT e.employee_name  AS employee,
       l.leader_name    AS leader
FROM employees e
INNER JOIN employees_leaders el
  ON e.employee_number = el.employee_number
INNER JOIN leaders l
  ON el.leader_number = l.leader_number
ORDER BY leader, employee;
