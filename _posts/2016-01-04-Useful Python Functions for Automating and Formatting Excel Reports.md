---
layout: post
title: Useful Python Functions for Automating and Formatting Excel Reports
---

Here are some useful Python functions that I found helpful in automating Excel reports. In this chapter, we will focus on pivoting a dataframe using Pandas and applying data formats using xlsxwriter.

## Pivoting Dataframes

Suppose you have sales and units data by Product ID (UPC_ID) and week (WEEK_ID) in a table, and you want to display WEEK_ID as columns with sales or units as a metric. You can easily achieve this using pandas to pivot WEEK_ID into columns and select the metric.

Here is the code for pivoting data and export to Excel file:

```python

#Import library
import pandas as pd
import psycopg2 as pg

#Connect to the database
conn = pg.connect( user = <username>, password = <password>, database = <database>, host = <host>)
cus=conn.cursor()

#reads the results of the SQL query into a DataFrame
df = pd.read_sql_query("""
select          DISTINCT UPC_ID
                , UPC_DSC
                , CATEGORY
                , WEEK_ID 
                , SALES
                , UNITS  
from            TABLE_A ;""",conn)

# Create sales by week data, drop the 'units' column
df_sales=df.drop(['units'],axis=1)
df_sales_pv=pd.pivot_table(df_sales,index = ['upc_id','upc_dsc','category'],columns = 'week_id',values='sales').reset_index()
df_sales_pv.fillna(0, inplace=True)


# Create units by week data, drop the 'sales' column
df_units=df.drop(['sales'],axis=1)
df_units_pv=pd.pivot_table(df_units,index = ['upc_id','upc_dsc','category'],columns = 'week_id',values='units').reset_index()
df_units_pv.fillna(0, inplace=True)

# Export to Excel
output_path_sales = '<your folder path>/<file name>.xlsx'  # Replace with your output file path
df_sales_pv.to_excel(output_path_sales, index=False)

output_path_units = '<your folder path>/<file name>.xlsx'  # Replace with your output file path
df_units_pv.to_excel(output_path_units, index=False)

```


## Setting Data Formats in Excel

When automating Excel reports, you may need to apply specific data formats. The following example shows how to use xlsxwriter to format data in an Excel file.

Here, we demonstrate key features of xlsxwriter to improve your Excel reports' presentation and usability.

### Key Data Formatting Features of xlsxwriter
⋅⋅* Text Formatting
⋅⋅* Number Formatting
⋅⋅* Color Formatting
⋅⋅* Alignment Formatting
⋅⋅* Border Formatting
⋅⋅* Combining Multiple Formats
⋅⋅* Data Bars for Visualization
⋅⋅* Applying a 3-Scale Color Based on Value
⋅⋅* Setting Tab Color
⋅⋅* Freezing Panes
⋅⋅* Applying Autofilters


An example below shows how to apply these key features to a summary of sales and units data by Division, Department, and Category in an Excel file. 

This image illustrates the final output.
<img src="/images/Python automate/summary.png" alt="summary" class="fit image">

```python

#Import library
import pandas as pd
import xlsxwriter

# Sample data
data = {
    "Division": ["Electronics","Electronics", "Furniture", "Furniture"],
    "Department": ["Laptops","Ovens", "Chairs","Tables"],
    "Category": ["Gaming","Cooking", "Office","Dining"],
    "Sales L52W": [10000, 20000,5000,7500],
    "Sales L52W Change": [500, -3300,1000,-500],
    "Sales L52W % Change": [0.05, -0.03,0.02,-0.001],
    "Units L52W": [150, 300,500,450],
    "Units L52W Change": [100, -50, 180,-100],
    "Units L52W % Change": [0.07, -0.02,0.02,-0.001],
    "Remark" : ["increase","decrease","increase","decrease"]
}

df_summary = pd.DataFrame(data)
df_summary.head()

```

Construct output path and set up Excel writer

```python
output_path = '<your folder path>/<file name>.xlsx'
writer = pd.ExcelWriter(output_path, engine='xlsxwriter')
df_summary.to_excel(writer, 'SUMMARY', startrow=1, index=False, header=False)

workbook = writer.book
worksheet = writer.sheets['SUMMARY']

```

### Text Formatting

'bold_format': Bold text
'italic_format': Italic text
'underline_format': Underlined text

```python
bold_format = workbook.add_format({'bold': True})
italic_format = workbook.add_format({'italic': True})
underline_format = workbook.add_format({'underline': True})
format_bold_italic = workbook.add_format({'bold': True, 'italic': True})

worksheet.set_column('A:A', 15, bold_format)
worksheet.set_column('B:B', 15, italic_format)
worksheet.set_column('C:C', 15, format_bold_italic)
worksheet.set_column('j:j', 15, underline_format)
```

### 'Number Formatting' 

'currency_format': Currency format with the Euro symbol
'percentage_format': Percentage format with one decimal place

```python
currency_format = workbook.add_format({'num_format': '€#,##0.00'})
percentage_format = workbook.add_format({'align': 'center', 'num_format': '0.0%'})

worksheet.set_column('D:D', 18, currency_format)
worksheet.set_column('E:E', 18, currency_format)
worksheet.set_column('F:F', 18, percentage_format)
worksheet.set_column('I:I', 18, percentage_format)
```

### Color Formatting

'red_format': Red font color 
'pink_bg_format': Pink background
'blue_bg_format': Blue background

```python
red_format = workbook.add_format({'font_color': '#e00d2d'})
pink_bg_format = workbook.add_format({'bg_color': '#FFC0CB'}) # Pink color
blue_bg_format = workbook.add_format({'bg_color': '#ADD8E6'}) # Blue color


# Conditional formatting to show red font color for negative values
worksheet.conditional_format('E2:E100', {'type': 'cell', 'criteria': '<', 'value': 0, 'format': red_format})
worksheet.conditional_format('F2:F100', {'type': 'cell', 'criteria': '<', 'value': 0, 'format': red_format})
worksheet.conditional_format('H2:H100', {'type': 'cell', 'criteria': '<', 'value': 0, 'format': red_format})
worksheet.conditional_format('I2:I100', {'type': 'cell', 'criteria': '<', 'value': 0, 'format': red_format})


# Conditional formatting to apply background color for specific text
worksheet.conditional_format('B2:B100', {
    'type': 'text',
    'criteria': 'containing',
    'value': 'Laptops',
    'format': pink_bg_format  # Pink color for Laptops
})
worksheet.conditional_format('B2:B100', {
    'type': 'text',
    'criteria': 'containing',
    'value': 'Chairs',
    'format': blue_bg_format  # Blue color for Chairs
})
```

### Alignment Formatting 

'center_format' : Center Alignment

```python
center_format = workbook.add_format({'align': 'center'})

worksheet.set_column('G:G', 18, center_format)
worksheet.set_column('H:H', 18, center_format)
```

### Border Formatting

'thick_border_format': Thick border on all sides
'medium_border_format': Medium border on all sides

```python
thick_border_format = workbook.add_format({'top': 2, 'bottom': 2, 'left': 2, 'right': 2})
medium_border_format = workbook.add_format({'top': 1, 'bottom': 1, 'left': 1, 'right': 1})


# Apply border formatting to non-blank cells
worksheet.conditional_format('A2:J2', {'type': 'no_blanks', 'format': thick_border_format})
worksheet.conditional_format('A3:J100', {'type': 'no_blanks', 'format': medium_border_format})
```

### Combine Multiple Formats

You can write a custom header and apply a custom format that includes multiple formatting features. For example, include formatting for font, alignment, font color, and background color.

```python

header_format = workbook.add_format({
    'bold': True,
    'align': 'left',
    'font_color': '#FFFFFF',
    'bg_color': '#000000'
})

header_format_sales = workbook.add_format({
    'bold': True,
    'align': 'center',
    'font_color': '#FFFFFF',
    'bg_color': '#006400'
})

header_format_units = workbook.add_format({
    'bold': True,
    'align': 'center',
    'font_color': '#FFFFFF',
    'bg_color': '#00008B'
})

# Write custom headers starting from column A
headers = ["Division", "Department", "Category"]
for col_num, value in enumerate(headers):
    worksheet.write(0, col_num, value, header_format)

# Write sales headers starting from column D
headers_sales = ["€Sales L52W", "Sales L52W Change", "Sales L52W % Change"]
start_col_sales = len(headers)  # Start where the previous headers ended
for col_num, value in enumerate(headers_sales, start=start_col_sales):
    worksheet.write(0, col_num, value, header_format_sales)

# Write units headers starting from column G
headers_units = ["Units L52W", "Units L52W Change", "Units L52W % Change"]
start_col_units = start_col_sales + len(headers_sales)  # Start where the sales headers ended
for col_num, value in enumerate(headers_units, start=start_col_units):
    worksheet.write(0, col_num, value, header_format_units)
    
# Write remark headers starting from column J
headers_remark = ["Remark"]
start_col_remark = start_col_units + len(headers_units)  # Start where the units headers ended
for col_num, value in enumerate(headers_remark, start=start_col_remark):
    worksheet.write(0, col_num, value, header_format)

```

### Data Bars for Visualization

Data bars in Excel help visualize the magnitude of values in your data. Here's how you can add data bars to a column using 'data_bar':

```python
# Data bars for percentage change columns
worksheet.conditional_format('F2:F100', {'type': 'data_bar', 'bar_color': '#63C384', 'bar_negative_color': 'red', 'data_bar_2010': True})
worksheet.conditional_format('I2:I100', {'type': 'data_bar', 'bar_color': '#63C384', 'bar_negative_color': 'red', 'data_bar_2010': True})

```

### Apply 3-scale color based on value

Apply a color scale to represent the magnitude of data. Here’s how you can add a color scale to a column using '3_color_scale':

```python
worksheet.conditional_format('D2:D100', {'type': '3_color_scale'})
worksheet.conditional_format('G2:G100', {'type': '3_color_scale'})
```

### Setting Tab Color

'set_tab_color' : Apply color to the tab using a color code.

```python
worksheet.set_tab_color('#31869B') #Blue
```

### Freeze Pane

'freeze_panes' : Freeze Pane at the specified cell

```python
worksheet.freeze_panes('D2')
```

### Apply Autofilter
'autofilter' : Apply autofilter at the specified range

```python
worksheet.autofilter('A1:J1')
```