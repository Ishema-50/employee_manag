# employee_bonus_manag

Problem Title:
"Employee Annual Bonus Calculation with Executive Override Rule Using PL/SQL Collections, Records, and GOTO"
Problem Description (Clearly demonstrates all three required concepts):
The Human Resources department needs to automate the annual bonus process with the following business rules:

Employees with more than 3 years of service receive a 15% performance bonus.
The system must collect and store all eligible employees (with their updated salary) in memory for final reporting and payroll upload.
Required PL/SQL FeatureHow This Problem Forces Its UseRecordsEach employee's data (ID, name, hire date, old salary, new salary, department) </br> 
must be treated as one unit → forces use of a RECORD typeCollectionsThere are a variable number of eligible employees → must use a collection (Associative Array/Nested Table) </br>
to store only approved bonus recipientsGOTO StatementWhen the $100,000 executive threshold is hit, we must instantly skip adding that record to the collection and jump to a labeled section

here are the codes 
-- Create DEPARTMENTS table
CREATE TABLE DEPARTMENTS (
    DEPARTMENT_ID NUMBER(4) PRIMARY KEY,
    DEPARTMENT_NAME VARCHAR2(30) NOT NULL,
    LOCATION VARCHAR2(50)
);

-- Create EMPLOYEES table
CREATE TABLE EMPLOYEES (
    EMPLOYEE_ID NUMBER(6) PRIMARY KEY,
    FIRST_NAME VARCHAR2(20),
    LAST_NAME VARCHAR2(25) NOT NULL,
    EMAIL VARCHAR2(25) NOT NULL UNIQUE,
    PHONE_NUMBER VARCHAR2(20),
    HIRE_DATE DATE NOT NULL,
    JOB_TITLE VARCHAR2(35) NOT NULL,
    SALARY NUMBER(8,2),
    DEPARTMENT_ID NUMBER(4),
    CONSTRAINT FK_DEPARTMENT FOREIGN KEY (DEPARTMENT_ID) REFERENCES DEPARTMENTS(DEPARTMENT_ID)
);
-- Insert into DEPARTMENTS
INSERT INTO DEPARTMENTS (DEPARTMENT_ID, DEPARTMENT_NAME, LOCATION) VALUES (10, 'Administration', 'New York');
INSERT INTO DEPARTMENTS (DEPARTMENT_ID, DEPARTMENT_NAME, LOCATION) VALUES (20, 'Marketing', 'Los Angeles');
INSERT INTO DEPARTMENTS (DEPARTMENT_ID, DEPARTMENT_NAME, LOCATION) VALUES (30, 'IT', 'San Francisco');
INSERT INTO DEPARTMENTS (DEPARTMENT_ID, DEPARTMENT_NAME, LOCATION) VALUES (40, 'Sales', 'Chicago');

-- Insert into EMPLOYEES
INSERT INTO EMPLOYEES (EMPLOYEE_ID, FIRST_NAME, LAST_NAME, EMAIL, PHONE_NUMBER, HIRE_DATE, JOB_TITLE, SALARY, DEPARTMENT_ID)
VALUES (100, 'niyo', 'bosco', 'bosco@gmail.com', '123-456-7890', TO_DATE('2020-01-15', 'YYYY-MM-DD'), 'Manager', 80000, 10);

INSERT INTO EMPLOYEES (EMPLOYEE_ID, FIRST_NAME, LAST_NAME, EMAIL, PHONE_NUMBER, HIRE_DATE, JOB_TITLE, SALARY, DEPARTMENT_ID)
VALUES (101, 'manzi', 'Bernard', 'bern@gmail.com', '987-654-3210', TO_DATE('2019-05-20', 'YYYY-MM-DD'), 'Developer', 60000, 30);

INSERT INTO EMPLOYEES (EMPLOYEE_ID, FIRST_NAME, LAST_NAME, EMAIL, PHONE_NUMBER, HIRE_DATE, JOB_TITLE, SALARY, DEPARTMENT_ID)
VALUES (102, 'Alice', 'mwiza', 'alice@gmail.com', '555-123-4567', TO_DATE('2021-03-10', 'YYYY-MM-DD'), 'Sales Rep', 55000, 40);

INSERT INTO EMPLOYEES (EMPLOYEE_ID, FIRST_NAME, LAST_NAME, EMAIL, PHONE_NUMBER, HIRE_DATE, JOB_TITLE, SALARY, DEPARTMENT_ID)
VALUES (103, 'Bob', 'prince', 'prince@gmail.com', '444-987-6543', TO_DATE('2018-11-05', 'YYYY-MM-DD'), 'Marketer', 50000, 20);

INSERT INTO EMPLOYEES (EMPLOYEE_ID, FIRST_NAME, LAST_NAME, EMAIL, PHONE_NUMBER, HIRE_DATE, JOB_TITLE, SALARY, DEPARTMENT_ID)
VALUES (104, 'ishimwe', 'Davis', 'davis@gmail.com', '333-456-7890', TO_DATE('2022-07-01', 'YYYY-MM-DD'), 'Analyst', 65000, 30);

CREATE OR REPLACE PROCEDURE PROCESS_EMPLOYEE_BONUSES AS
    -- Define a Record type to hold employee details
    TYPE employee_record_type IS RECORD (
        employee_id EMPLOYEES.EMPLOYEE_ID%TYPE,
        full_name VARCHAR2(50),  -- Concatenated first and last name
        hire_date EMPLOYEES.HIRE_DATE%TYPE,
        salary EMPLOYEES.SALARY%TYPE,
        department_name DEPARTMENTS.DEPARTMENT_NAME%TYPE
    );
    
    -- Define a Collection (associative array) to store eligible employee records
    TYPE employee_collection_type IS TABLE OF employee_record_type INDEX BY PLS_INTEGER;
    eligible_employees employee_collection_type;
    
    -- Cursor to fetch employee data joined with departments
    CURSOR emp_cursor IS
        SELECT e.EMPLOYEE_ID,
               e.FIRST_NAME || ' ' || e.LAST_NAME AS FULL_NAME,
               e.HIRE_DATE,
               e.SALARY,
               d.DEPARTMENT_NAME
        FROM EMPLOYEES e
        LEFT JOIN DEPARTMENTS d ON e.DEPARTMENT_ID = d.DEPARTMENT_ID;
    
    -- Variables
    emp_rec employee_record_type;  -- Single record instance
    idx PLS_INTEGER := 1;  -- Index for collection
    adjusted_salary NUMBER;
    years_employed NUMBER;
    
BEGIN
    -- Open and loop through the cursor
    OPEN emp_cursor;
    LOOP
        FETCH emp_cursor INTO emp_rec;
        EXIT WHEN emp_cursor%NOTFOUND;
        
        -- Calculate years employed
        years_employed := TRUNC(MONTHS_BETWEEN(SYSDATE, emp_rec.hire_date) / 12);
        
        -- Check eligibility for bonus (hired more than 3 years ago)
        IF years_employed > 3 THEN
            -- Calculate 10% bonus
            adjusted_salary := emp_rec.salary * 1.10;
            
            -- Special case: If adjusted salary > 70000, use GOTO to jump to logging
            IF adjusted_salary > 70000 THEN
                GOTO high_earner_skip;
            END IF;
            
            -- Add to collection if eligible and no skip
            eligible_employees(idx) := emp_rec;
            eligible_employees(idx).salary := adjusted_salary;  -- Update salary in record
            idx := idx + 1;
        END IF;
        
        <<high_earner_skip>>
        IF adjusted_salary > 70000 THEN
            DBMS_OUTPUT.PUT_LINE('High earner skipped: ' || emp_rec.full_name || ' (Adjusted: ' || adjusted_salary || ')');
        END IF;
    END LOOP;
    CLOSE emp_cursor;
    
    -- Output the collection of eligible employees
    IF eligible_employees.COUNT > 0 THEN
        DBMS_OUTPUT.PUT_LINE('Eligible Employees for Bonus:');
        FOR i IN eligible_employees.FIRST .. eligible_employees.LAST LOOP
            DBMS_OUTPUT.PUT_LINE('ID: ' || eligible_employees(i).employee_id ||
                                 ', Name: ' || eligible_employees(i).full_name ||
                                 ', Department: ' || eligible_employees(i).department_name ||
                                 ', Adjusted Salary: ' || eligible_employees(i).salary);
        END LOOP;
    ELSE
        DBMS_OUTPUT.PUT_LINE('No eligible employees found.');
    END IF;
    
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('No employee data available.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('An error occurred: ' || SQLERRM);
END PROCESS_EMPLOYEE_BONUSES;
/




