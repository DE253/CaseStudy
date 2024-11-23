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

## Create pipeline for daata ingestion into the datalake storage

### Create linked services
1.  SQL Database
    Go to Manage > Linked Services in ADF.
    Create a new linked service for SQL database.
    Provide the connection details:
        Server name, Database name, Authentication method (SQL Authentication or Azure AD), and credentials.
	Add key vault for sql password and create linked service for key vault. Then go to settings in data factory and copy "Manage Identity object". Then go to Vault > access policies> click create
 	Under permission - select all
  	And in principal - paste the copied "Manage Identity object" and select from the list and click create. This will give data factory the access to key vault. 
        Test the connection.

2.  Data Lake Storage
    Create another linked service for Azure Data Lake Storage Gen2.
    Provide the account name.
    Test the connection.
    
3.  HTTP
    Create another linked service for HTTP.
    Provide the details:
    name, base url (click on the dataset in github, click raw and copy the url), choose authentication type as Anonymous as the data is public and click create.
    We will have created 4 http for address, city, country and customer.

### Create Copy data activity
    Add a Copy Data Activity:
    Drag a Copy Data activity into the pipeline.

    For SQL database
    Configure the source with SQL dataset by selecting the table name.
    Configure the sink with Data Lake dataset. Define the file name and format (CSV) for the output with path as raw container. We will have 1 copy data activity for SQL db.
        

    For HTTP
    Configure the source with http dataset.
    Configure the sink with Data Lake dataset. Define the file name and format (CSV) for the output with path as raw container. We will have 4 copy data activities for each API source.

    Connect all the activities through "on completion".

    Added a scheduled trigger for every 15 days.

    Validate and debug the pipeline. All the files moved to raw container. 

   



















    
