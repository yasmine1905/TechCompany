
👉 Når du lægger det i din README.md på GitHub, bliver alt inde i blokken farvekodet som SQL.  

Her er dit hele script pakket korrekt ind:  

```sql
USE tech_company;


/********************************************************************
 *                      SINGLE TABLE ASSIGNMENTS                    *
 ********************************************************************/

-- 1) Find medarbejdernummeret for medarbejdere der hedder 'MARTIN'
-- Simpelt SELECT med WHERE på employee_name = 'MARTIN'
SELECT employee_number
FROM employees
WHERE employee_name = 'MARTIN';

-- 2) Find de medarbejdere, der har en løn større end 1500
-- Filtrer på salary > 1500
SELECT *
FROM employees
WHERE salary > 1500;

-- 3) Vis navne på sælgere (SALESMAN), der tjener mere end 1300
-- Kombiner job_title-filter og løn-filter
SELECT employee_name
FROM employees
WHERE job_title = 'SALESMAN'
  AND salary > 1300;

-- 4) Vis navne på medarbejdere, der IKKE er sælgere
-- Brug <> (forskellig fra) på job_title.
SELECT employee_name
FROM employees
WHERE job_title <> 'SALESMAN';

-- 5) Vis alle CLERK med deres løn + løn med 10% fratrukket
-- Udregn ny kolonne som salary * 0.90.
SELECT employee_name,
       salary,
       salary * 0.90 AS salary_after_10pct_deduction
FROM employees
WHERE job_title = 'CLERK';

-- 6) Find navne på medarbejdere ansat før maj 1981
-- Dato-sammenligning; maj 1981 starter 1981-05-01
SELECT employee_name
FROM employees
WHERE hire_date < '1981-05-01';

-- 7) Vis medarbejdere sorteret efter løn faldende
-- ORDER BY salary DESC.
SELECT *
FROM employees
ORDER BY salary DESC;

-- 8) Vis afdelinger sorteret efter lokation (A → Z)
SELECT *
FROM departments
ORDER BY office_location ASC;

-- 9) Find afdelingens navn der ligger i NEW YORK
-- WHERE på office_location = 'NEW YORK'.
SELECT department_name
FROM departments
WHERE office_location = 'NEW YORK';

-- 10) Find medarbejdere hvis navn starter med J og slutter med S
--  LIKE 'J%S' random tegn imellem
SELECT *
FROM employees
WHERE employee_name LIKE 'J%S';

-- 11) Samme som 10, men kun hvis job_title = MANAGER
SELECT *
FROM employees
WHERE employee_name LIKE 'J%S'
AND job_title = 'MANAGER';

-- 12) Hvor mange medarbejdere er der i hver afdeling?
-- GROUP BY department_number for at tælle per afdeling
SELECT department_number,
COUNT(*) AS num_employees
FROM employees
GROUP BY department_number
ORDER BY department_number;

-- 12b) Samme som 12, men vis også afdelingsnavn og inkluder afdelinger uden ansatte
-- LEFT JOIN fra departments til employees + COUNT over e.employee_number
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
-- Brug IFNULL(commission 0) så NULL provision behandles som 0
SELECT SUM(salary + IFNULL(commission, 0)) AS total_salary_plus_commission
FROM employees;


/********************************************************************
JOIN ASSIGNMENTS
 ********************************************************************/

-- 1) INNER JOIN: medarbejdere + deres afdelingsnavn.
-- Kun rækker hvor employee.department_number matcher en department
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

-- 3) LEFT JOIN på (employee_name, department_name)
-- Viser ALLE medarbejdere, også hvis de ikke har afdeling (NULL)
-- Hvem kommer med nu? KING (PRESIDENT) har department_number = NULL → var ikke med i INNER JOIN
SELECT e.employee_name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_number = d.department_number
ORDER BY e.employee_name;

-- 4) Overvej nedenstående query (INNER JOIN + COUNT pr. department_name)
-- Hvilken afdeling mangler og hvorfor?
-- 'OPERATIONS' (dept 40) mangler, fordi der ikke er ansatte i den afdeling
-- INNER JOIN viser kun rækker hvor der findes match i employees.
SELECT d.department_name, COUNT(e.employee_number) AS cnt
FROM employees e
JOIN departments d
ON d.department_number = e.department_number
GROUP BY d.department_name;

-- 5) Ret forrige til RIGHT JOIN (eller LEFT JOIN fra departments-siden) for at få alle afdelinger med
-- RIGHT JOIN sikrer at alle departments vises, også uden ansatte (COUNT bliver 0)
SELECT d.department_name, COUNT(e.employee_number) AS cnt
FROM employees e
RIGHT JOIN departments d
ON d.department_number = e.department_number
GROUP BY d.department_name
ORDER BY d.department_name;

-- 6) SCOTT’s selv-join-query: Forklar hvad den gør (teknisk)
-- SELECT *
-- FROM employees employee
-- JOIN employees manager
-- ON employee.manager_id = manager.employee_number
-- ORDER BY employee.employee_name;

-- Tabellen employees bruges to gange: som 'employee' (den ansatte) og 'manager' (lederen)
-- Join-betingelsen matcher employee.manager_id med manager.employee_number.
-- Resultatet er alle ansatte som har en manager registreret, og viser alle kolonner fra begge aliaser
-- ORDER BY sorterer på medarbejderens navn.

-- 7) To kolonner: medarbejder + deres manager
-- Selv join som ovenfor men vælger  kun relevante kolonner
SELECT e.employee_name AS employee,
       m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.employee_number
ORDER BY e.employee_name;

-- 8) HAVING: Afdelinger med mere end 3 medarbejdere
-- Først GROUP BY så HAVING filtrere grupperne på tælling > 3
--  I vores data har fx dept 20 og 30 mere end 3 ansatte
SELECT e.department_number,
       COUNT(e.department_number) AS number_of_employees
FROM employees e
GROUP BY e.department_number
HAVING number_of_employees > 3
ORDER BY number_of_employees DESC;

-- 9) Subquery: Ansatte med løn over gennemsnittet
-- SELECT beregner AVG(salary), som bruges i WHERE
SELECT employee_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
ORDER BY salary DESC;


/********************************************************************
JOIN TABLE MANY TO MANY
 ********************************************************************/

-- 1) Opret en ny tabel leaders (enkelt katalog over ledere)
-- Simpel PK på leader_number
DROP TABLE IF EXISTS employees_leaders;
DROP TABLE IF EXISTS leaders;

CREATE TABLE leaders (
leader_number INT PRIMARY KEY,
leader_name   VARCHAR(50) NOT NULL
);

-- Indsæt et par ledere (eksempeldata)
INSERT INTO leaders (leader_number, leader_name) VALUES
(9001, 'ALPHA'),
(9002, 'BETA'),
(9003, 'GAMMA');

-- 2) Opret join-tabel 'employees_leaders' for many-to-many
--  primærnøgle (employee_number, leader_number) + FK til begge sider
CREATE TABLE employees_leaders (
employee_number INT,
leader_number   INT,
PRIMARY KEY (employee_number, leader_number),
FOREIGN KEY (employee_number) REFERENCES employees(employee_number),
FOREIGN KEY (leader_number)   REFERENCES leaders(leader_number)
);

-- 3) Opret forbindelser mellem konkrete employees og leaders
--  flere employees til samme leader og en employee med flere leaders
INSERT INTO employees_leaders (employee_number, leader_number) VALUES
(7839, 9001),  -- KING ledes af ALPHA (som "sponsor"/board)
(7698, 9001),  -- BLAKE -> ALPHA
(7566, 9002),  -- JONES -> BETA
(7782, 9003),  -- CLARK -> GAMMA
(7902, 9002),  -- FORD  -> BETA
(7788, 9002),  -- SCOTT -> BETA
(7499, 9001),  -- ALLEN -> ALPHA
(7844, 9001),  -- TURNER -> ALPHA
(7521, 9003),  -- WARD  -> GAMMA
(7654, 9003);  -- MARTIN-> GAMMA

-- 4) Many to many query: medarbejdere og deres ledere
-- To JOINs fra employees til employees_leaders (for parrene) og videre til leaders (for navnet)
SELECT e.employee_name  AS employee,
       l.leader_name    AS leader
FROM employees e
INNER JOIN employees_leaders el
ON e.employee_number = el.employee_number
INNER JOIN leaders l
ON el.leader_number = l.leader_number
ORDER BY leader, employee;
