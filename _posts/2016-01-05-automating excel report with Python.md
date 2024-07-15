---
layout: post
title: Automate Excel Reports with SQL & Python - Handling Different Configurations
---

In this chapter, I will demonstrate how to use Python to automate multiple Excel reports based on different configurations. This approach ensures efficiency and consistency when tailoring reports to meet the specific requirements of different clients.

## Example Scenario
Imagine you have created a base report in Excel that needs to be shared with five different clients, each requiring minor adjustments to the data structure. The framework provided below streamlines this process, allowing you to generate muliple reports with different data specifications in a single execution.

In this example, the base report contains **three standard sheets**:

* _Cover Page_ : Each client shows _different logo_, and _size_.
* _Glossary_ : Same across all clients.
* _Total Store_ : All clients has this sheet.
     
Additionally, there are **two optional sheets** based on the subscription:

* _Online_ : Only for clients subscribed to the online data.
* _Offline_ : Only for clients subscribed to the offline data.

Summary of steps

1. Create a blank Excel template with all necessary data formatting for automation.
2. Set up a master file with data specifications for each client.
3. Write SQL code to export raw data for each client in separate .csv files, all saved in the same folder and follow the same naming structure.
4. Save all brand logos in the same folder and follow the same naming structure.
5. Use Python to automate running multiple reports in one execution based on the configuration set in the master file.


### Step 1 : Create a blank Excel template for automation.

Create a blank Excel template named **Template.xlsx** that includes all five sheets and data formatting.

Here is a screenshot of the sample report: 

<img src="/images/Python automate/Cover.png" alt="Cover" width="200" height="150">

### Step 2 : Setup a master file

Create a master file contains data specification for each client. 

In this example, the master file contains client name, logo dimensions, and channel names (Total, Online, Offline).

file name : Master_File.csv

Client Name  | Category Name | Logo Width (Pixels) | Logo Height (Pixels) | Online Channel | Offline Channel | 
--- | --- | --- | --- | --- | ---
Client A | Soft Drinks | 288 | 115 | Y | Y
Client B | Fruits | 395 | 190 |  | Y
<!-- Client C | 3.84 | 1.10 |  |  -->


## Step 2 : Setup SQL Code and Export Raw Data

Export data for each client in .csv using SQL. Assume you have a table called 'DATA_ALL' that combines data for all clients.

Use the following sample code to export data for each client through the command prompt.

```bash
-- Connect to the database
ybsql -h <host> -d <database> -U <username>
```

* &lt;host&gt;: Replace with your database host.
* &lt;database&gt;: Replace with your database name.
* &lt;username&gt;: Replace with your username.

```bash
\copy (SELECT CHANNEL, PRODUCT_ID, PRODUCT_NAME, WEEK_ID, SALES, UNITS FROM <schema>.DATA_ALL WHERE CLIENT_NAME = 'Client A' ORDER BY 1,2,3,4) to '<path_to_output>/Client A_DATA.csv' WITH ( FORMAT CSV,HEADER TRUE,DELIMITER ',', ENCODING 'UTF-8');
```

```bash
\copy (SELECT CHANNEL, PRODUCT_ID, PRODUCT_NAME, WEEK_ID, SALES, UNITS FROM <schema>.DATA_ALL WHERE CLIENT_NAME = 'Client B' ORDER BY 1,2,3,4) to '<path_to_output>/Client B_DATA.csv' WITH ( FORMAT CSV,HEADER TRUE,DELIMITER ',', ENCODING 'UTF-8');
```
* &lt;schema&gt;: Replace with your schema name.
* &lt;path_to_output&gt;: Replace with your output file path.

Ensure the file name structure is **{Client Name}_DATA.csv** and save all files in the same folder, naming as 'Data File'.

Example 1 :Client A

TOP 5 rows for Client A_DATA.csv

Channel | Product ID | Week | Product Name | Sales | Units
--- | --- | --- | --- | --- | ---
Total | P001 | NAME 1 | 202401 | $900 | 30
Total | P002 | NAME 2 | 202403 | $500 | 20
Offline | P001 | NAME 1 | 202401 | $650 | 40
Offline | P002 | NAME 2 | 202403 | $390 | 15
Online | P001 | NAME 1 | 202403 | $250 | 10


Data File Example 2 : Client B

file name : Client B_DATA.csv

TOP 5 rows for Client B_DATA.csv

Channel | Product ID | Week | Product Name | Sales | Units
--- | --- | --- | --- | --- | ---
Total | P001 | NAME 1 | 202401 | $1600 | 50
Total | P002 | NAME 2 | 202402 | $1800 | 80
Total | P003 | NAME 3 | 202403 | $2000 | 100
Offline | P001 | NAME 1 | 202401 | $100 | 10
Offline | P002 | NAME 2 | 202402 | $150 | 20


<!-- Data File Example 3 : Client C

file name : Client C_DATA.csv

TOP 5 rows for Client C_DATA.csv

Channel | Product ID | Product Name | Sales | Units
--- | --- | --- | --- | ---
Total | P001 | NAME 1 | $750 | 50
Total | P002 | NAME 2 | $500 | 35
Total | P003 | NAME 3 | $1000 | 100
Total | P004 | NAME 4 | $800 | 80
Total | P005 | NAME 5 | $100 | 70 -->


### Step 2 : Manage Brand Logo

1. Create a folder named 'Brand Logo' in your preferred path
2. Save each client's logo in this folder, using the structure **{Client Name}_LOGO.png**.

Example Logos:

* Client A_LOGO.png
* Client B_LOGO.png






