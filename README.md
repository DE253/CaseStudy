# CaseStudy - Retail
This case study entails  ingesting data from database server and http API into the datalake storage account and further cleaning and transforming data. The transformed data will finally stored in the staging container.
## Created a resource group - Casestudy1
Within this resource group, created a database (sourcedb), data factory (servertoraw) and a data lake storage account (dlretail).
## Inserting data in database
Using SQL Scripts in Azure SQL Database
1. Go to the Azure Portal, navigate to SQL Database, and open the Query Editor.
2. Run a script to create a table and add data:
### SQL code
DROP TABLE IF EXISTS Employees;

CREATE TABLE Employees (
    emp_id INT PRIMARY KEY,
    first_name NVARCHAR(100) NOT NULL,
	last_name NVARCHAR(100) NOT NULL,
	birthdate DATE,
    hire_date DATE,
    salary FLOAT
);

INSERT INTO Employees (emp_id, first_name, last_name, birthdate, hire_date, salary)
VALUES (3, 'Jose', 'Portilla', '1997-11-03', '2010-01-01', 100),
       (4, 'Sam', 'Smith', '1995-11-03', '2008-01-01', 100),
	   (5, 'Sid', 'Peter', '1996-11-03', '2020-01-01', 87);

select * from Employees

##
