**02 Normalization**



#### **Task 1: First Normal Form (1NF) Analysis**



**Given the following un-normalized flat table that was used before the EMS database was designed:**

**EMP\_RAW ( emp\_id, full\_name, phone\_numbers, dept\_name, dept\_location, job\_titles, salary )**

**(a) Identify every violation of 1NF in this table and explain each one clearly.**



1.phone\_nubers

is multi-valued. stores two phone numbers in one field.



2.job\_title

is multi-valued. An employee can hold several job titles stored as a comma-separated list.



3.dept\_name + dept\_location

redundant data. Every employee in the IT department stores the same values "IT" and "Muscat" over and over. This is a repeating group — data about a department being embedded and duplicated inside every employee row rather than stored once in its own entity.



4.full\_name

is a composite,Combining first name and last name into one column means neither part can be used independently.





**(b) Rewrite the table in 1NF. Show the resulting table(s) with at least 3 rows of sample data.**



1.EMPLOYEE

(emp\_id, fname, lname, salary, dept\_id)



|emp\_id|fname|lname|salary|dept\_id|
|-|-|-|-|-|
|1|Ali|Hassan|1200|10|
|2|Sara|Noor|1500|20|
|3|Omar|Said|1300|10|







2.DEPARTMENT

(dept\_id, dept\_name, dept\_location)



|dept\_id|dept\_name|dept\_location|
|-|-|-|
|10|HR|Muscat|
|20|Finance|Sohar|





3.EMP\_PHONE

(emp\_id, phone\_number)





|emp\_id|phone\_number|
|-|-|
|1|99887766|
|1|99775544|
|2|91234567|





4.EMP\_JOB

(emp\_id, job\_title)



|emp\_id|job\_title|
|-|-|
|1|Recruiter|
|2|Accountant|
|3|Analyst|
|3|Auditor|







**(c) Check your own EMS schema (from Section 01) — confirm that all 6 tables satisfy 1NF and explain why.**





1.JOB\_DEPARTMENT----(job\_dept , name, description , salary\_range)holds one single text value , and job\_ID is unique PK.



2.SALARY\_BOUNS----(amount , annul , bonus)No multi-valued, salary\_ID is PK and job\_ID is FK.



3.EMPLOYEE----(fname , lname, gender, age, contact\_add, emp\_email , emp\_pass)  full\_name was split into fname + lname, emp\_ID is PK and job\_ID , salary\_id are FK.



4.QUALIFICATION---(position , date\_in , requirements)

PK is qual\_ID and emp\_ID is FK.treated as one atomic descriptive text value (VARCHAR2), not a list.



5.LEAVE----(reason , date )date is one DATE value, reason is one VARCHAR2 string ,No multi-valued: one leave record per row, not a list of dates. emp\_ID FK and PK is leave\_ID.



6.PAYROL-----(date , report , total\_amount) every column holds exactly one value — no lists anywhere ,No multi-valued: each FK references one row in its parent table ,PK: payroll\_ID.





**------------------------------------------------------------------------------------------------------------**

#### **03 SQL DML — INSERT / UPDATE / DELETE / SELECT**



Task 1: Populate All Tables with Seed Data Easy 1 pts

Write INSERT statements to populate the database with realistic seed data:

• At least 5 departments, 10 employees, 5 salary/bonus records, 8 payroll records, 5 leave records, 5

qualification records.

• Use Oracle sequences (CREATE SEQUENCE … START WITH 1 INCREMENT BY 1) to generate PKs for all

tables.

• Ensure referential integrity — do not insert child records before parent records.







\----------- TABLE 1: JOB\_DEPARTMENT



CREATE TABLE JOB\_DEPARTMENT (

&#x20;   job\_ID       NUMBER(10)    CONSTRAINT pk\_job\_dept    PRIMARY KEY,

&#x20;   job\_dept     VARCHAR2(100) NOT NULL,

&#x20;   name         VARCHAR2(150) NOT NULL,

&#x20;   description  VARCHAR2(500),

&#x20;   salary\_range VARCHAR2(50)

);



\----------- TABLE 2: SALARY\_BONUS



CREATE TABLE SALARY\_BONUS (

&#x20;   salary\_ID  NUMBER(10)   CONSTRAINT pk\_salary\_bonus  PRIMARY KEY,

&#x20;   amount     NUMBER(12,2) NOT NULL

&#x20;                           CONSTRAINT chk\_salary\_amount CHECK (amount > 0),

&#x20;   annual     NUMBER(12,2) NOT NULL,

&#x20;   bonus      NUMBER(12,2) NOT NULL,

&#x20;   job\_ID     NUMBER(10)   NOT NULL,

&#x20;   CONSTRAINT fk\_salary\_job

&#x20;       FOREIGN KEY (job\_ID) REFERENCES JOB\_DEPARTMENT (job\_ID)

);



\----------- TABLE 3: EMPLOYEE



CREATE TABLE EMPLOYEE (

&#x20;   emp\_ID      NUMBER(10)    CONSTRAINT pk\_employee     PRIMARY KEY,

&#x20;   fname       VARCHAR2(100) NOT NULL,

&#x20;   lname       VARCHAR2(100) NOT NULL,

&#x20;   gender      CHAR(1)       NOT NULL

&#x20;                             CONSTRAINT chk\_emp\_gender CHECK (gender IN ('M','F')),

&#x20;   age         NUMBER(3)     NOT NULL,

&#x20;   contact\_add VARCHAR2(300) NOT NULL,

&#x20;   emp\_email   VARCHAR2(200) NOT NULL,

&#x20;   emp\_pass    VARCHAR2(255) NOT NULL,

&#x20;   job\_ID      NUMBER(10)    NOT NULL,

&#x20;   salary\_ID   NUMBER(10)    NOT NULL,

&#x20;   CONSTRAINT fk\_emp\_job

&#x20;       FOREIGN KEY (job\_ID)    REFERENCES JOB\_DEPARTMENT (job\_ID),

&#x20;   CONSTRAINT fk\_emp\_salary

&#x20;       FOREIGN KEY (salary\_ID) REFERENCES SALARY\_BONUS   (salary\_ID)

);



\----------- TABLE 4: QUALIFICATION

CREATE TABLE QUALIFICATION (

&#x20;   qual\_ID      NUMBER(10)    CONSTRAINT pk\_qualification PRIMARY KEY,

&#x20;   position     VARCHAR2(150) NOT NULL,

&#x20;   requirements VARCHAR2(500) NOT NULL,

&#x20;   date\_in      DATE          NOT NULL,

&#x20;   emp\_ID       NUMBER(10)    NOT NULL,

&#x20;   CONSTRAINT fk\_qual\_emp

&#x20;       FOREIGN KEY (emp\_ID) REFERENCES EMPLOYEE (emp\_ID)

);



\-------------- TABLE 5: LEAVE

CREATE TABLE LEAVE (

&#x20;   leave\_ID  NUMBER(10)    CONSTRAINT pk\_leave PRIMARY KEY,

&#x20;   date\_from DATE          NOT NULL,

&#x20;   reason    VARCHAR2(500) NOT NULL,

&#x20;   emp\_ID    NUMBER(10)    NOT NULL,

&#x20;   CONSTRAINT fk\_leave\_emp

&#x20;       FOREIGN KEY (emp\_ID) REFERENCES EMPLOYEE (emp\_ID)

);





\--------------- TABLE 6: PAYROLL

CREATE TABLE PAYROLL (

&#x20;   payroll\_ID   NUMBER(10)    CONSTRAINT pk\_payroll PRIMARY KEY,

&#x20;   date\_paid    DATE          NOT NULL,

&#x20;   report       VARCHAR2(500) NOT NULL,

&#x20;   total\_amount NUMBER(12,2)  NOT NULL,

&#x20;   emp\_ID       NUMBER(10)    NOT NULL,

&#x20;   job\_ID       NUMBER(10)    NOT NULL,

&#x20;   salary\_ID    NUMBER(10)    NOT NULL,

&#x20;   leave\_ID     NUMBER(10),

&#x20;   CONSTRAINT fk\_payroll\_emp

&#x20;       FOREIGN KEY (emp\_ID)    REFERENCES EMPLOYEE       (emp\_ID),

&#x20;   CONSTRAINT fk\_payroll\_job

&#x20;       FOREIGN KEY (job\_ID)    REFERENCES JOB\_DEPARTMENT (job\_ID),

&#x20;   CONSTRAINT fk\_payroll\_salary

&#x20;           FOREIGN KEY (salary\_ID) REFERENCES SALARY\_BONUS   (salary\_ID),

&#x20;   CONSTRAINT fk\_payroll\_leave

&#x20;       FOREIGN KEY (leave\_ID)  REFERENCES LEAVE          (leave\_ID)

);







CREATE SEQUENCE job\_seq START WITH 1 INCREMENT BY 1;

CREATE SEQUENCE salary\_seq START WITH 1 INCREMENT BY 1;

CREATE SEQUENCE emp\_seq START WITH 1 INCREMENT BY 1;

CREATE SEQUENCE qual\_seq START WITH 1 INCREMENT BY 1;

CREATE SEQUENCE leave\_seq START WITH 1 INCREMENT BY 1;

CREATE SEQUENCE payroll\_seq START WITH 1 INCREMENT BY 1;





INSERT INTO JOB\_DEPARTMENT (job\_ID, job\_dept, name, description, salary\_range)

VALUES (job\_seq.NEXTVAL, 'HR', 'Human Resources', 'Handles recruitment and employee relations', '1200-1500');



INSERT INTO JOB\_DEPARTMENT (job\_ID, job\_dept, name, description, salary\_range)

VALUES (job\_seq.NEXTVAL, 'FIN', 'Finance', 'Manages company finances', '1500-2000');



INSERT INTO JOB\_DEPARTMENT (job\_ID, job\_dept, name, description, salary\_range)

VALUES (job\_seq.NEXTVAL, 'IT', 'Information Technology', 'Maintains systems and networks', '2000-2500');



INSERT INTO JOB\_DEPARTMENT (job\_ID, job\_dept, name, description, salary\_range)

VALUES (job\_seq.NEXTVAL, 'MKT', 'Marketing', 'Promotes products and services', '1400-1800');



INSERT INTO JOB\_DEPARTMENT (job\_ID, job\_dept, name, description, salary\_range)

VALUES (job\_seq.NEXTVAL, 'OPS', 'Operations', 'Oversees daily business processes', '1300-1600');





INSERT INTO SALARY\_BONUS (salary\_ID, amount, annual, bonus, job\_ID)

VALUES (salary\_seq.NEXTVAL, 1200, 14400, 500, 1);



INSERT INTO SALARY\_BONUS (salary\_ID, amount, annual, bonus, job\_ID)

VALUES (salary\_seq.NEXTVAL, 1500, 18000, 600, 2);



INSERT INTO SALARY\_BONUS (salary\_ID, amount, annual, bonus, job\_ID)

VALUES (salary\_seq.NEXTVAL, 2000, 24000, 800, 3);



INSERT INTO SALARY\_BONUS (salary\_ID, amount, annual, bonus, job\_ID)

VALUES (salary\_seq.NEXTVAL, 1400, 16800, 400, 4);



INSERT INTO SALARY\_BONUS (salary\_ID, amount, annual, bonus, job\_ID)

VALUES (salary\_seq.NEXTVAL, 1300, 15600, 300, 5);





INSERT INTO EMPLOYEE (emp\_ID, fname, lname, gender, age, contact\_add, emp\_email, emp\_pass, job\_ID, salary\_ID)

VALUES (emp\_seq.NEXTVAL, 'Ali', 'Hassan', 'M', 30, 'Muscat', 'ali.hassan@company.com', 'pass123', 1, 1);



INSERT INTO EMPLOYEE (emp\_ID, fname, lname, gender, age, contact\_add, emp\_email, emp\_pass, job\_ID, salary\_ID)

VALUES (emp\_seq.NEXTVAL, 'Sara', 'Noor', 'F', 28, 'Sohar', 'sara.noor@company.com', 'pass123', 2, 2);



INSERT INTO EMPLOYEE (emp\_ID, fname, lname, gender, age, contact\_add, emp\_email, emp\_pass, job\_ID, salary\_ID)

VALUES (emp\_seq.NEXTVAL, 'Omar', 'Said', 'M', 35, 'Nizwa', 'omar.said@company.com', 'pass123', 3, 3);



INSERT INTO EMPLOYEE (emp\_ID, fname, lname, gender, age, contact\_add, emp\_email, emp\_pass, job\_ID, salary\_ID)

VALUES (emp\_seq.NEXTVAL, 'Layla', 'Khalid', 'F', 26, 'Muscat', 'layla.khalid@company.com', 'pass123', 4, 4);



INSERT INTO EMPLOYEE (emp\_ID, fname, lname, gender, age, contact\_add, emp\_email, emp\_pass, job\_ID, salary\_ID)

VALUES (emp\_seq.NEXTVAL, 'Ahmed', 'Salim', 'M', 40, 'Sur', 'ahmed.salim@company.com', 'pass123', 5, 5);





INSERT INTO QUALIFICATION (qual\_ID, position, requirements, date\_in, emp\_ID)

VALUES (qual\_seq.NEXTVAL, 'Analyst', 'Bachelor’s Degree', SYSDATE, 1);



INSERT INTO QUALIFICATION (qual\_ID, position, requirements, date\_in, emp\_ID)

VALUES (qual\_seq.NEXTVAL, 'Accountant', 'CPA Certification', SYSDATE, 2);



INSERT INTO QUALIFICATION (qual\_ID, position, requirements, date\_in, emp\_ID)

VALUES (qual\_seq.NEXTVAL, 'Developer', 'Java Certification', SYSDATE, 3);



INSERT INTO QUALIFICATION (qual\_ID, position, requirements, date\_in, emp\_ID)

VALUES (qual\_seq.NEXTVAL, 'Marketer', 'Digital Marketing Diploma', SYSDATE, 4);



INSERT INTO QUALIFICATION (qual\_ID, position, requirements, date\_in, emp\_ID)

VALUES (qual\_seq.NEXTVAL, 'Manager', 'MBA Degree', SYSDATE, 5);







INSERT INTO LEAVE (leave\_ID, date\_from, reason, emp\_ID)

VALUES (leave\_seq.NEXTVAL, TO\_DATE('2026-06-01','YYYY-MM-DD'), 'Medical', 1);



INSERT INTO LEAVE (leave\_ID, date\_from, reason, emp\_ID)

VALUES (leave\_seq.NEXTVAL, TO\_DATE('2026-06-05','YYYY-MM-DD'), 'Vacation', 2);



INSERT INTO LEAVE (leave\_ID, date\_from, reason, emp\_ID)

VALUES (leave\_seq.NEXTVAL, TO\_DATE('2026-06-10','YYYY-MM-DD'), 'Family Emergency', 3);



INSERT INTO LEAVE (leave\_ID, date\_from, reason, emp\_ID)

VALUES (leave\_seq.NEXTVAL, TO\_DATE('2026-06-12','YYYY-MM-DD'), 'Training', 4);



INSERT INTO LEAVE (leave\_ID, date\_from, reason, emp\_ID)

VALUES (leave\_seq.NEXTVAL, TO\_DATE('2026-06-15','YYYY-MM-DD'), 'Conference', 5);







INSERT INTO PAYROLL (payroll\_ID, date\_paid, report, total\_amount, emp\_ID, job\_ID, salary\_ID, leave\_ID)

VALUES (payroll\_seq.NEXTVAL, SYSDATE, 'Monthly Payroll', 12500, 1, 1, 1, NULL);



INSERT INTO PAYROLL (payroll\_ID, date\_paid, report, total\_amount, emp\_ID, job\_ID, salary\_ID, leave\_ID)

VALUES (payroll\_seq.NEXTVAL, SYSDATE, 'Monthly Payroll', 18600, 2, 2, 2, 2);



INSERT INTO PAYROLL (payroll\_ID, date\_paid, report, total\_amount, emp\_ID, job\_ID, salary\_ID, leave\_ID)

VALUES (payroll\_seq.NEXTVAL, SYSDATE, 'Monthly Payroll', 24800, 3, 3, 3, NULL);



INSERT INTO PAYROLL (payroll\_ID, date\_paid, report, total\_amount, emp\_ID, job\_ID, salary\_ID, leave\_ID)

VALUES (payroll\_seq.NEXTVAL, SYSDATE, 'Monthly Payroll', 17200, 4, 4, 4, 4);



INSERT INTO PAYROLL (payroll\_ID, date\_paid, report, total\_amount, emp\_ID, job\_ID, salary\_ID, leave\_ID)

VALUES (payroll\_seq.NEXTVAL, SYSDATE, 'Monthly Payroll', 15900, 5, 5, 5, 5);











**-------------------------------------------------------------------------------**

#### **Task 2: Conditional SELECT Queries**



**Write SELECT queries for each of the following requirements:**

**(a) List all employees whose age is between 25 and 40, ordered by last name ascending.**

**(b) Retrieve all payroll records where total\_amount exceeds 5000, showing employee name and department.**

**(c) Find all employees who have taken leave with reason containing the word 'sick' (case-insensitive).**

**(d) List all departments that have no employees assigned. (Use outer join or NOT EXISTS.)**





\--A.

SELECT  emp\_ID, fname, lname , age

FROM EMPLOYEE

WHERE age BETWEEN 25 AND 40

ORDER BY lname ASC;





\--B.

SELECT p.payroll\_ID,

&#x20;      e.fname || ' ' || e.lname AS employee\_name,

&#x20;      d.job\_dept AS department,

&#x20;      p.total\_amount

FROM PAYROLL p

JOIN EMPLOYEE e ON p.emp\_ID = e.emp\_ID

JOIN JOB\_DEPARTMENT d ON p.job\_ID = d.job\_ID

WHERE p.total\_amount > 5000;





\--C.



SELECT DISTINCT e.emp\_ID, e.fname, e.lname, l.reason

FROM EMPLOYEE e

JOIN LEAVE l ON e.emp\_ID = l.emp\_ID

WHERE LOWER(l.reason) LIKE '%sick%';





\--D.



SELECT d.job\_ID, d.job\_dept, d.name

FROM JOB\_DEPARTMENT d

LEFT JOIN EMPLOYEE e ON d.job\_ID = e.job\_ID

WHERE e.emp\_ID IS NULL;







**----------------------------------------------------------------------------------**



#### **04 Aggregation Functions**



##### **Task 1: Basic Aggregation**



**Write queries using standard aggregate functions:**

**(a) Total number of employees in each department.**



SELECT d.job\_ID,

&#x20;       d.job\_dept,

&#x20;        COUNT(e.emp\_ID) AS total\_employees

FROM JOB\_DEPARTMENT d

LEFT JOIN EMPLOYEE e ON d.job\_ID = e.job\_ID

GROUP BY d.job\_ID, d.job\_dept

ORDER BY d.job\_dept;





**(b) Minimum, maximum, and average salary (amount) across all salary records.**





SELECT MIN(amount) AS min\_salary,

MAX(amount) AS max\_salary,

AVG(amount) AS avg\_salary

FROM SALARY\_BONUS;





**(c) Total bonus paid out across the entire company.**







SELECT SUM(bonus) AS total\_bonus

FROM SALARY\_BONUS;







##### **Task 2: GROUP BY with HAVING**



**Write GROUP BY queries with HAVING filters:**

**(a) List departments where the average employee age exceeds 30.**





SELECT d.job\_ID,

d.job\_dept,

AVG(e.age) AS avg\_age

FROM JOB\_DEPARTMENT d

JOIN EMPLOYEE e ON d.job\_ID = e.job\_ID

GROUP BY d.job\_ID, d.job\_dept

HAVING AVG(e.age) > 30;







**(b) Show all job titles where more than 2 employees share that qualification position.**





SELECT q.position,

COUNT(e.emp\_ID) AS num\_employees

FROM QUALIFICATION q

JOIN EMPLOYEE e ON q.emp\_ID = e.emp\_ID

GROUP BY q.position

HAVING COUNT(e.emp\_ID) >2;





**(c) Find months (from PAYROLL.date) where the total payroll amount exceeds 20,000**





**S**ELECT TO\_CHAR(p.date\_paid, 'YYYY-MM') AS payroll\_month,

SUM(p.total\_amount) AS monthy\_totla

FROM PAYROLL p

GROUP BY TO\_CHAR(p.date\_paid, 'YYYY-MM')

HAVING SUM(p.total\_amount) > 20000

ORDER BY payroll\_month;





**----------------------------------------------------------------------**

#### **05 Joins**

##### **Task 1: INNER JOIN — Employee Full Profile**



**Write a query using INNER JOINs to retrieve a complete employee profile:**

**Columns: emp\_ID, full name (fname || ' ' || lname), department name, job title (from QUALIFICATION), salary**

**amount, latest leave date.**

**Filter: only employees who have both a payroll record AND a qualification record.**





SELECT e.emp\_ID,

e.fname || ' ' || e.lname AS full\_name,

d.name AS department\_name,

q.position AS job\_title,

s.amount AS salary\_amount,

MAX(l.date\_from) AS latest\_leave\_date

FROM EMPLOYEE e

INNER JOIN JOB\_DEPARTMENT d ON e.job\_id = d.job\_ID

INNER JOIN QUALIFICATION q ON e.emp\_ID = q.emp\_ID

INNER JOIN SALARY\_BONUS s ON e.salary\_ID = s.salary\_ID

INNER JOIN PAYROLL p ON e.emp\_ID = p.emp\_ID

LEFT JOIN LEAVE l ON e.emp\_id = l.emp\_ID

GROUP BY e.emp\_ID, e.fname, e.lname , d.name, q.position , s.amount;









#### **Task 2: LEFT OUTER JOIN — Missing Records**



**Use LEFT OUTER JOINs to find gaps in data:**

**(a) List all employees who have never taken any leave (no matching LEAVE record).**





SELECT e.emp\_ID,

e.fname || ' ' || e.lname AS full\_name,

e.job\_ID

FROM EMPLOYEE e 

LEFT OUTER JOIN LEAVE l ON e.emp\_ID = l.emp\_ID

WHERE l.leave\_ID IS NULL;





**(b) List all departments that have no salary/bonus records associated with them.**



SELECT d.job\_ID,

d.job\_dept,

d.name

FROM JOB\_DEPARTMENT d

LEFT OUTER JOIN SALARY\_BONUS s ON d.job\_ID = s.job\_ID

WHERE s.salary\_ID IS NULL;







**-----------------------------------------------------------------------------**



### **06 Subqueries**



##### **Task 1: Single-Row Subquery**



**Write queries using single-row subqueries:**



**(a) Find all employees whose salary is greater than the average salary of the entire company.**





SELECT e.emp\_ID,

e.fname || ' ' || e.lname AS full\_name,

s.amount AS salary\_amount

FROM EMPLOYEE e 

JOIN SALARY\_BONUS s ON e.salary\_ID = s.salary\_ID

WHERE s.amount > (

SELECT AVG(amount)

FROM SALARY\_BONUS

)

ORDER BY s.amount DESC;





**(b) Retrieve the department with the highest total payroll amount**







SELECT d.job\_ID,

d.job\_dept,

d.name,

(SELECT SUM(p.total\_amount)

FROM PAYROLL p

WHERE p.job\_ID = d.job\_ID) AS dept\_totl

FROM JOB\_DEPARTMENT d

WHERE d.job\_ID = (

SELECT job\_ID

FROM PAYROLL 

GROUP BY job\_ID 

ORDER BY SUM(total\_amount) DESC

FETCH FIRST 1 ROW ONLY



);









##### **Task 2: Multi-Row Subquery with IN / ANY / ALL**



**(a) List all employees who work in departments that have at least one salary record with a bonus greater than 500.**

**Use IN.**



SELECT e.emp\_ID,

e.fname || ' ' || e.lname AS full\_name,

e.job\_ID

FROM EMPLOYEE e

WHERE e.job\_ID IN (

SELECT DISTINCT job\_ID

FROM SALARY\_BONUS

WHERE bonus > 500



);





**(b) Find employees whose salary is greater than ALL salaries in the 'Maintenance' department. Use ALL.**







SELECT e.emp\_ID,

e.fname || ' ' || e.lname AS full\_name,

s.amount AS salary\_amount

FROM EMPLOYEE e

JOIN SALARY\_BONUS s ON e.salary\_ID = s.salary\_ID

WHERE s.amount > ALL (

SELECT s2.amount

FROM EMPLOYEE e2

JOIN JOB\_DEPARTMENT d ON e2.job\_ID = d.job\_ID

JOIN SALARY\_BONUS s2 ON e2.salary\_ID = s2.salary\_ID

WHERE d.job\_dept = 'Maintenance'



);







**(c) Find employees whose salary is greater than ANY salary in the 'HR' department. Use ANY.**







SELECT e.emp\_ID,

e.fname || ' ' || e.lname AS full\_name,

s.amount AS salary\_amount

FROM EMPLOYEE e 

JOIN SALARY\_BONUS s ON e.salary\_ID = s.salary\_ID

WHERE s .amount > ANY (

SELECT s2.amount 

FROM EMPLOYEE e2

JOIN JOB\_DEPARTMENT d ON e2.job\_ID = D.JOB\_ID

JOIN SALARY\_BONUS s2 ON e2.salary\_ID = s2.salary\_ID

WHERE d.job\_dept = 'HR'



);











**---------------------------------------------------------------------------**



#### **07 Views**

##### **Task 1: Simple Read-Only View**



**Create a view named VW\_EMPLOYEE\_SUMMARY that returns:**



**emp\_ID, full name, gender, age, department name, and job title (from QUALIFICATION).**

**After creating it:**



**(a) Query the view to list all female employees over 30.**



**(b) Try to INSERT a row through this view and document the Oracle error message you receive.**



CREATE OR REPLACE VIEW VW\_EMPLOYEE\_SUMMARY AS

SELECT e.emp\_ID,

e.fname || ' ' || e.lname AS full\_name,

e.gender,

e.age,

d.name AS department\_name,

q.position AS job\_title

FROM EMPLOYEE e

JOIN JOB\_DEPARTMENT d ON e.job\_ID = d.job\_ID

JOIN QUALIFICATION q ON e.emp\_ID = q.emp\_ID;



\--



SELECT emp\_ID, full\_name , department\_name , job\_title

FROM VW\_EMPLOYEE\_SUMMARY

WHERE gender = 'F'

AND age > 30;



\--2.

INSERT INTO VW\_EMPLOYEE\_SUMMERY (emp\_ID , full\_name , gender , age , department\_name , job\_title)

VALUES(979,'Maha' , 'M' ,24 , 'Finance' , 'Analyst ');







More Details :

https://docs.oracle.com/error-help/db/ora-00942/



Error starting at line : 571 in command -

INSERT INTO VW\_EMPLOYEE\_SUMMARY (emp\_ID , full\_name , gender , age , department\_name , job\_title)

VALUES(979,'Maha' , 'F' ,24 , 'Finance' , 'Analyst ')

Error at Command Line : 571 Column : 34

Error report -

SQL Error: ORA-01779: cannot modify a column which maps to a non key-preserved table



https://docs.oracle.com/error-help/db/ora-01779/01779. 00000 -  "cannot modify a column which maps to a non key-preserved table"

\*Cause:    An attempt was made to insert or update columns of a join view which

&#x20;          map to a non-key-preserved table.

\*Action:   Modify the underlying base tables directly.



More Details :

https://docs.oracle.com/error-help/db/ora-01779/















##### **Task 2: Payroll Dashboard View**



**Create a view named VW\_PAYROLL\_DASHBOARD that returns for each payroll record:**

**payroll\_ID, employee full name, department, salary amount, bonus, leave reason, payroll date, and total\_amount.**

**Query the view to find the top 5 payroll records by total\_amount**





CREATE OR REPLACE VIEW VW\_PAYROLL\_DASHBOARD AS

SELECT p.payroll\_ID,

e.fname || ' ' || e.lname AS employee\_name,

d.name AS department\_name,

s.amount AS salary\_amount,

s.bonus,

l.reason AS leave\_reason,

p.date\_paid AS payroll\_date,

p.total\_amount

FROM PAYROLL p 

JOIN EMPLOYEE e ON p.emp\_ID = e.emp\_ID

JOIN JOB\_DEPARTMENT d ON p.job\_ID = d.job\_ID

JOIN SALARY\_BONUS s ON p.salary\_ID = s.salary\_ID

LEFT JOIN LEAVE l ON p.leave\_ID = l.leave\_ID;





SELECT payroll\_ID,

&#x20;employee\_name,

&#x20;department\_name,

&#x20;salary\_amount,

&#x20;bonus,

&#x20;leave\_reason,

&#x20;payroll\_date,

&#x20;total\_amount

FROM VW\_PAYROLL\_DASHBOARD

ORDER BY total\_amount DESC

FETCH FIRST 5 ROWS ONLY;







**----------------------------------------------------------------**

#### **08 Stored Procedures \& Functions**



###### **Task 1: Procedure — Add New Employee**



**Create a stored procedure named SP\_ADD\_EMPLOYEE with IN parameters for all EMPLOYEE columns (except**

**emp\_ID which is auto-generated from a sequence).**

**The procedure must:**

**• Validate that emp\_email is not already in use (raise an exception if duplicate).**

**• Insert the new employee record.**

**• Print a confirmation message using DBMS\_OUTPUT.PUT\_LINE.**

**Test by calling the procedure twice with the same email to confirm the exception fires**







CREATE TABLE SALARY\_BONUS (

&#x20;   salary\_ID   NUMBER PRIMARY KEY,

&#x20;   amount      NUMBER,

&#x20;   bonus       NUMBER,

&#x20;   annual      NUMBER NOT NULL

);



INSERT INTO SALARY\_BONUS (salary\_ID, amount, bonus, annual)

VALUES (222, 1500, 200, 1700);



SELECT job\_ID, name FROM JOB\_DEPARTMENT;



\--Task 1: Procedure

CREATE OR REPLACE PROCEDURE SP\_ADD\_EMPLOYEE (

p\_job\_ID        IN EMPLOYEE.job\_ID%TYPE,

p\_salary\_ID     IN EMPLOYEE.salary\_ID%TYPE,

p\_fname         IN EMPLOYEE.fname%TYPE,

p\_lname         IN EMPLOYEE.lname%TYPE,

p\_gender        IN EMPLOYEE.gender%TYPE,

p\_age           IN EMPLOYEE.age%TYPE,

p\_contact\_add   IN EMPLOYEE.contact\_add%TYPE,

p\_emp\_email     IN EMPLOYEE.emp\_email%TYPE,

p\_emp\_pass      IN EMPLOYEE.emp\_pass%TYPE



) AS

&#x20;   v\_count NUMBER;

&#x20;   

&#x20;   BEGIN

&#x20;   

&#x20;   SELECT COUNT(\*)

&#x20;   INTO v\_count

&#x20;   FROM EMPLOYEE

&#x20;   WHERE emp\_email = p\_emp\_email;

&#x20;   

&#x20;   

&#x20;   IF v\_count > 0 THEN

&#x20;   RAISE\_APPLICATION\_ERROR(-20001, 'Email already exists: ' || p\_emp\_email);

&#x20;   END IF;

&#x20;   

&#x20;   

&#x20;   INSERT INTO EMPLOYEE (

&#x20;   emp\_ID , job\_ID , salary\_ID, fname , lname , gender , age,contact\_add , emp\_email , emp\_pass 

&#x20;   )

&#x20;   VALUES (

&#x20;   emp\_seq.NEXTVAL , p\_job\_ID , p\_salary\_ID , p\_fname , p\_lname , p\_gender , p\_age , p\_contact\_add ,p\_emp\_email , p\_emp\_pass

&#x20;   );

&#x20;   

&#x20;   DBMS\_OUTPUT.PUT\_LINE('Employee ' || p\_fname || ' ' || p\_lname || ' added successfully.');

END;

/



BEGIN 

SP\_ADD\_EMPLOYEE(

p\_job\_ID        => 1,

p\_salary\_ID     => 222,

p\_fname         => 'Maha',

p\_lname         => 'Ali',

p\_gender        => 'F',

p\_age           => 28,

p\_contact\_add   => 'Muscat ,Oman',

p\_emp\_email       => 'Maha.ali@gmail.com',

p\_emp\_pass      => 'securepass'



);

END;

/





**------------------------------------**

BEGIN 

SP\_ADD\_EMPLOYEE(

p\_job\_ID        => 1,

p\_salary\_ID     => 222,

p\_fname         => 'Maha',

p\_lname         => 'Ali',

p\_gender        => 'F',

p\_age           => 29,

p\_contact\_add   => 'Muscat ,Oman',

p\_emp\_email       => 'Maha.ali@gmail.com',

p\_emp\_pass      => 'anotherpass'



);

END;

/



OUTPUT: ORA-20001: Email already exists: Maha.ali@gmail.com





###### Task 2: Function — Calculate Net Salary



**Create a function named FN\_NET\_SALARY that accepts emp\_ID as input and returns the employee's net salary**

**(amount + bonus) as a NUMBER.**

**Use it in a SELECT query to display all employees with their calculated net salary, ordered highest to lowest.**



CREATE OR REPLACE FUNCTION FN\_NET\_SALARY(

&#x20;p\_emp\_ID IN EMPLOYEE.emp\_ID%TYPE

) RETURN NUMBER

IS

&#x20;v\_net\_salary NUMBER;

&#x20;

BEGIN

SELECT s.amount + s.bonus

INTO v\_net\_salary

FROM SALARY\_BONUS s

JOIN EMPLOYEE e ON e.salary\_ID = s.salary\_ID

WHERE e.emp\_ID = p\_emp\_ID ;

RETURN v\_net\_salary;

EXCEPTION

&#x20;   WHEN NO\_DATA\_FOUND THEN

&#x20;       RETURN 0;

END;

/





SELECT  e.emp\_ID,

e.fname || ' ' || e.lname AS full\_name,

FN\_NET\_SALARY(e.emp\_ID) AS net\_salary

FROM EMPLOYEE e

ORDER BY net\_salary DESC;





\--------------------------------------------------------------------------------------

#### **09 Triggers**

##### **Task 1: BEFORE INSERT — Auto-Assign emp\_ID**



**Create a BEFORE INSERT trigger named TRG\_EMP\_ID on the EMPLOYEE table.**

**If the inserted emp\_ID is NULL, the trigger should populate it from a sequence (EMP\_SEQ.NEXTVAL).**

**Test: INSERT an employee without specifying emp\_ID and verify it was auto-assigned**





\-- TASK 1



CREATE OR REPLACE TRIGGER TRG\_EMP\_ID

BEFORE INSERT ON EMPLOYEE

FOR EACH ROW

BEGIN

&#x20;   IF :NEW.emp\_ID IS NULL THEN

&#x20;   SELECT emp\_seq.NEXTVAL

&#x20;   INTO :NEW.emp\_ID

&#x20;   FROM dual;

&#x20; END IF;

END;

/

&#x20; 



INSERT INTO EMPLOYEE( job\_ID, salary\_ID, fname, lname, gender, age,

&#x20;   contact\_add, emp\_email, emp\_pass

&#x20;   )

&#x20;   VALUES

&#x20;   (1 , 222 , 'Maha', 'Ali' , 'F' , 28,'Muscat ,Oman','Maha.ali@gmail.com','securepass'

&#x20;   );





SELECT emp\_ID , fname , lname, emp\_email

FROM EMPLOYEE

WHERE emp\_email = 'maha.auto2@gmail.com';







###### **Task 2: AFTER INSERT — Welcome Log**



**Create a table EMPLOYEE\_LOG(log\_id, emp\_id, action, log\_timestamp).**

**Create an AFTER INSERT trigger named TRG\_EMP\_WELCOME\_LOG on EMPLOYEE that inserts a log row with**

**action = 'NEW HIRE' whenever a new employee is added.**

**Test by inserting 2 employees and querying EMPLOYEE\_LOG**







CREATE TABLE EMPLOYEE\_LOG (

log\_id        NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,

emp\_id        NUMBER,

action        VARCHAR2(50),

log\_timestamp  DATE

);





CREATE OR REPLACE TRIGGER TRG\_EMP\_WELCOME\_LOG

AFTER INSERT ON EMPLOYEE

FOR EACH ROW

BEGIN

&#x20;INSERT INTO EMPLOYEE\_LOG (emp\_id , action , log\_timestamp)

&#x20;VALUES(:new.emp\_ID, 'NEW HIRE' , SYSDATE);

&#x20;END;

&#x20;/



INSERT INTO EMPLOYEE ( job\_ID, salary\_ID, fname, lname, gender, age, contact\_add, emp\_email, emp\_pass

) VALUES (1, 222, 'Sara', 'Khamees', 'F', 30, 'Muscat, Oman', 'sara.kha@gmail.com', 'pass123');





INSERT INTO EMPLOYEE ( job\_ID, salary\_ID, fname, lname, gender, age, contact\_add, emp\_email, emp\_pass

) VALUES (2, 333, 'Hameed', 'Salim', 'M', 35, 'Muscat, Oman', 'Hammed.Sal@gmail.com', 'pass456');



COMMIT;



SELECT log\_id , emp\_id , action, log\_timestamp

FROM EMPLOYEE\_LOG

ORDER BY log\_id;









**----------------------------------------------------------------------------------**





#### **10 Oracle Scheduler Jobs**

###### **Task 1: Simple One-Time Scheduler Job**

**Create a one-time DBMS\_SCHEDULER job named JOB\_GREET\_EMPLOYEES that runs 2 minutes from now**

**and executes an anonymous PL/SQL block that prints 'Payroll System Initialized' to DBMS\_OUTPUT and inserts a**

**record into EMPLOYEE\_LOG.**

**BEGIN**

**DBMS\_SCHEDULER.CREATE\_JOB(**

**job\_name => 'JOB\_GREET\_EMPLOYEES',**

**job\_type => 'PLSQL\_BLOCK',**

**job\_action => 'BEGIN ... END;',**

**start\_date => SYSTIMESTAMP + INTERVAL ''2'' MINUTE,**

**enabled => TRUE),**

**);**

**END;**

**After the job runs, query USER\_SCHEDULER\_JOB\_LOG to confirm execution status.**











BEGIN

&#x20;   DBMS\_SCHEDULER.CREATE\_JOB (

&#x20;       job\_name        => 'JOB\_GREET\_EMPLOYEES',

&#x20;       job\_type        => 'PLSQL\_BLOCK',

&#x20;       job\_action      => '

&#x20;           BEGIN

&#x20;               -- Print to DBMS\_OUTPUT

&#x20;               DBMS\_OUTPUT.PUT\_LINE(''Payroll System Initialized'');



&#x20;               -- Insert into EMPLOYEE\_LOG

&#x20;               INSERT INTO EMPLOYEE\_LOG (emp\_id, action, log\_timestamp)

&#x20;               VALUES (NULL, ''SYSTEM INIT'', SYSDATE);

&#x20;           END;',

&#x20;       start\_date      => SYSTIMESTAMP + INTERVAL '2' MINUTE,

&#x20;       enabled         => TRUE

&#x20;   );

END;

/







SELECT job\_name, status, log\_date, additional\_info

FROM USER\_SCHEDULER\_JOB\_LOG

WHERE job\_name = 'JOB\_GREET\_EMPLOYEES'

ORDER BY log\_date DESC;





###### **Task 2: Recurring Job — Daily Leave Report**



**Create a recurring DBMS\_SCHEDULER job named JOB\_DAILY\_LEAVE\_REPORT that:**

**• Runs every day at 07:00 AM.**

**• Calls the procedure SP\_PROCESS\_PAYROLL or simply counts leave records taken that day and inserts a**

**summary into EMPLOYEE\_LOG.**

**Use a repeat interval: FREQ=DAILY; BYHOUR=7; BYMINUTE=0; BYSECOND=0.**

**Show the job definition using: SELECT \* FROM USER\_SCHEDULER\_JOBS WHERE JOB\_NAME =**

**'JOB\_DAILY\_LEAVE\_REPORT';**







BEGIN

&#x20;   DBMS\_SCHEDULER.CREATE\_JOB (

&#x20;       job\_name        => 'JOB\_DAILY\_LEAVE\_REPORT',

&#x20;       job\_type        => 'PLSQL\_BLOCK',

&#x20;       job\_action      => '

&#x20;           BEGIN

&#x20;               -- Option 1: Call your payroll procedure

&#x20;               -- SP\_PROCESS\_PAYROLL;



&#x20;               -- Option 2: Count today''s leave records and log

&#x20;               DECLARE

&#x20;                   v\_leave\_count NUMBER;

&#x20;               BEGIN

&#x20;                   SELECT COUNT(\*)

&#x20;                   INTO v\_leave\_count

&#x20;                   FROM LEAVE

&#x20;                   WHERE TRUNC(leave\_date) = TRUNC(SYSDATE);



&#x20;                   INSERT INTO EMPLOYEE\_LOG (emp\_id, action, log\_timestamp)

&#x20;                   VALUES (NULL, ''LEAVE SUMMARY: '' || v\_leave\_count, SYSDATE);

&#x20;               END;

&#x20;           END;',

&#x20;       start\_date      => SYSTIMESTAMP,

&#x20;       repeat\_interval => 'FREQ=DAILY; BYHOUR=7; BYMINUTE=0; BYSECOND=0',

&#x20;       enabled         => TRUE

&#x20;   );

END;

/









SELECT job\_name, enabled, repeat\_interval, job\_type, job\_action

FROM USER\_SCHEDULER\_JOBS

WHERE job\_name = 'JOB\_DAILY\_LEAVE\_REPORT';



















