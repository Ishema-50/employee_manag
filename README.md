# employee_bonus_manag

Problem :
"Employee Annual Bonus Calculation with Executive Override Rule Using PL/SQL Collections, Records, and GOTO"
Problem Description (Clearly demonstrates all three required concepts):
The Human Resources department needs to automate the annual bonus process with the following business rules:

Employees with more than 3 years of service receive a 15% performance bonus.
The system must collect and store all eligible employees (with their updated salary) in memory for final reporting and payroll upload.
Required PL/SQL FeatureHow This Problem Forces Its UseRecordsEach employee's data (ID, name, hire date, old salary, new salary, department) </br> 
must be treated as one unit → forces use of a RECORD typeCollectionsThere are a variable number of eligible employees → must use a collection (Associative Array/Nested Table) </br>
to store only approved bonus recipientsGOTO StatementWhen the $100,000 executive threshold is hit, we must instantly skip adding that record to the collection and jump to a labeled section

