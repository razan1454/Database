&#x20;                               Enterprise Database Engineering Assignment



&#x20;                                          ( DAY 2)





Scenario

You have recently joined the Database Engineering Team of a large enterprise organization.

Management wants to evaluate your understanding of how enterprise systems use database objects beyond

simple tables and queries.

Your task is to investigate, analyze, design, and implement solutions using:

• Views

• Procedures

• Triggers

• Scheduler Jobs

You are expected to think like a Database Engineer, not only a SQL developer.



Day 1 – Enterprise Investigation \& Solution Design

Part 1 – Research \& Understanding



Research the following database objects:

Views



Investigate:

1\. What is a View?



A View is a virtual table defined by a stored SQL SELECT statement. It has no data of its own — every time you query it, the database runs the underlying query against the real base tables and returns a fresh result. From the user's or application's perspective, it looks and behaves exactly like a regular table.





2\. Why do companies use Views?



Simplicity — complex multi-table joins, aggregations, and business logic can be wrapped in a single named View. Developers query monthly\_revenue instead of writing a 40-line SQL statement every time.

Reusability — one View definition is shared across dozens of reports, applications, and analysts. If the logic changes, you update it in one place.

Abstraction / stability — when the underlying schema changes (e.g. a table is renamed or split), you update the View definition and all downstream applications keep working without modification.





3\. What security benefits do Views provide?



This is one of the most compelling reasons companies adopt Views:



Column-level access control — expose only the columns a user role needs. A emp\_directory View shows name and department, but never salary or ssn. Users are granted SELECT on the View, not the base table.



Row-level filtering — a View can include a WHERE clause that limits which rows a user sees (e.g. WHERE region = 'EMEA'), enforcing data segmentation at the database layer rather than relying on application code.



Minimal privilege — users never need direct access to base tables. If a table is compromised at the application layer, sensitive columns hidden behind a View are not exposed.



Audit surface reduction — fewer direct table grants means simpler, easier-to-audit permission structures.







4\. What is the difference between:



o Table

The foundation of every relational database. A table physically stores rows of data on disk. It has columns with defined data types, supports indexes for fast lookups, and accepts INSERT, UPDATE, and DELETE operations. Everything else — Views and Materialized Views — ultimately reads from tables.



o View

&#x20;

A View stores no data at all. It is purely a saved SQL query with a name. Every time you run SELECT \* FROM emp\_directory, the database executes the underlying query against the base tables at that moment and returns a live result. This means the data is always current, but for expensive queries (large joins, aggregations across millions of rows), it can be slow — because it does the full work every single time.



o Materialized View



A Materialized View solves the performance problem of a regular View by storing the query result as a physical snapshot. The first time it is built (or refreshed), the database runs the query and saves the output to disk — just like a table. Future queries hit that cached copy instantly, with no recomputation. The trade-off is staleness: the snapshot only reflects the data as it was at the last refresh. You can schedule refreshes (e.g. hourly, nightly) or trigger them manually, but between refreshes the data is out of date.



\------------------------------------------------------------------------------------------------------------------

Stored Procedures

Investigate:



1\. What is a Stored Procedure?



A stored procedure is a precompiled SQL code that can be saved and reused.



If you have an SQL query that you write over and over again, save it as a stored procedure, and then just call it to execute it.



A stored procedure can also have parameters, so it can act based on the parameter value(s) that is passed.



2\. Why do companies use Procedures?



Companies use stored procedures for six core reasons:



Reusability — a procedure is written once and called by any application, report, or script that needs it. A calculate\_invoice\_total procedure can be called from a web app, a mobile app, and a batch job without duplicating the logic.



Performance — when a stored procedure is first executed, the database engine compiles it and caches its execution plan. Subsequent calls skip the parsing and optimisation step entirely, making repeated execution significantly faster than sending raw SQL strings each time.



Security — users and applications can be granted permission to execute a procedure without being granted direct access to the underlying tables. This is the same principle as Views: the procedure acts as a controlled gateway. It also eliminates SQL injection risk, since parameters are handled separately from the query structure.



Centralised business logic — instead of scattering rules across multiple application codebases (web, mobile, desktop, APIs), critical logic like pricing rules, stock checks, or audit trails lives in one place inside the database. If a rule changes, you update the procedure once.





Reduced network traffic — instead of sending 10 separate SQL statements from the application to the database server (each one a round trip over the network), a single EXEC process\_order(@order\_id) call triggers all 10 statements on the server side. Only the final result travels back across the network.





Transaction control — procedures can wrap multiple operations in a single BEGIN TRANSACTION ... COMMIT / ROLLBACK block. If any step fails (e.g. deducting stock succeeds but recording the payment fails), the whole thing rolls back atomically, keeping data consistent.





3\. What problems do Procedures solve?



1\. Duplicated logic

Without procedures, the same SQL query gets copy-pasted into the web app, mobile app, reporting tool, and batch scripts. When the business rule changes, a developer has to hunt down and update every copy — and will almost certainly miss one, causing inconsistent behaviour. A procedure gives that logic a single home that everyone calls.



2\. Slow repeated queries

Every time raw SQL is sent to the database, the engine has to parse the text, validate it, and generate an execution plan before running it. With a stored procedure, this work happens once on first execution. The plan is cached and reused on every subsequent call, making high-frequency operations noticeably faster.



3\. SQL injection risk

When an application builds a query by concatenating user input — "SELECT \* FROM users WHERE name = '" + input + "'" — a malicious user can break out of the string and inject their own SQL. Procedures accept typed parameters that are never treated as executable code, eliminating this entire class of attack.



4\. Scattered business rules

In large organisations, the same rule (e.g. "a discount cannot exceed 30% for a standard customer") might be enforced in the web app, a mobile app, an admin tool, and a nightly job — each written differently. When the rule changes, every codebase needs updating. A procedure moves the rule into the database layer, where it applies universally regardless of which application is calling.



5\. Chatty network traffic

Processing an order might require checking stock, reserving inventory, writing the order, and updating a ledger — four or more separate SQL statements. If the application sends each one individually, that is four network round trips to the database server. A procedure executes all of them server-side and returns a single response, dramatically reducing latency, especially in cloud environments where the app and database are not on the same machine.



6\. Partial updates / data corruption

Without transaction control, if step 1 (deduct stock) succeeds and step 2 (record payment) fails due to an error, the database is left in a broken state — stock is gone but no payment was recorded. A procedure wraps both steps in a BEGIN TRANSACTION ... COMMIT / ROLLBACK block. If anything fails, the entire operation is rolled back as if nothing happened, preserving data integrity.







4\. What is the difference between:

o Procedure

o Function



Return value

A function must always return exactly one value — that is its entire purpose. A procedure does not have to return anything; if it does pass data back, it uses OUT parameters rather than a return value.



How they are called

A procedure is called as a standalone statement: EXEC process\_order(1042). A function is called from inside another SQL statement: SELECT calculate\_tax(price) FROM orders — it slots in wherever a value is expected, just like a column name or a number.



Usable in SELECT

Because a function returns a value, the query engine can treat it like any expression. You can use it in SELECT, WHERE, JOIN ON, and ORDER BY clauses. A procedure cannot be used this way.

5\. Give at least three enterprise use cases.



Transactions

Procedures can contain full transaction logic — BEGIN TRANSACTION, COMMIT, and ROLLBACK. Functions in most databases (SQL Server, PostgreSQL, Oracle) cannot manage transactions, because they are designed to be pure computations that can be called safely inside any query.





Can modify data?

Procedures can freely run INSERT, UPDATE, and DELETE statements — that is often their main job. Functions are typically restricted to read-only operations; most databases prevent a function from having side effects so that calling it inside a SELECT does not unpredictably alter data.







5\. Give at least three enterprise use cases.



Procedure:                                           Function:

Banking Finance                                      Retail E-commerace

Healthcare                                           insurance Credit

Any large enterprise







\------------------------------------------------------------------------------------------------------------------

Scheduler Jobs

Investigate:



1\. What is a Scheduler Job?



(also called a scheduled job, database job, or agent job) is a pre-defined task that the database runs automatically at a set time or in response to a trigger — with no human needing to kick it off.



The three parts of every scheduler job:



1\. The trigger — what fires it

A job must have a reason to start. The most common trigger is time-based: "run every night at 2 AM", "run every Monday at 6 AM", or "run every 15 minutes". Jobs can also be event-driven (fire when a table reaches a certain row count, or when an alert condition is met) or triggered on-demand by an operator or API call.



2\. The agent — what manages it

Every major database platform has a built-in job scheduling engine. SQL Server uses SQL Server Agent, PostgreSQL uses pgAgent, Oracle has its own DBMS\_SCHEDULER, and MySQL uses the Event Scheduler. The agent is the background service that watches the clock, fires jobs at the right moment, and monitors whether they succeed or fail.



3\. The task — what it actually runs

The job executes something: a stored procedure, a plain SQL script, an operating system command, or a data pipeline package (like an SSIS package in SQL Server). The task can be a single step or a chain of steps run in sequence.



2\. Why do companies use Scheduler Jobs?



Companies use scheduler jobs to automate repetitive, time-sensitive, or resource-heavy tasks, ensuring reliability and efficiency. The key difference is that a trigger reacts to an event, while a scheduler job runs based on a defined schedule. Together, they enable enterprises to automate critical processes like backups, reporting, and data integration.



Automation efficiency :reduces manual intervention and human error.

Resource optimization: jobs can be prioritized and executed during off-peak hours.

Reliability: Ensure tasks like backups or ETL pipelines run consistently.

Scalability : Handles large workloads across distributed systems.

Compliance : Automates audit logs, reporting, and regulatory checks.





3\. What is the difference between:

o Trigger

o Scheduler Job



Difference Between Trigger and Scheduler Job



Concept 	Tigger	Scheduler Job

Definition	Event-based automation mechanism.	Time-based or recurring automation mechanism.

Execution	Fires immediately when a condition/event occurs (e.g., file arrival, record insert).	Executes at predefined times or intervals.

Use Case	Dynamic, unpredictable events.	Predictable, routine tasks.

Examples	Run job when a file is uploaded, or when a transaction completes.	Daily backups, weekly reports, monthly billing.











4\. What processes are commonly automated?



Database backup and archiving.

ETL pipelines(Extract , Transform ,load) for data warehouses.

Batch processing of invoices, payroll, or orders.

System monitoring and log cleanup.

Report generation for finance, HR, or compliance.

File transfers between systems.





5\. Give at least three enterprise use cases.





1.Financial services: Automating end-of-day reconciliation, fraud detection triggers, and compliance reporting.



2.Retail and e-commerce: Inventory updates, order fulfillment batch jobs, and promotional pricing rollouts.



3.Telecommunications: Automating billing cycles, customer usage reports, and network monitoring alerts.



4.Healthcare: Patient record synchronization, lab result notifications, and compliance audits.



5.Manufacturing: Production scheduling, supply chain updates, and predictive maintenance jobs.





\----------------------------------------------------------------------------------------------------------------------



Part 2 – Enterprise Decision Making

For each scenario:

1\. Identify which object should be used:

o View

o Procedure

o Trigger

o Scheduler Job

2\. Explain your reasoning.



Scenario 1

The HR department should only see:

• Employee Name

• Department Name

They should not see salary information.



Object: View.

reason:

&#x20;a view can selected columns employee\_name,department\_name which help to hide the sensitive data like salary.





Scenario 2

Every salary update must automatically be recorded for auditing purposes.



Object: Tigger.

reason:

automatically fires whenever a salary update occurs.





Scenario 3

Management wants a report generated automatically every Friday at 4:00 PM.



Object: Scheduler job.

reason:

runs at a fixed time every Friday at 4:00 PM.





Scenario 4

The Finance department wants one reusable process that calculates annual bonuses.



Object: Procedure.

reason:

A procedure encapsulates reusable business logic (bonus calculation).







Scenario 5

The company wants a notification whenever an employee's salary is modified.



Object: Trigger.

reason:

can send notifications or insert records into a monitoring table.





Scenario 6

Management wants a dashboard displaying employee information without exposing underlying tables.



Object: View.

reason:

a secure abstraction layer for dashboards.



\----------------------------------------------------------------------------------------------------------------------

**Part 3 – Architecture Design**









&#x20;         HR Users  ----------------------->  Procedures  ------------------------> Dashboard View

&#x20;                  access employees data    (Bonus Calculation)                access employee data

&#x20;                                                  |

&#x20;                                                  |

&#x20;                                                  |

&#x20;                                                  |



&#x20;                        query data                               log change

&#x20;             Views < ------------------------ DATABASE --------------------------->Triggers

&#x20;       Employee info view                        **|**                          Salary change trigger

&#x20;          |           |                          |                                  |

employee table       Department table             |                             audit table

&#x20;                                                 |

&#x20;                                             Processes

&#x20;                                                 |       generate reports

&#x20;                                           Scheduler job----------------->reports





\----------------------------------------------------------------------------------------------------------------------

**Part 4 – Reflection**



**If applications can perform all business logic themselves, why do enterprise systems still place logic inside the**

**database?**



modern applications can handle business logic in their own code, enterprises still embed logic inside the database for reasons of consistency, performance, security, and maintainability. Let’s break this down using the four key database objects.





|**Database Object**|**Purpose in Enterprise Systems**|**Why It’s Still Used**|
|-|-|-|
|**Views**|Present filtered or aggregated data|Centralize data access rules, enforce security, and simplify queries for multiple applications.|
|**Procedures**|Encapsulate reusable business logic|Ensure consistent calculations (e.g., payroll, bonuses) across all applications without duplicating logic.|
|**Triggers**|React automatically to data changes|Guarantee audit trails and enforce integrity even if applications bypass validation.|
|**Scheduler Jobs**|Automate time-based tasks|Handle recurring operations (reports, backups) independently of application uptime.|





**-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------**

&#x20;                                                                                                  **Day 2 – Oracle Implementation**

**Part 5 – View Implementation**

**Task 1**

**Create a View displaying:**

**• Employee ID**

**• Employee Name**

**• Department ID**

**• Salary**







CREATE OR REPLACE VIEW EMP\_BASIC\_INFO AS

SELECT EMPLOYEE\_ID,

&#x20;      FIRST\_NAME || ' ' || LAST\_NAME   AS Employee\_Name,

&#x20;      DEPARTMENT\_ID,

&#x20;      SALARY

FROM EMPLOYEES ;



\--QUERY THE VIEW

SELECT \* FROM EMP\_BASIC\_INFO

ORDER BY EMPLOYEE\_ID;



\--VERIFY VIEW WAS CREATED

SELECT VIEW\_NAME,

&#x20;      TEXT\_LENGTH

FROM USER\_VIEWS

WHERE VIEW\_NAME = 'EMP\_BASIC\_INFO';







**--------------------------------------------------------**

**Task 2**

**Create a View displaying:**

**• Employee Name**

**• Department Name**

**• Job Title**

**• Salary**



CREATE OR REPLACE VIEW EMP\_DETAILS\_VIEW AS

SELECT E.EMPLOYEE\_ID,

&#x20;      E.FIRST\_NAME || ' ' || E.LAST\_NAME   AS Employee\_Name,

&#x20;      D.DEPARTMENT\_NAME,

&#x20;      J.JOB\_TITLE,

&#x20;      E.SALARY

FROM EMPLOYEES   E

JOIN DEPARTMENTS D ON E.DEPARTMENT\_ID = D.DEPARTMENT\_ID

JOIN JOBS        J ON E.JOB\_ID        = J.JOB\_ID;



SELECT \* FROM EMP\_DETAILS\_VIEW

ORDER BY DEPARTMENT\_NAME,

SALARY DESC;





**-------------------------------------------------------**



**Task 3**

**Using the View created above:**

**Display employees earning more than 10000.**





SELECT \* FROM EMP\_DETAILS\_VIEW

WHERE SALARY > 10000

ORDER BY SALARY DESC;



\-----------------------------------------------

**Task 4**

**Explain:**

**Why would management prefer querying the View instead of querying EMPLOYEES directly?**



**1.data security**

view can restrict access to sensitive columns. this ensure managers only see what they are autjorized to see.



**2. simplified queries**

view encapsulate complex join, filter, or aggregations.



**3.data  abstraction**

hide the underlying business schema complexity. If the EMPLOYEES table changes (new columns, renamed fields), the View can be updated without requiring managers to rewrite their queries.



**4.business logic consistency**

enforce standardized business rules.



**5.performance optimization**

Views (especially indexed views) can improve query performance by precomputing joins or aggregations.



Querying the EMPLOYEES table directly exposes raw data, which may be too detailed, inconsistent, or sensitive. Views act as a business-friendly lens: they protect sensitive information, simplify access, and ensure managers always see data aligned with company policies.





**-----------------------------------------------------**

**Part 6 – Procedure Implementation**



**Task 5**



**Create a Procedure that accepts:**

**• Department ID**



**and displays:**

**• Employee Name**

**• Salary**



**for employees belonging to that department.**





CREATE OR REPLACE PROCEDURE GET\_EMP\_BY\_DEPT

&#x20;   (P\_DEPT\_ID IN  NUMBER)

&#x20;   IS

&#x20;   CURSOR EMP\_CUR IS

&#x20;       SELECT FIRST\_NAME || ' ' || LAST\_NAME AS Emp\_Name,

&#x20;              SALARY

&#x20;       FROM EMPLOYEES

&#x20;       WHERE DEPARTMENT\_ID = P\_DEPT\_ID;

&#x20;

&#x20;BEGIN

&#x20;      DBMS\_OUTPUT.PUT\_LINE('Department ID:'  || P\_DEPT\_ID);

&#x20;      DBMS\_OUTPUT.PUT\_LINE('----------------------------');



&#x20;      FOR REC IN EMP\_CUR LOOP

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Name  : ' || REC.Emp\_Name);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Salary: ' || REC.SALARY);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('-------------------------');

&#x20;   END LOOP;

END;

/



SET SERVEROUTPUT ON;

EXEC GET\_EMP\_BY\_DEPT(90);



**---------------------------------------------------**

**Task 6**

**Modify the Procedure so it only returns employees whose salary is above the department average salary.**



CREATE OR REPLACE PROCEDURE GET\_EMP\_BY\_DEBT

&#x20;  (P\_DEPT\_ID IN NUMBER)

IS

BEGIN

&#x20;   DBMS\_OUTPUT.PUT\_LINE('Department ID:' || P\_DEPT\_ID);

&#x20;   DBMS\_OUTPUT.PUT\_LINE('EMPLOYEES EARNING ABOVE DEPARTMENT AVERAGE');

&#x20;   DBMS\_OUTPUT.PUT\_LINE('--------------------------------------------');



&#x20;     FOR REC IN (

&#x20;     SELECT FIRST\_NAME || ' ' || LAST\_NAME AS Emp\_Name,

&#x20;           SALARY

&#x20;     FROM EMPLOYEES e

&#x20;     WHERE e.DEPARTMENT\_ID = P\_DEPT\_ID

&#x20;     AND e.SALARY > (

&#x20;           SELECT AVG(SALARY)

&#x20;           FROM EMPLOYEES

&#x20;           WHERE DEPARTMENT\_ID = P\_DEPT\_ID

&#x20;           )

&#x20;        )

&#x20;

&#x20;        LOOP

&#x20;   DBMS\_OUTPUT.PUT\_LINE('Name : ' || REC.Emp\_Name);

&#x20;   DBMS\_OUTPUT.PUT\_LINE('Salary: ' || REC.Emp\_Name);

&#x20;   DBMS\_OUTPUT.PUT\_LINE('--------------------------------------------');

&#x20;  END LOOP;

&#x20;

&#x20;  END;

&#x20;  /



SET SERVEROUTPUT ON;

EXEC GET\_EMP\_BY\_DEPT(90);





\----------------------------------------------------------------------------

**Task 7**

**Explain:**

**Why is a Procedure better than rewriting the query repeatedly?**



because rewriting queries repeated leads to duplicate, inconsistency, and maintenance headaches. The procedures centralize the logic, making systems more robust, secure and efficient.



\------------------------------------------------------------------------

**Part 7**



**– Trigger Implementation**

**Management wants salary changes audited.**



**Task 8**

**Create a table:**

**SALARY\_AUDIT**

**containing:**

**• Employee ID**

**• Old Salary**

**• New Salary**

**• Update Date**



CREATE TABLE SALARY\_AUDIT(

&#x20;   EMPLOYEE\_ID     NUMBER(6,0)  NOT NULL,

&#x20;   OLD\_SALARY      NUMBER(8,2),

&#x20;   NEW\_SALARY      NUMBER(8,2),

&#x20;   UPDATE\_DATE     DATE          DEFAULT SYSDATE NOT NULL,

&#x20;   CONSTRAINT PK\_SALARY\_AUDIT PRIMARY KEY (EMPLOYEE\_ID, UPDATE\_DATE)

&#x20;   );



CREATE OR REPLACE TRIGGER TRG\_SALARY\_AUDIT

AFTER UPDATE OF SALARY ON EMPLOYEES

FOR EACH ROW

BEGIN

&#x20;    INSERT INTO SALARY\_AUDIT(

&#x20;    EMPLOYEE\_ID,

&#x20;    OLD\_SALARY,

&#x20;    NEW\_SALARY,

&#x20;    UPDATE\_DATE

&#x20;    )

&#x20;    VALUES(

&#x20;      :OLD.EMPLOYEE\_ID,

&#x20;       :OLD.SALARY,

&#x20;       :NEW.SALARY,

&#x20;       SYSDATE

&#x20;   );

END;

/



UPDATE EMPLOYEES

SET SALARY = SALARY \* 1.30

WHERE EMPLOYEE\_ID = 101;





SELECT EMPLOYEE\_ID,

&#x20;      OLD\_SALARY,

&#x20;      NEW\_SALARY,

&#x20;      UPDATE\_DATE

FROM SALARY\_AUDIT

ORDER BY UPDATE\_DATE DESC;



**----------------------------------------------------------------**



**Task 9**

**Create a Trigger that automatically records salary changes.**





CREATE OR REPLACE TRIGGER TRG\_SALARY\_AUDIT

AFTER UPDATE OF SALARY ON EMPLOYEES

FOR EACH ROW

BEGIN

&#x20;   INSERT INTO SALARY\_AUDIT(

&#x20;      EMPLOYEE\_ID,

&#x20;      OLD\_SALARY,

&#x20;      NEW\_SALARY,

&#x20;      UPDATE\_DATE

&#x20;      )

&#x20;

&#x20;      VALUES(

&#x20;      :OLD.EMPLOYEE\_ID,

&#x20;      :OLD.SALARY,

&#x20;      :NEW.SALARY,

&#x20;       SYSDATE

&#x20;       );

&#x20;       END;

&#x20;       /

&#x20;



&#x20;UPDATE EMPLOYEES

SET SALARY = SALARY \* 1.30

WHERE EMPLOYEE\_ID = 101;







SELECT \* FROM SALARY\_AUDIT

ORDER BY UPDATE\_DATE DESC;







**------------------------------------------------------------------------**



**Task 10**

**Test the Trigger.**

**Update employee salaries and verify audit records are generated correctly.**





&#x20;UPDATE EMPLOYEES

SET SALARY = SALARY \* 1.30

WHERE EMPLOYEE\_ID = 101;





&#x20;UPDATE EMPLOYEES

SET SALARY = SALARY \*130

WHERE EMPLOYEE\_ID = 105;





SELECT EMPLOYEE\_ID,

&#x20;      OLD\_SALARY,

&#x20;      NEW\_SALARY,

&#x20;      UPDATE\_DATE

FROM SALARY\_AUDIT

ORDER BY UPDATE\_DATE DESC;







**--------------------------------------------------------------------------------**

**Task 11**

**Enhance the Trigger to also store:**

**• Username**

**• Timestamp**



ALTER TABLE SALARY\_AUDIT

ADD(

&#x20;  USERNAME        VARCHAR2(30),

&#x20;  UPDATE\_TIMESTAMP TIMESTAMP DEFAULT SYSTIMESTAMP

);





CREATE OR REPLACE TRIGGER TRG\_SALARY\_AUDIT

AFTER UPDATE OF SALARY ON EMPLOYEES

FOR EACH ROW

BEGIN

&#x20;   INSERT INTO SALARY\_AUDIT (

&#x20;       EMPLOYEE\_ID,

&#x20;       OLD\_SALARY,

&#x20;       NEW\_SALARY,

&#x20;       UPDATE\_DATE,

&#x20;       USERNAME,

&#x20;       UPDATE\_TIMESTAMP

&#x20;   )

&#x20;   VALUES (

&#x20;       :OLD.EMPLOYEE\_ID,

&#x20;       :OLD.SALARY,

&#x20;       :NEW.SALARY,

&#x20;       SYSDATE,

&#x20;       USER,

&#x20;       SYSTIMESTAMP

&#x20;   );

END;

/





**-----------------------------------------------------------------------------**

**Part 8 – Scheduler Job Design**

**You are not required to fully configure Oracle Scheduler.**

**Instead, design Scheduler Jobs for the following business requirements.**



**Task 12**

**Every day at 8:00 PM:**

**Generate a report identifying employees whose salaries exceed their department average salary.**

**Provide:**

**• Job Name**

**• Schedule**

**• Logic Executed**





CREATE OR REPLACE PROCEDURE REPORT\_EMP\_ABOVE\_AVG IS

BEGIN

&#x20;   DBMS\_OUTPUT.PUT\_LINE('Employees earning above department average');

&#x20;   DBMS\_OUTPUT.PUT\_LINE('-------------------------------------------');



&#x20;   FOR REC IN (

&#x20;       SELECT e.EMPLOYEE\_ID,

&#x20;              e.FIRST\_NAME || ' ' || e.LAST\_NAME AS Emp\_Name,

&#x20;              e.SALARY,

&#x20;              e.DEPARTMENT\_ID

&#x20;       FROM EMPLOYEES e

&#x20;       WHERE e.SALARY > (

&#x20;           SELECT AVG(SALARY)

&#x20;           FROM EMPLOYEES

&#x20;           WHERE DEPARTMENT\_ID = e.DEPARTMENT\_ID

&#x20;       )

&#x20;   )

&#x20;   LOOP

&#x20;       DBMS\_OUTPUT.PUT\_LINE('ID     : ' || REC.EMPLOYEE\_ID);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Name   : ' || REC.Emp\_Name);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Salary : ' || REC.SALARY);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Dept   : ' || REC.DEPARTMENT\_ID);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('-------------------------------------------');

&#x20;   END LOOP;

END;

/



BEGIN

&#x20;   DBMS\_SCHEDULER.CREATE\_JOB (

&#x20;       job\_name        => 'JOB\_EMP\_ABOVE\_AVG',

&#x20;       job\_type        => 'STORED\_PROCEDURE',

&#x20;       job\_action      => 'REPORT\_EMP\_ABOVE\_AVG',

&#x20;       start\_date      => SYSTIMESTAMP,

&#x20;       repeat\_interval => 'FREQ=DAILY; BYHOUR=20; BYMINUTE=0; BYSECOND=0',

&#x20;       enabled         => TRUE

&#x20;   );

END;

/



**-------------------------------------------------------------------------**

**Task 13**

**Every Friday at 4:00 PM:**

**Generate a report listing departments with the highest average salaries.**

**Provide:**

**• Job Name**

**• Schedule**

**• Logic Executed**



CREATE OR REPLACE PROCEDURE REPORT\_DEPT\_HIGH\_AVG\_SAL IS

BEGIN

&#x20;   DBMS\_OUTPUT.PUT\_LINE('Departments with Highest Average Salaries');

&#x20;   DBMS\_OUTPUT.PUT\_LINE('------------------------------------------');



&#x20;   FOR REC IN (

&#x20;       SELECT d.DEPARTMENT\_ID,

&#x20;              d.DEPARTMENT\_NAME,

&#x20;              AVG(e.SALARY) AS Avg\_Salary

&#x20;       FROM EMPLOYEES e

&#x20;       JOIN DEPARTMENTS d ON e.DEPARTMENT\_ID = d.DEPARTMENT\_ID

&#x20;       GROUP BY d.DEPARTMENT\_ID, d.DEPARTMENT\_NAME

&#x20;       ORDER BY Avg\_Salary DESC

&#x20;   )

&#x20;   LOOP

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Dept ID   : ' || REC.DEPARTMENT\_ID);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Dept Name : ' || REC.DEPARTMENT\_NAME);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Avg Salary: ' || REC.Avg\_Salary);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('------------------------------------------');

&#x20;   END LOOP;

END;

/



BEGIN

&#x20;   DBMS\_SCHEDULER.CREATE\_JOB (

&#x20;       job\_name        => 'JOB\_DEPT\_HIGH\_AVG\_SAL',

&#x20;       job\_type        => 'STORED\_PROCEDURE',

&#x20;       job\_action      => 'REPORT\_DEPT\_HIGH\_AVG\_SAL',

&#x20;       start\_date      => SYSTIMESTAMP,

&#x20;       repeat\_interval => 'FREQ=WEEKLY; BYDAY=FRI; BYHOUR=16; BYMINUTE=0; BYSECOND=0',

&#x20;       enabled         => TRUE

&#x20;   );

END;

/





\---------------------------------------------------------

**Task 14**

**Every month:**

**Generate a report identifying employees who have not received salary updates during the last six months.**

**Provide:**

**• Job Name**

**• Schedule**

**• Logic Executed**



CREATE OR REPLACE PROCEDURE REPORT\_EMP\_NO\_UPDATE\_6M IS

BEGIN

&#x20;   DBMS\_OUTPUT.PUT\_LINE('Employees with no salary update in last 6 months');

&#x20;   DBMS\_OUTPUT.PUT\_LINE('------------------------------------------------');



&#x20;   FOR REC IN (

&#x20;       SELECT e.EMPLOYEE\_ID,

&#x20;              e.FIRST\_NAME || ' ' || e.LAST\_NAME AS Emp\_Name,

&#x20;              e.SALARY,

&#x20;              e.DEPARTMENT\_ID

&#x20;       FROM EMPLOYEES e

&#x20;       WHERE NOT EXISTS (

&#x20;           SELECT 1

&#x20;           FROM SALARY\_AUDIT sa

&#x20;           WHERE sa.EMPLOYEE\_ID = e.EMPLOYEE\_ID

&#x20;             AND sa.UPDATE\_DATE >= ADD\_MONTHS(SYSDATE, -6)

&#x20;       )

&#x20;   )

&#x20;   LOOP

&#x20;       DBMS\_OUTPUT.PUT\_LINE('ID     : ' || REC.EMPLOYEE\_ID);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Name   : ' || REC.Emp\_Name);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Salary : ' || REC.SALARY);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Dept   : ' || REC.DEPARTMENT\_ID);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('------------------------------------------------');

&#x20;   END LOOP;

END;

/



BEGIN

&#x20;   DBMS\_SCHEDULER.CREATE\_JOB (

&#x20;       job\_name        => 'JOB\_EMP\_NO\_UPDATE\_6M',

&#x20;       job\_type        => 'STORED\_PROCEDURE',

&#x20;       job\_action      => 'REPORT\_EMP\_NO\_UPDATE\_6M',

&#x20;       start\_date      => SYSTIMESTAMP,

&#x20;       repeat\_interval => 'FREQ=MONTHLY; BYHOUR=20; BYMINUTE=0; BYSECOND=0',

&#x20;       enabled         => TRUE

&#x20;   );

END;

/





**-------------------------------------------------------------------------**

**Part 9 – Enterprise Change Request**



**Management has changed requirements.**

**Modify your design accordingly.**



**Change Request 1**

**The HR View must now include:**

**• Job Title**

**• Department Name**

**• City**

**without exposing underlying tables.**

**Explain how you would modify the View.**



CREATE OR REPLACE VIEW HR\_EMPLOYEE\_VIEW AS

SELECT e.EMPLOYEE\_ID,

&#x20;      e.FIRST\_NAME || ' ' || e.LAST\_NAME AS Employee\_Name,

&#x20;      j.JOB\_TITLE,

&#x20;      d.DEPARTMENT\_NAME,

&#x20;      l.CITY

FROM EMPLOYEES e

JOIN JOBS j

&#x20; ON e.JOB\_ID = j.JOB\_ID

JOIN DEPARTMENTS d

&#x20; ON e.DEPARTMENT\_ID = d.DEPARTMENT\_ID

JOIN LOCATIONS l

&#x20; ON d.LOCATION\_ID = l.LOCATION\_ID;



Data Security → HR users don’t see sensitive columns like salary or hire date.



Abstraction → Underlying table complexity is hidden.



Business-Friendly → HR queries a single view instead of writing complex joins.



Consistency → Ensures all HR reports use the same standardized dataset.

\-----------------------------------------------------------------------------



**Change Request 2**

**The Procedure must now return:**

**• employees earning above department average**

**• ordered by highest salary**

**Modify your design.**



CREATE OR REPLACE PROCEDURE REPORT\_EMP\_ABOVE\_AVG IS

BEGIN

&#x20;   DBMS\_OUTPUT.PUT\_LINE('Employees earning above department average (ordered by highest salary)');

&#x20;   DBMS\_OUTPUT.PUT\_LINE('-------------------------------------------------------------');



&#x20;   FOR REC IN (

&#x20;       SELECT e.EMPLOYEE\_ID,

&#x20;              e.FIRST\_NAME || ' ' || e.LAST\_NAME AS Emp\_Name,

&#x20;              e.SALARY,

&#x20;              e.DEPARTMENT\_ID

&#x20;       FROM EMPLOYEES e

&#x20;       WHERE e.SALARY > (

&#x20;           SELECT AVG(SALARY)

&#x20;           FROM EMPLOYEES

&#x20;           WHERE DEPARTMENT\_ID = e.DEPARTMENT\_ID

&#x20;       )

&#x20;       ORDER BY e.SALARY DESC

&#x20;   )

&#x20;   LOOP

&#x20;       DBMS\_OUTPUT.PUT\_LINE('ID     : ' || REC.EMPLOYEE\_ID);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Name   : ' || REC.Emp\_Name);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Salary : ' || REC.SALARY);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('Dept   : ' || REC.DEPARTMENT\_ID);

&#x20;       DBMS\_OUTPUT.PUT\_LINE('-------------------------------------------------------------');

&#x20;   END LOOP;

END;

/



&#x20;With this modification, the procedure now produces a ranked list of top earners above average, making it easier for HR and management to spot the highest-paid employees relative to their peers.



\-------------------------------------------------------------------------------



**Change Request 3**

**The Trigger must now record:**

**• Username**

**• Timestamp**

**• Department ID**

**Modify your design.**



SELECT TRIGGER\_NAME,

&#x20;      STATUS,

&#x20;      TRIGGER\_TYPE

FROM   USER\_TRIGGERS

WHERE  TRIGGER\_NAME = 'TRG\_SALARY\_AUDIT';



SET SERVEROUTPUT ON;



SELECT EMPLOYEE\_ID,

&#x20;      FIRST\_NAME,

&#x20;      SALARY

FROM   EMPLOYEES

WHERE  EMPLOYEE\_ID = 100;



UPDATE EMPLOYEES

SET    SALARY = 25000

WHERE  EMPLOYEE\_ID = 100;

COMMIT;





SELECT \* FROM SALARY\_AUDIT;



DROP TRIGGER TRG\_SALARY\_AUDIT;



CREATE OR REPLACE TRIGGER TRG\_SALARY\_AUDIT

AFTER UPDATE OF SALARY ON EMPLOYEES

FOR EACH ROW

BEGIN

&#x20;   INSERT INTO SALARY\_AUDIT (

&#x20;       EMPLOYEE\_ID,

&#x20;       OLD\_SALARY,

&#x20;       NEW\_SALARY,

&#x20;       CHANGE\_AMOUNT,

&#x20;       UPDATE\_DATE,

&#x20;       USERNAME,

&#x20;       UPDATE\_TIMESTAMP

&#x20;   )

&#x20;   VALUES (

&#x20;       :OLD.EMPLOYEE\_ID,

&#x20;       :OLD.SALARY,

&#x20;       :NEW.SALARY,

&#x20;       :NEW.SALARY - :OLD.SALARY,

&#x20;       SYSDATE,

&#x20;       SYS\_CONTEXT('USERENV','SESSION\_USER'), -- fix USERNAME

&#x20;       SYSTIMESTAMP

&#x20;   );

END;

/



DELETE FROM SALARY\_AUDIT;

COMMIT;



SELECT EMPLOYEE\_ID,

&#x20;      FIRST\_NAME,

&#x20;      SALARY

FROM   EMPLOYEES

WHERE  EMPLOYEE\_ID IN (100, 101, 105)

ORDER BY EMPLOYEE\_ID;



UPDATE EMPLOYEES

SET    SALARY = 25000

WHERE  EMPLOYEE\_ID = 100;

COMMIT;



SELECT EMPLOYEE\_ID,

&#x20;      OLD\_SALARY,

&#x20;      NEW\_SALARY,

&#x20;      CHANGE\_AMOUNT,

&#x20;      UPDATE\_DATE,

&#x20;      USERNAME,

&#x20;      UPDATE\_TIMESTAMP

FROM   SALARY\_AUDIT

ORDER BY AUDIT\_ID;



SELECT COLUMN\_NAME,

&#x20;      DATA\_TYPE

FROM   USER\_TAB\_COLUMNS

WHERE  TABLE\_NAME = 'SALARY\_AUDIT'

ORDER BY COLUMN\_ID;





DROP TABLE SALARY\_AUDIT;



CREATE TABLE SALARY\_AUDIT (

&#x20;   AUDIT\_ID         NUMBER GENERATED ALWAYS

&#x20;                    AS IDENTITY PRIMARY KEY,

&#x20;   EMPLOYEE\_ID      NUMBER,

&#x20;   OLD\_SALARY       NUMBER,

&#x20;   NEW\_SALARY       NUMBER,

&#x20;   CHANGE\_AMOUNT    NUMBER,

&#x20;   UPDATE\_DATE      DATE,

&#x20;   USERNAME         VARCHAR2(50),

&#x20;   UPDATE\_TIMESTAMP TIMESTAMP

);



CREATE OR REPLACE TRIGGER TRG\_SALARY\_AUDIT

AFTER UPDATE OF SALARY ON EMPLOYEES

FOR EACH ROW

BEGIN

&#x20;   INSERT INTO SALARY\_AUDIT (

&#x20;       EMPLOYEE\_ID,

&#x20;       OLD\_SALARY,

&#x20;       NEW\_SALARY,

&#x20;       CHANGE\_AMOUNT,

&#x20;       UPDATE\_DATE,

&#x20;       USERNAME,

&#x20;       UPDATE\_TIMESTAMP

&#x20;   )

&#x20;   VALUES (

&#x20;       :OLD.EMPLOYEE\_ID,

&#x20;       :OLD.SALARY,

&#x20;       :NEW.SALARY,

&#x20;       :NEW.SALARY - :OLD.SALARY,

&#x20;       SYSDATE,

&#x20;       SYS\_CONTEXT('USERENV',

&#x20;                   'SESSION\_USER'),

&#x20;       SYSTIMESTAMP

&#x20;   );

END;

/



SELECT COLUMN\_NAME,

&#x20;      DATA\_TYPE

FROM   USER\_TAB\_COLUMNS

WHERE  TABLE\_NAME = 'SALARY\_AUDIT'

ORDER BY COLUMN\_ID;



UPDATE EMPLOYEES

SET    SALARY = 25000

WHERE  EMPLOYEE\_ID = 100;

COMMIT;





SELECT AUDIT\_ID,

&#x20;      EMPLOYEE\_ID,

&#x20;      OLD\_SALARY,

&#x20;      NEW\_SALARY,

&#x20;      CHANGE\_AMOUNT,

&#x20;      UPDATE\_DATE,

&#x20;      USERNAME

FROM   SALARY\_AUDIT

ORDER BY AUDIT\_ID;



UPDATE EMPLOYEES

SET    SALARY = 24000

WHERE  EMPLOYEE\_ID = 100;

COMMIT;



\---------------------------------------------------------------------------

**Change Request 4**

**The Scheduler Job must run:**

**• Every Friday**

**• At 4:00 PM**

**instead of every day.**

**Modify your design.**









BEGIN

&#x20;   DBMS\_SCHEDULER.CREATE\_JOB(

&#x20;       JOB\_NAME        => 'WEEKLY\_HIGH\_SALARY\_REPORT',

&#x20;       JOB\_TYPE        => 'STORED\_PROCEDURE',

&#x20;       JOB\_ACTION      => 'HIGH\_SALARY\_REPORT',

&#x20;       REPEAT\_INTERVAL => 'FREQ=WEEKLY; BYDAY=FRI;

&#x20;                           BYHOUR=16; BYMINUTE=0',

&#x20;       ENABLED         => TRUE

&#x20;   );

END;

/



SELECT JOB\_NAME,

&#x20;      REPEAT\_INTERVAL,

&#x20;      ENABLED,

&#x20;      STATE

FROM   USER\_SCHEDULER\_JOBS

WHERE  JOB\_NAME = 'WEEKLY\_HIGH\_SALARY\_REPORT';

\-------------------------------------------------------------------











































































