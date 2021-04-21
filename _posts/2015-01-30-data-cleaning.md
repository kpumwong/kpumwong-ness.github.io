---
layout: post
title: Data cleaning
---
<img src="/images/fulls/02.jpg" class="fit image">This project was about preparing weekly sales data from wholesalers selling various products. The main problem with the data was that it came as multiple datasets, one after the other in ONE csv file. As seen below there ar 2 empty lines between each dataset and 6 rows with info about the next data set.

![png](/images/Clean-wrangle/before.png)

Further the information from the 6 rows between the datasets needed to be kept as columns in the final data set.

Lastly, the sales for each product is recorded in 6 columns due to 6 KPIs, sales value, sales units, transactions, buying customer, promo sales and average price per sales unit. So the number of columns are 6 x [n of products] plus an aggregate of all the products (first 6 columns).
This wide format we want to change to a long format so we only have 8 columns in the end. One column for week, one for product names and the 6 KPIs. Notice there are two header rows (multiindex).

So the approach I took to go from this csv containing nultiple datasets to one big dataset in long format is the following:

1. Locate each dataset (dataframe) and save them seperately in a dictionnary.
2. Loop through each dataframe, extract the info from the 6 headline and add it to the dataframe as 6 new columns
3. Loop through each dataframe again and 

```python
df = df.fillna(0)
```



```python
indexes = df.index[df['col_0'] == 0].tolist()[::2]
```


```python
all_df = {}
all_df['df_1'] = df.iloc[:indexes[0]]
df_num = 2
index_before = indexes[0]

for index in indexes[1:]:
    all_df['df_{}'.format(df_num)] = df.iloc[index_before + 2:index]
    df_num += 1
    index_before = index
```


```python
for df in all_df.keys():
    # Adding header info to values in designated columns
    all_df[df]['Geography'] = all_df[df].iloc[1,0].split(':')[1]
    all_df[df]['Customer Type'] = all_df[df].iloc[2,0].split(':')[1]
    all_df[df]['Purchase Channel'] = all_df[df].iloc[3,0].split(':')[1]
    all_df[df]['National Account'] = all_df[df].iloc[4,0].split(':')[1]
    all_df[df]['Supplier'] = all_df[df].iloc[5,0].split(':')[1]
    all_df[df] = all_df[df][6:]

    # Adding header info column names to the second line as the column names we are going to use
    extra_cols = ['Geography', 'Customer Type', 'Purchase Channel', 'National Account', 'Supplier']
    for item in range(5):
        all_df[df].iloc[1,-5:][item] = extra_cols[item]

    # Copy second row to column names
    all_df[df].columns = all_df[df].iloc[1]

    # Drop second row as it has been copied to the column names
    all_df[df] = all_df[df].drop(all_df[df].index[1])
```


```python
for df in all_df.keys():
    # Splitting the last 5 coloumns from the dataframe
    df_melt = all_df[df].iloc[:, :-5].copy()
    last_five = all_df[df].iloc[:, -5:].copy()

    # Adding the information in the first row to the coloumn names so we can do the melt
    df_melt.columns = df_melt.iloc[0] + 'XXXX' + df_melt.columns
    df_melt.rename(columns={'TimeXXXXTime': 'Time'}, inplace=True)

    # Melting the dataframe but keeping the Time coloumn as is
    df_melt = df_melt.iloc[1:, :].melt(id_vars='Time')

    # Spliting the variable coloumn name on the XXXX and putting the values into Product and Column coloumns
    df_melt['product'] = df_melt['variable'].apply(lambda x : x.split('XXXX')[0]).values
    df_melt['column'] = df_melt['variable'].apply(lambda x : x.split('XXXX')[1]).values

    # Converting the value coloumn to floats so the pivot will run faster
    df_melt['value'] = pd.to_numeric(df_melt['value'])

    # Pivoting the data into the right format
    df_melt = df_melt.pivot_table(index=['Time', 'product'], columns='column', values='value', aggfunc='sum').reset_index()

    # Adding the last 5 coloumns back into the melted and pivoted dataframe
    for column_name, value in last_five.iloc[0].reset_index().values:
        df_melt[column_name] = value
    all_df[df] = df_melt
```


```python
# Stacking all the dataframes into one big dataframe
stacked_df = all_df['df_1']
for df in all_df.keys():
    if df == 'df_1':
        continue
    stacked_df = pd.concat([stacked_df, all_df[df]])
```


```python
stacked_df.shape
```

*OUTPUT:*
```python
(2298296, 13)
```


```python

```

```python

```