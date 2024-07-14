---
layout: post
title: Automate Excel Reports with SQL & Python - Handling Different Configurations
---

In this chapter, I will demonstrate how to use Python to automate multiple Excel reports based on different configurations. This approach ensures efficiency and consistency when tailoring reports to meet the specific requirements of different clients.

### Example Scenario
Imagine you have created a base report in Excel. This report needs to be shared with five different clients, each requiring minor adjustments to the data structure. The Python code provided below streamlines this process, allowing you to generate all necessary reports with a single execution.

## Step 1 : Setup a master file

This master file should contain data specification for each of the client. In this exmaple this report tell category performance base on the category and channel that each client subscribe, each client has their brand logo of different size which the logo needs to be put in the cover page. Hence, data specification contains Client name, category name to put in Cover Page,  Logo Width (Inches), Logo Height (Inches) and channel names

For example

Client Name  | Logo Width (Inches) | Logo Height (Inches) | Channel Total | Channel Online | Channel Offline | 
--- | --- | ---| ---| ---| ---
Client A | 3.42 | 1.24 | Y | Y | Y
Client B | 4.13 | 0.79 | Y |  | Y
Client C | 3.84 | 1.10 | Y |  | 


## Step 2 : Setup an SQL code and export the raw data for the report in csv

Set up an SQL code and export the data by Client Name in separate file. Assumming you have table called DATA_ALL that combines data for every client, here is the example of code to export all data files by client at the same time.

```bash

-- Connect to the database
ybsql -h <host> -d <database> -U <username>

```

* &lt;host&gt;: Replace with your database host.
* &lt;database&gt;: Replace with your database name.
* &lt;username&gt;: Replace with your username.

```bash
--CLIENT A EXPORT
\copy (SELECT   CHANNEL
                , PRODUCT_ID
                , PRODUCT_NAME
                , SALES
                , UNITS
    FROM        <schema>.DATA_ALL
    WHERE       CLIENT_NAME = 'Client A' -- Client A 
    ORDER BY    1,2,3) 
    to '<path_to_output>/Client A_DATA.csv' WITH (
    FORMAT CSV, 
    HEADER TRUE, 
    DELIMITER ',', 
    ENCODING 'UTF-8'
);
```

```bash
--CLIENT B EXPORT
\copy (SELECT   CHANNEL
                , PRODUCT_ID
                , PRODUCT_NAME
                , SALES
                , UNITS
    FROM        <schema>.DATA_ALL
    WHERE       CLIENT_NAME = 'Client B' -- Client B
    ORDER BY    1,2,3) 
    to '<path_to_output>/Client B_DATA.csv' WITH (
    FORMAT CSV, 
    HEADER TRUE, 
    DELIMITER ',', 
    ENCODING 'UTF-8'
);

```

* &lt;schema&gt;: Replace with your schema name.
* &lt;path_to_output&gt;: Replace with your output file path.

```bash
-- Connect to the database
ybsql -h <host> -d <database> -U <username>

```

* &lt;host&gt;: Replace with your database host.
* &lt;database&gt;: Replace with your database name.
* &lt;username&gt;: Replace with your username.

```bash
<div style="overflow-x: auto;">
<pre>
<code>
\copy (SELECT CHANNEL, PRODUCT_ID, PRODUCT_NAME, SALES, UNITS FROM        <schema>.DATA_ALL WHERE CLIENT_NAME = 'Client A' ORDER BY 1,2,3) to '<path_to_output>/Client A_DATA.csv' WITH ( FORMAT CSV,HEADER TRUE,DELIMITER ',', ENCODING 'UTF-8');
</code>
</pre>
</div>
```

```bash
<div style="overflow-x: auto;">
<pre>
<code>
\copy (SELECT CHANNEL, PRODUCT_ID, PRODUCT_NAME, SALES, UNITS FROM        <schema>.DATA_ALL WHERE CLIENT_NAME = 'Client B' ORDER BY 1,2,3) to '<path_to_output>/Client B_DATA.csv' WITH ( FORMAT CSV,HEADER TRUE,DELIMITER ',', ENCODING 'UTF-8');
</code>
</pre>
</div>
```
* &lt;schema&gt;: Replace with your schema name.
* &lt;path_to_output&gt;: Replace with your output file path.


Export the file in csv based by each client, 
1. Create folder 'Data File' in your preferred path
2. Make sure to save the file as {Client Name}_DATA.csv all files into the same folder

Example 1 :Client A

TOP 5 rows for Client A_DATA.csv

Channel | Product ID | Product Name | Sales | Units
--- | --- | --- | --- | ---
Total | P001 | NAME 1 | $1600 | 50
Total | P002 | NAME 2 | $1800 | 80
Total | P003 | NAME 3 | $2000 | 100
Online | P001 | NAME 1 | $100 | 10
Online | P002 | NAME 2 | $150 | 20



Example 2 : Client B

file name : Client B_DATA.csv

TOP 5 rows for Client B_DATA.csv

Channel | Product ID | Product Name | Sales | Units
--- | --- | --- | --- | ---
Total | P001 | NAME 1 | $900 | 30
Total | P002 | NAME 2 | $500 | 20
Offline | P001 | NAME 1 | $650 | 40
Offline | P002 | NAME 2 | $390 | 15
Online | P001 | NAME 1 | $250 | 10


### II. Brand Logo

1. Create folder 'Brand Logo' in your preferred path
2. Save the image of the brand logo under this folder 'Brand Logo', makes sure you name it with a strucure {Client Name}_LOGO.png

Example 1 :Client A

Logo image : Client A_LOGO.png


Example 2 : Client B

Logo image : Client B_LOGO.png



### III. Set up the master file 

The master file based on these two clients would

Example : 


Client Name  | Logo Width (Inches) | Logo Height (Inches) | Channel Total | Channel Online | Channel Offline | 
--- | --- | ---| ---| ---| ---
Client A | 3.42 | 1.24 | Y | Y | Y
Client B | 4.13 | 0.79 | Y |  | Y
Client C | 3.84 | 1.10 | Y |  | 
Client D | 2.51 | 1.18 | Y | Y | 
Client E | 5.04 | 1.16 | Y | Y | Y


