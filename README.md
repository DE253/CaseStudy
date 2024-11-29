# CaseStudy - Retail
This case study entails  ingesting data from database server and http API into the datalake storage account and further cleaning and transforming data. The transformed data will finally stored in the staging container.
## Created a resource group - Casestudy1
Within this resource group, created a database (sourcedb), data factory (servertoraw) and a data lake storage account (dlretail).
## Inserting data in database
Download Microsoft SQL Server 2022 & SSMS 20
Open SSMS and create a new Query

### SQL code
Create database retail;

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

![alt text](C:\Users\deept\OneDrive\Desktop\DE Prep\Case Study 1\SQL server data.jpg)

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

   
## Cleaning and storing the data in curated container

Create databricks resource and launch workspace.
Create a cluster with unrestricted policy, single node, and access mode - no isolation shared. For performance, select runtime as 15.4 LTS, node type - Standard_DS3_v2.

Then created a folder Retail CS in workspace and opened a new notebook.

### Accessing storage account through service principal
Go to app registrations in Microsoft Entra ID and and create a new app. In the overview, copy the client ID and directory id.
Then create secrets within that app and copy the client secret value. Use Key vault to encrypt this value.

Issue  Storage Blob Data Reader and  Storage Blob Data Contributor role for the app in the storage account (IAM) to establish connection and accessibility to databricks.

Use the client ID, directory id, and client secret for databricks to access storage account.

### Reading and transforming tables
Then mount the storage container and check if the mount was successful.

Define Schema for all the tables and read data in each table.

Performed transformations b dropping last_update column in all tables and address2 column from address table.

## Troubleshooting scenarios

### Enable Adaptive Query Execution (AQE)
Enable AQE to optimise skewed data and perform join operation better.
spark.conf.set("spark.sql.adaptive.enabled", "true")

### Adjust Partitioning
Analyze partitions with : df.rdd.getNumPartitions()
Count number of rows for better understanding with row_count = df.count()
Then, repartition for better balance and increase parllelism with : df = df.repartition(10) 

Used coalesce to reduce the management of too many small partitions and to optimize resource utilization df=df.coalesce().

## Moving Data to Curated container
Mount the target container and write the updated DataFrame to ADLS in delta format using append mode.

## Performing join operation to list active customers

We need to retrieve a list of active customers along with their associated address, city, and country information. The data should be filtered for only active customers (activebool = TRUE). For each customer, the following details are needed:

Customer Information: customer_id, first_name, last_name, email, active (status of the customer)
Address Information: address, district, postal_code, phone
City Information: city
Country Information: country

1. Load Partitioned Data from curated container

Load files df1, df2, df3, df4 from the curated container into dataframes address_df, city_df, country_df , and customer_df respectively.

2. Filter Active Customers and join the tables. Finally select the required columns and form a single table.

3. Then save this in delta format in staging container.


















    
