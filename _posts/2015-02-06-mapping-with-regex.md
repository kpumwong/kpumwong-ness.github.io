---
layout: post
title: Mapping with Regex
---
<img src="/images/fulls/01.jpg" class="fit image">This is an example of mapping different values in the results of an encoded questionnaire (4237 rows and 194 columns). In stead of the actualy brand name it only states brand_0014 for example. Also, the answers based on some scale only has numbers where we for example want "Adults over 18", "Teenagers", etc. in stead.

This could have been done with Dataiku as well using a simple "Find and replace" function, however it would have been much more complex and time consuming, as you would have had to do it one column at the time.
Using Python we can easily loop over each column and do the mapping efficiently.

Below you can see the encoded data set:

![png](/images/Mapping-with-Regex/before1.png)
![png](/images/Mapping-with-Regex/before2.png)

Firste we a couple of simple replacements on a couple of the the coloumns refering to the first standard questions.
Also, we remove all the curly brackets and underscores from the dataset.

```python
df_map.replace({'{':'', '}':''}, regex=True, inplace=True)
df_map.replace({'_':''}, regex=True, inplace=True)
df_map['SEX'].replace({'_1':'Male', '_2':'Female'}, regex=True, inplace=True)
df_map['AGEBANDS'].replace({'isinrange_':''}, regex=True, inplace=True)
df_map['CHIEF_FMCG'].replace({'_1':'Almost all the time', '_2':'Half or more than half the time'}, regex=True, inplace=True)
```

Next we create a dictionnarey determining how the brand codes should be mapped. This could also have been stored in a seperate mapping dataset, but it was ok to do it like this in this example.
The dict is bigger below is just a couple of rows. Also, the brand names has been changed to not reveal business sensitive data.

```python
mapping_dict = {'brands_1036':'Aba',
'brands_0024':'Deald',
'brands_0016':'Jack D',
'brands_0023':'Tiger',
'brands_0003':'Maan',
'brands_0001':'Lurian',
'brands_1050':'Tapern',
```

Now we can use this dictionnary map the brand codes in the column names:

```python
df_map.columns = pd.Series(df_map.columns.tolist()).replace({'{':'', '}':'','Brands':'brands', '.slice':''}, regex=True).replace(mapping_dict, regex=True)
```

And now map the all the columns:

```python
for col in df_map.columns:
    if df_map[col].dtype == object:
        df_map[col] = df_map[col].replace(mapping_dict, regex=True)
```

Since the mapping dict was created with underscores in the brand codes (eg. brands_0024), we had to wait with removing these until now:

```python
df_map.replace({'_':''}, regex=True, inplace=True)
```
For mapping the all the answers that were based on a scale and not just a brand, a dictionnary was created for each "area". All the questions related to "familiarity_time" had to be mapped like this for example:

```python
familiarity_time = {'1':'I buy this brand most often',
                    '2':'I buy this brand regularly',
                    '3':'I’ve bought this brand in the past 3 months',
                    '4':'I’ve bought this brand longer ago',
                    '5':'I’ve seen or heard about it but never tried it',
                    '6':'I’ve never seen or heard of it before today'}
```
And all the question relating to "consideration like this:

```python
consideration = {'1':'My first choice',
                 '2':'Would seriously consider',
                 '3':'Might consider',
                 '4':'Would not consider'}
```
There was about 10 different areas in total.

Finally we can loop over all the columns and map the values accordingly:

```python
for col in df_map.columns:
    for topic in topics:
        if topic == 'hhcomp' and topic in col.lower():
            df_map[col] = (df_map[col].str.replace('1', '1x')
                                  .str.replace('2', '2x')
                                  .str.replace('3', '3x')
                                  .str.replace('4', '4x')
                                  .str.replace('5', '5x')
                                  .str.replace('998', '6x')
                                  .str.replace('1x', 'Adults 18 and over')
                                  .str.replace('2x', 'Teenagers 13 to 17')
                                  .str.replace('3x', 'Children 10 to 12')
                                  .str.replace('4x', 'Children 6 to 9')
                                  .str.replace('5x', 'Children under 6')
                                  .str.replace('6x', 'Do not wish to answer')
                          )
        
        elif topic != 'hhcomp' and topic in col.lower():
            df_map[col] = df_map[col].map(eval(topic)).fillna(df_map[col])
        
        else:
            continue
```

The 'hhcomp' coloumn was able to have more than one answer seperated with a ',':

![png](/images/Mapping-with-Regex/before2.png)

Since the answers also had numbers in them, it was not straight forward to replace the values. I decided to replace 1 with 1x, 2 with 2x etc. so there would be no issues with replacing in 'Children 10 to 12' for example.

The rest of the columns were straight forward in mapping based on the different dictionnairies that was created, as can be seen in the "elif" part of the loop.

Below is a snap shoot of the final data set where everything is mapped:

![png](/images/Mapping-with-Regex/after1.png)