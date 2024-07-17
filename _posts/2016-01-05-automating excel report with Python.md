---
layout: post
title: Automate Excel Reports with SQL & Python - Handling Different Configurations
---

In this chapter, I will demonstrate how to use Python to automate multiple Excel reports based on different configurations. This approach ensures efficiency and consistency when tailoring reports to meet the specific requirements of different clients.

## Example Scenario
Imagine you have created a base report in Excel that needs to be shared with different clients, each requiring minor adjustments to the data structure. The framework provided below streamlines this process, allowing you to generate muliple reports with different data specifications in a single execution.

In this example, the base report contains __three standard sheets__:

<img src="/images/Python automate/Cover.png" alt="Cover" width="300" height="200">  <img src="/images/Python automate/Glossary.png" alt="Glossary" width="300" height="200">  <img src="/images/Python automate/tot.png" alt="tot" width="300" height="200">  

* **Cover Page** : Each client has a **different brand logo** and **size**. The **category name** on the cover also varies based on the subscription.
* **Glossary** : Same across all clients.
* **Total Store** : Every client subscribed total store data.
     
Additionally, there are **two optional sheets** based on the subscription:

<img src="/images/Python automate/Online.png" alt="Online" width="300" height="200">  <img src="/images/Python automate/Offline.png" alt="Offline" width="300" height="200">

* **Online** : Only for clients subscribed to the online data.
* **Offline** : Only for clients subscribed to the offline data.

**Process Summary**

1. Create an **Excel template** with all the sheets mentioned above, leaving the data areas blank.
2. Set up **a master file** contains data specifications for each client.
3. Write SQL code to **export raw data** for each client separately, all saved in the same folder and follow the naming structure as {Client Name}_DATA.csv.
4. Save all **brand logos** in the same folder and follow the naming structure as {Client Name}_LOGO.png.
5. Use **Python to automate running multiple reports** in one execution based on the configuration set in the master file.

---

### Step 1 : Create a blank Excel template for automation.

Create a blank Excel template named **Template.xlsx** with all five sheets as shown above and keep all necessary formatting.

### Step 2 : Setup a master file

Create a master file contains data specification for each client. 

In this example, the master file contains client name, logo dimensions, and channel flag.
file name : Master_File.csv

Client Name  | Category Name | Logo Width (Pixels) | Logo Height (Pixels) | Online Channel | Offline Channel | 
--- | --- | --- | --- | --- | ---
Client A | Soft Drinks | 288 | 115 | Y | Y
Client B | Fruits | 395 | 190 |  | Y
Client C | HBA | 100 | 50 | Y | 
Client D | Tea | 120 | 60 | Y | Y


## Step 2 : Setup SQL Code and Export Raw Data

Export data for each client in .csv using Yellowbrick. Assume you have a table called 'DATA_ALL' that combines data for all clients and all channels that they subscribed to.

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

Ensure the file name structure is **{Client Name}_DATA.csv** and save all files in the same folder. The folder can be named as 'Data File'.

Example 1 :Client A

Channel | Product ID | Week | Product Name | Sales | Units
--- | --- | --- | --- | --- | ---
Total | P001 | NAME 1 | 202401 | $900 | 30
Total | P002 | NAME 2 | 202403 | $500 | 20
Offline | P001 | NAME 1 | 202401 | $650 | 40
Offline | P002 | NAME 2 | 202403 | $390 | 15
Online | P001 | NAME 1 | 202403 | $250 | 10


Data File Example 2 : Client B

file name : Client B_DATA.csv

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


### Step 3 : Manage Brand Logo

1. Create a folder named 'Brand Logo' in your preferred path
2. Save each client's logo in this folder, using the structure **{Client Name}_LOGO.png**.

Example Logos:

* Client A_LOGO.png
* Client B_LOGO.png


### Step 4 : Use Python to cut and paste data into the Excel template based on the Channel tab.

Import library

```python
import pandas as pd
import openpyxl 
from openpyxl.drawing.image import Image
import xlwings as xl
import os
import shutil 
```

Import data from Master CSV file
```python
csv_file_path = '<file location>/Master_File.csv'  
cpg_info_df = pd.read_csv(csv_file_path)
cpg_info_df.head()
```

create the path for Output folder
```python
output_folder = '<file location>/Output'  # Replace with your output folder path
```

create the path for Template file
```python
template_path = '<file location>/Template.xlsx'  # Replace with your template folder path
```
Generate the path for each client's data file and filter by the channel value.

```python
def create_df_data_total(client_name, filter_value):
    file_path = f'<file location>/{client_name}_DATA.csv'
    df_data_total_1 = pd.read_csv(file_path)

    # Filter the DataFrame based on the 'Channel' column value
    df_data_total = df_data_total_1[df_data_total_1['Channel'] == filter_value]  

    return df_data_total
```

Then do the same for online and offline

```python
def create_df_data_online(client_name, filter_value):
    file_path = f'<file location>/{client_name}_DATA.csv'
    df_data_online_1 = pd.read_csv(file_path)

    # Filter the DataFrame based on the 'Channel' column value
    df_data_online = df_data_online_1[df_data_online_1['Channel'] == filter_value]  

    return df_data_online

def create_df_data_offline(client_name, filter_value):
    file_path = f'<file location>/{client_name}_DATA.csv'
    df_data_offline_1 = pd.read_csv(file_path)

    # Filter the DataFrame based on the 'Channel' column value
    df_data_offline = df_data_offline_1[df_data_offline_1['Channel'] == filter_value]  

    return df_data_offline


```

Next, define a function to populate the cover page with the category name and brand logo.

```python
def fill_cover_page(master_file_info):
    
    #Set the name of the final report
    output_file_name = f'Final Report - {client_name}.xlsx'
    output_file_path = os.path.join(output_folder, output_file_name)

    #Copy the template file to the output folder 
    shutil.copy(template_path, output_file_path)

    # Load workbook
    wb = openpyxl.load_workbook(output_file_path)
    
    # Select the 'Cover Page' sheet
    sheet = wb['Cover Page']

    # Fill in category for each Client in cell B16
    sheet['B16'] = category_name
    
    # Find the logo file corresponding to the lient name
    logo_folder = '<file location>/Brand Logo'
    for file in os.listdir(logo_folder):
        if file.startswith(client_name) and file.endswith('.png'):
            logo_path = os.path.join(logo_folder, file)
            break
    # Add the image to cell K16
    cell_position = 'K16'
    
    # Add logo to the worksheet
    img = Image(logo_path)
    img.width = image_width_pixels
    img.height = image_height_pixels
    sheet.add_image(img, cell_position)  
    
    wb.save(output_file_path)
    wb.close()
    return output_file_path

```

Create the main automation loop that iterates through each Client from the master file

```python
# start iterating through each Client from the master file
for index, row in cpg_info_df.iterrows():    
    master_file_info = ( row['Client Name'],row['Category Name'], row['Logo Width (Pixels)'], row['Logo Height (Pixels)'], row['Online Channel']\
                , row['Offline Channel'])
    client_name = master_file_info[0]
    category_name = master_file_info[1]
    image_width_pixels = master_file_info[2]
    image_height_pixels = master_file_info[3]
    online = master_file_info[4]
    offline = master_file_info[5]
    
    #Populate the cover page base on category name and brand logo
    output_file_path = fill_cover_page(master_file_info)
    
    #Create dataframe contains data for 'Total' channel and drop 'Channel' column
    df_data_total = create_df_data_total(client_name, filter_value='Total')
    df_data_total = df_data_total.drop(columns=['Channel'])

    #Remove Header and convert to list
    data_total = df_data_total.reset_index(drop=True).values.tolist()
    
    #If Online Flag = 'Y', create dataframe contains data for 'Online' channel and drop 'Channel' column
    if online == 'Y' :
        df_data_online = create_df_data_online(client_name, filter_value='Online')
        df_data_online = df_data_online.drop(columns=['Channel'])
        #Remove Header and convert to list
        data_online = df_data_online.reset_index(drop=True).values.tolist()
    
    #If Offline Flag = 'Y', create dataframe contains data for 'Offline' channel and drop 'Channel' column
    if offline == 'Y' :
        df_data_offline = create_df_data_offline(client_name, filter_value='Offline')
        df_data_offline = df_data_offline.drop(columns=['Channel'])
        #Remove Header and convert to list
        data_offline = df_data_offline.reset_index(drop=True).values.tolist()
        
    #Open the output file
    wb = xl.Book(output_file_path)
    
    #Paste data into 'Total Store' sheet
    ws = wb.sheets[f'Total Store']
    ws.range(f"A4").value = data_total
    
    #If both Online and Offline flag = 'Y', paste data into both sheets
    if online == 'Y' and offline == 'Y':
        ws1 = wb.sheets[f'Online']
        ws1.range(f"A4").value = data_online
        
        ws2 = wb.sheets[f'Offline']
        ws2.range(f"A4").value = data_offline
    
    #If only Online flag = 'Y', paste data into sheet 'Online' and drop 'Offline' sheet
    if online == 'Y' and offline != 'Y':
        ws = wb.sheets[f'Online']
        ws.range(f"A4").value = data_online
        wb.sheets[f'Offline'].delete()

    #If only Offline flag = 'Y', paste data into sheet 'Offline' and drop 'Online' sheet 
    if online != 'Y' and offline == 'Y':
        ws = wb.sheets[f'Offline']
        ws.range(f"A4").value = data_offline
        wb.sheets[f'Online'].delete()
    
    #If none of Online or Offline flag = 'Y', drop both of the sheets.
    if online != 'Y' and offline != 'Y':
        wb.sheets[f'Online'].delete()
        wb.sheets[f'Offline'].delete()

    # Set the selected worksheet as the active sheet
    ws = wb.sheets['COVER PAGE']
    wb.active = ws
    
    wb.save()
    wb.close()   

```