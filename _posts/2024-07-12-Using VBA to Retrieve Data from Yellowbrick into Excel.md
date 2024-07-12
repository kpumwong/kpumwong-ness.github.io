---
layout: post
title: Using VBA to Retrieve Data from Yellowbrick into Excel
---

Provide a simple example of setting up an Excel template and VBA code to populate data from Yellowbrick. The goal is to enable users without database knowledge to export specific data from a database into an Excel file.

## Step 1 : Create the "INPUT" Sheet:

Create a sheet named "INPUT" in your Excel workbook.
This sheet will hold the database connection parameters and any other variables needed for the VBA code.

Here is an example of how to set up the 'INPUT' sheet:

<img src="/images/Cate Overview/cart input.png" alt="cart input" class="fit image">


## Step 2: Set Up Connections

Here is the code to define the function `oConn()` to establish and return a database connection.

```vba
' Function to establish and return a database connection
Public Function oConn() As Object

    ' Create a new ADO Connection object
    Set oConn = CreateObject("ADODB.Connection")

    ' Set command timeout to 600 seconds (10 minutes)
    oConn.commandTimeout = 600

    ' Retrieve connection parameters from the "INPUT" worksheet
    strDataSource = Sheets("INPUT").Range("B1").Value 
    strUsername = Sheets("INPUT").Range("B2").Value   
    strPassword = Sheets("INPUT").Range("B3").Value   

    ' Open the connection using the retrieved parameters
    oConn.Open "DSN=" & strDataSource & ";Server=SERVERNAME;Port=PORT_NUMBER;Database=DATABASE_NAME UserId=" & strUsername & ";Password=" & strPassword & ";"

End Function

```

## Step 3: Storing and Executing SQL Code

For example, TABLE_PRE is created based on TABLE_1, and TABLE_FINAL contains the final dataset that will be pasted into the Excel template.

```vba

' Establish database connection
Set adoCn = oConn()

' Create a new Recordset object
Set dbRes = CreateObject("ADODB.Recordset")

' Create table TABLE_PRE in Yellowbrick
dbRes.Open "" _
& " create table TABLE_PRE as " _
& " select A.* " _
& " from TABLE_1 A" _
& " ;", adoCn

' Create table TABLE_FINAL from TABLE_PRE in Yellowbrick
dbRes.Open "" _
& " create table TABLE_FINAL as " _
& " select A.* " _
& " from TABLE_PRE A" _
& " ;", adoCn

```

## Step 4: Paste Data into an excel template

Next, the data stored in the Recordset object can be pasted into the Excel template, specifically into the 'MAIN' sheet and a specified range.

```vba

' Retrieve data from TABLE_FINAL
dbRes.Open "" _
& " SELECT * FROM TABLE_FINAL order by 1, 2, 3; ", adoCn

' Paste data into the 'MAIN' sheet starting at cell A2
Sheets("MAIN").Range("A2").CopyFromRecordset dbRes

```