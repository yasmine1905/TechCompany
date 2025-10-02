USE tech_company;


/********************************************************************
 *                      SINGLE TABLE ASSIGNMENTS                    *
 ********************************************************************/

-- 1) Find medarbejdernummeret for medarbejdere der hedder 'MARTIN'
SELECT employee_number
FROM employees
WHERE employee_name = 'MARTIN';

-- 2) Find de medarbejdere, der har en løn større end 1500
SELECT *
FROM employees
WHERE salary > 1500;

-- 3) Vis navne på sælgere (SALESMAN), der tjener mere end 1300
SELECT employee_name
FROM employees
WHERE job_title = 'SALESMAN'
  AND salary > 1300;

-- 4) Vis navne på medarbejdere, der IKKE er sælgere
SELECT employee_name
FROM employees
WHERE job_title <> 'SALESMAN';

-- 5) Vis alle CLERK med deres løn + løn med 10% fratrukket
SELECT employee_name,
       salary,
       salary * 0.90 AS salary_after_10pct_deduction
FROM employees
WHERE job_title = 'CLERK';

-- 6) Find navne på medarbejdere ansat før maj 1981
SELECT employee_name
FROM employees
WHERE hire_date < '1981-05-01';

-- 7) Vis medarbejdere sorteret efter løn faldende
SELECT *
FROM employees
ORDER BY salary DESC;

-- 8) Vis afdelinger sorteret efter lokation (A → Z)
SELECT *
FROM departments
ORDER BY office_location ASC;

-- 9) Find afdelingens navn der ligger i NEW YORK
SELECT department_name
FROM departments
WHERE office_location = 'NEW YORK';

-- 10) Find medarbejdere hvis navn starter med J og slutter med S
SELECT *
FROM employees
WHERE employee_name LIKE 'J%S';

-- 11) Samme som 10, men kun hvis job_title = MANAGER
SELECT *
FROM employees
WHERE employee_name LIKE 'J%S'
AND job_title = 'MANAGER';

-- 12) Hvor mange medarbejdere er der i hver afdeling?
SELECT department_number,
COUNT(*) AS num_employees
FROM employees
GROUP BY department_number
ORDER BY department_number;

-- 12b) Samme som 12, men vis også afdelingsnavn og inkluder afdelinger uden ansatte
SELECT d.department_number,
d.department_name,
COUNT(e.employee_number) AS num_employees
FROM departments d
LEFT JOIN employees e
ON e.department_number = d.department_number
GROUP BY d.department_number, d.department_name
ORDER BY d.department_number;


/********************************************************************
 *                          AGGREGATE FUNCTIONS                     *
 ********************************************************************/

-- A1) Vis samlet antal medarbejdere
SELECT COUNT(*) AS total_employees
FROM employees;

-- A2) Vis summen af alle lønninger (ekskl. provision)
SELECT SUM(salary) AS total_salary
FROM employees;

-- A3) Vis gennemsnitsløn for afdeling 20
SELECT AVG(salary) AS avg_salary_dept20
FROM employees
WHERE department_number = 20;

-- A4) Vis unikke jobtitler i virksomheden
SELECT DISTINCT job_title
FROM employees;

-- A5) Vis antal medarbejdere i hver afdeling (med afdelingsnavn)
SELECT d.department_number,
d.department_name,
COUNT(e.employee_number) AS num_employees
FROM departments d
LEFT JOIN employees e
ON e.department_number = d.department_number
GROUP BY d.department_number, d.department_name
ORDER BY d.department_number;

-- A6) Vis højeste løn i hver afdeling, sorteret faldende efter max-løn
SELECT department_number,
       MAX(salary) AS max_salary
FROM employees
GROUP BY department_number
ORDER BY max_salary DESC;

-- A7) Vis samlet sum af (løn + provision) for alle medarbejdere
SELECT SUM(salary + IFNULL(commission, 0)) AS total_salary_plus_commission
FROM employees;


/********************************************************************
 *                          JOIN ASSIGNMENTS                        *
 ********************************************************************/

-- 1) INNER JOIN: medarbejdere + deres afdelingsnavn.
SELECT *
FROM employees e
INNER JOIN departments d
ON e.department_number = d.department_number;

-- 2) Kun to kolonner (employee_name, department_name) sorteret A→Z på employee_name
SELECT e.employee_name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_number = d.department_number
ORDER BY e.employee_name ASC;

-- 3) LEFT JOIN på (employee_name, department_name) - viser ALLE medarbejdere
SELECT e.employee_name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_number = d.department_number
ORDER BY e.employee_name;

-- 4) INNER JOIN + COUNT pr. department_name - 'OPERATIONS' mangler fordi den ingen ansatte har
SELECT d.department_name, COUNT(e.employee_number) AS cnt
FROM employees e
JOIN departments d
ON d.department_number = e.department_number
GROUP BY d.department_name;

-- 5) RIGHT JOIN for at få alle afdelinger med
SELECT d.department_name, COUNT(e.employee_number) AS cnt
FROM employees e
RIGHT JOIN departments d
ON d.department_number = e.department_number
GROUP BY d.department_name
ORDER BY d.department_name;

-- 6) Selv-join (medarbejder og deres manager)
SELECT e.employee_name AS employee,
       m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.employee_number
ORDER BY e.employee_name;

-- 7) HAVING: Afdelinger med mere end 3 medarbejdere
SELECT e.department_number,
       COUNT(e.department_number) AS number_of_employees
FROM employees e
GROUP BY e.department_number
HAVING number_of_employees > 3
ORDER BY number_of_employees DESC;

-- 8) Subquery: Ansatte med løn over gennemsnittet
SELECT employee_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
ORDER BY salary DESC;


/********************************************************************
 *                   JOIN TABLE MANY TO MANY                        *
 ********************************************************************/

-- 1) Opret en ny tabel leaders
DROP TABLE IF EXISTS employees_leaders;
DROP TABLE IF EXISTS leaders;

CREATE TABLE leaders (
leader_number INT PRIMARY KEY,
leader_name   VARCHAR(50) NOT NULL
);

-- Indsæt ledere
INSERT INTO leaders (leader_number, leader_name) VALUES
(9001, 'ALPHA'),
(9002, 'BETA'),
(9003, 'GAMMA');

-- 2) Opret join-tabel 'employees_leaders'
CREATE TABLE employees_leaders (
employee_number INT,
leader_number   INT,
PRIMARY KEY (employee_number, leader_number),
FOREIGN KEY (employee_number) REFERENCES employees(employee_number),
FOREIGN KEY (leader_number)   REFERENCES leaders(leader_number)
);

-- 3) Opret forbindelser mellem konkrete employees og leaders
INSERT INTO employees_leaders (employee_number, leader_number) VALUES
(7839, 9001),  -- KING -> ALPHA
(7698, 9001),  -- BLAKE -> ALPHA
(7566, 9002),  -- JONES -> BETA
(7782, 9003),  -- CLARK -> GAMMA
(7902, 9002),  -- FORD  -> BETA
(7788, 9002),  -- SCOTT -> BETA
(7499, 9001),  -- ALLEN -> ALPHA
(7844, 9001),  -- TURNER -> ALPHA
(7521, 9003),  -- WARD  -> GAMMA
(7654, 9003);  -- MARTIN -> GAMMA

-- 4) Many to many query: medarbejdere og deres ledere
SELECT e.employee_name  AS employee,
       l.leader_name    AS leader
FROM employees e
INNER JOIN employees_leaders el
ON e.employee_number = el.employee_number
INNER JOIN leaders l
ON el.leader_number = l.leader_number
ORDER BY leader, employee;
