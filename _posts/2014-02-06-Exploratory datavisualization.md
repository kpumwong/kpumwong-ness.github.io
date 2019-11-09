---

layout: post
title: Exploratory data visualization

---

<img src="/images/fulls/02.jpg" class="fit image">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Morbi cursus fringilla dui. Donec laoreet maximus elit. Mauris ullamcorper condimentum lobortis. Donec maximus, mi vitae iaculis placerat, ligula mi maximus lectus, ac varius mi nulla id neque. Quisque porta eros nisi, eu fringilla urna dictum vitae. 

```python
import pandas as pd
import matplotlib.pyplot as plt
%matplotlib inline
```


```python
recent_grads = pd.read_csv("recent-grads.csv")
recent_grads.iloc[0]
```




    Rank                                        1
    Major_code                               2419
    Major                   PETROLEUM ENGINEERING
    Total                                    2339
    Men                                      2057
    Women                                     282
    Major_category                    Engineering
    ShareWomen                           0.120564
    Sample_size                                36
    Employed                                 1976
    Full_time                                1849
    Part_time                                 270
    Full_time_year_round                     1207
    Unemployed                                 37
    Unemployment_rate                   0.0183805
    Median                                 110000
    P25th                                   95000
    P75th                                  125000
    College_jobs                             1534
    Non_college_jobs                          364
    Low_wage_jobs                             193
    Name: 0, dtype: object




```python
recent_grads.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Rank</th>
      <th>Major_code</th>
      <th>Major</th>
      <th>Total</th>
      <th>Men</th>
      <th>Women</th>
      <th>Major_category</th>
      <th>ShareWomen</th>
      <th>Sample_size</th>
      <th>Employed</th>
      <th>...</th>
      <th>Part_time</th>
      <th>Full_time_year_round</th>
      <th>Unemployed</th>
      <th>Unemployment_rate</th>
      <th>Median</th>
      <th>P25th</th>
      <th>P75th</th>
      <th>College_jobs</th>
      <th>Non_college_jobs</th>
      <th>Low_wage_jobs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2419</td>
      <td>PETROLEUM ENGINEERING</td>
      <td>2339.0</td>
      <td>2057.0</td>
      <td>282.0</td>
      <td>Engineering</td>
      <td>0.120564</td>
      <td>36</td>
      <td>1976</td>
      <td>...</td>
      <td>270</td>
      <td>1207</td>
      <td>37</td>
      <td>0.018381</td>
      <td>110000</td>
      <td>95000</td>
      <td>125000</td>
      <td>1534</td>
      <td>364</td>
      <td>193</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2416</td>
      <td>MINING AND MINERAL ENGINEERING</td>
      <td>756.0</td>
      <td>679.0</td>
      <td>77.0</td>
      <td>Engineering</td>
      <td>0.101852</td>
      <td>7</td>
      <td>640</td>
      <td>...</td>
      <td>170</td>
      <td>388</td>
      <td>85</td>
      <td>0.117241</td>
      <td>75000</td>
      <td>55000</td>
      <td>90000</td>
      <td>350</td>
      <td>257</td>
      <td>50</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2415</td>
      <td>METALLURGICAL ENGINEERING</td>
      <td>856.0</td>
      <td>725.0</td>
      <td>131.0</td>
      <td>Engineering</td>
      <td>0.153037</td>
      <td>3</td>
      <td>648</td>
      <td>...</td>
      <td>133</td>
      <td>340</td>
      <td>16</td>
      <td>0.024096</td>
      <td>73000</td>
      <td>50000</td>
      <td>105000</td>
      <td>456</td>
      <td>176</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2417</td>
      <td>NAVAL ARCHITECTURE AND MARINE ENGINEERING</td>
      <td>1258.0</td>
      <td>1123.0</td>
      <td>135.0</td>
      <td>Engineering</td>
      <td>0.107313</td>
      <td>16</td>
      <td>758</td>
      <td>...</td>
      <td>150</td>
      <td>692</td>
      <td>40</td>
      <td>0.050125</td>
      <td>70000</td>
      <td>43000</td>
      <td>80000</td>
      <td>529</td>
      <td>102</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>2405</td>
      <td>CHEMICAL ENGINEERING</td>
      <td>32260.0</td>
      <td>21239.0</td>
      <td>11021.0</td>
      <td>Engineering</td>
      <td>0.341631</td>
      <td>289</td>
      <td>25694</td>
      <td>...</td>
      <td>5180</td>
      <td>16697</td>
      <td>1672</td>
      <td>0.061098</td>
      <td>65000</td>
      <td>50000</td>
      <td>75000</td>
      <td>18314</td>
      <td>4440</td>
      <td>972</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>




```python
recent_grads.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Rank</th>
      <th>Major_code</th>
      <th>Major</th>
      <th>Total</th>
      <th>Men</th>
      <th>Women</th>
      <th>Major_category</th>
      <th>ShareWomen</th>
      <th>Sample_size</th>
      <th>Employed</th>
      <th>...</th>
      <th>Part_time</th>
      <th>Full_time_year_round</th>
      <th>Unemployed</th>
      <th>Unemployment_rate</th>
      <th>Median</th>
      <th>P25th</th>
      <th>P75th</th>
      <th>College_jobs</th>
      <th>Non_college_jobs</th>
      <th>Low_wage_jobs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>168</th>
      <td>169</td>
      <td>3609</td>
      <td>ZOOLOGY</td>
      <td>8409.0</td>
      <td>3050.0</td>
      <td>5359.0</td>
      <td>Biology &amp; Life Science</td>
      <td>0.637293</td>
      <td>47</td>
      <td>6259</td>
      <td>...</td>
      <td>2190</td>
      <td>3602</td>
      <td>304</td>
      <td>0.046320</td>
      <td>26000</td>
      <td>20000</td>
      <td>39000</td>
      <td>2771</td>
      <td>2947</td>
      <td>743</td>
    </tr>
    <tr>
      <th>169</th>
      <td>170</td>
      <td>5201</td>
      <td>EDUCATIONAL PSYCHOLOGY</td>
      <td>2854.0</td>
      <td>522.0</td>
      <td>2332.0</td>
      <td>Psychology &amp; Social Work</td>
      <td>0.817099</td>
      <td>7</td>
      <td>2125</td>
      <td>...</td>
      <td>572</td>
      <td>1211</td>
      <td>148</td>
      <td>0.065112</td>
      <td>25000</td>
      <td>24000</td>
      <td>34000</td>
      <td>1488</td>
      <td>615</td>
      <td>82</td>
    </tr>
    <tr>
      <th>170</th>
      <td>171</td>
      <td>5202</td>
      <td>CLINICAL PSYCHOLOGY</td>
      <td>2838.0</td>
      <td>568.0</td>
      <td>2270.0</td>
      <td>Psychology &amp; Social Work</td>
      <td>0.799859</td>
      <td>13</td>
      <td>2101</td>
      <td>...</td>
      <td>648</td>
      <td>1293</td>
      <td>368</td>
      <td>0.149048</td>
      <td>25000</td>
      <td>25000</td>
      <td>40000</td>
      <td>986</td>
      <td>870</td>
      <td>622</td>
    </tr>
    <tr>
      <th>171</th>
      <td>172</td>
      <td>5203</td>
      <td>COUNSELING PSYCHOLOGY</td>
      <td>4626.0</td>
      <td>931.0</td>
      <td>3695.0</td>
      <td>Psychology &amp; Social Work</td>
      <td>0.798746</td>
      <td>21</td>
      <td>3777</td>
      <td>...</td>
      <td>965</td>
      <td>2738</td>
      <td>214</td>
      <td>0.053621</td>
      <td>23400</td>
      <td>19200</td>
      <td>26000</td>
      <td>2403</td>
      <td>1245</td>
      <td>308</td>
    </tr>
    <tr>
      <th>172</th>
      <td>173</td>
      <td>3501</td>
      <td>LIBRARY SCIENCE</td>
      <td>1098.0</td>
      <td>134.0</td>
      <td>964.0</td>
      <td>Education</td>
      <td>0.877960</td>
      <td>2</td>
      <td>742</td>
      <td>...</td>
      <td>237</td>
      <td>410</td>
      <td>87</td>
      <td>0.104946</td>
      <td>22000</td>
      <td>20000</td>
      <td>22000</td>
      <td>288</td>
      <td>338</td>
      <td>192</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 21 columns</p>
</div>




```python
recent_grads = recent_grads.dropna()
```


```python
recent_grads.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Rank</th>
      <th>Major_code</th>
      <th>Total</th>
      <th>Men</th>
      <th>Women</th>
      <th>ShareWomen</th>
      <th>Sample_size</th>
      <th>Employed</th>
      <th>Full_time</th>
      <th>Part_time</th>
      <th>Full_time_year_round</th>
      <th>Unemployed</th>
      <th>Unemployment_rate</th>
      <th>Median</th>
      <th>P25th</th>
      <th>P75th</th>
      <th>College_jobs</th>
      <th>Non_college_jobs</th>
      <th>Low_wage_jobs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.00000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
      <td>172.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>87.377907</td>
      <td>3895.953488</td>
      <td>39370.081395</td>
      <td>16723.406977</td>
      <td>22646.674419</td>
      <td>0.522223</td>
      <td>357.941860</td>
      <td>31355.80814</td>
      <td>26165.767442</td>
      <td>8877.232558</td>
      <td>19798.843023</td>
      <td>2428.412791</td>
      <td>0.068024</td>
      <td>40076.744186</td>
      <td>29486.918605</td>
      <td>51386.627907</td>
      <td>12387.401163</td>
      <td>13354.325581</td>
      <td>3878.633721</td>
    </tr>
    <tr>
      <th>std</th>
      <td>49.983181</td>
      <td>1679.240095</td>
      <td>63483.491009</td>
      <td>28122.433474</td>
      <td>41057.330740</td>
      <td>0.231205</td>
      <td>619.680419</td>
      <td>50777.42865</td>
      <td>42957.122320</td>
      <td>14679.038729</td>
      <td>33229.227514</td>
      <td>4121.730452</td>
      <td>0.030340</td>
      <td>11461.388773</td>
      <td>9190.769927</td>
      <td>14882.278650</td>
      <td>21344.967522</td>
      <td>23841.326605</td>
      <td>6960.467621</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>1100.000000</td>
      <td>124.000000</td>
      <td>119.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>0.00000</td>
      <td>111.000000</td>
      <td>0.000000</td>
      <td>111.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>22000.000000</td>
      <td>18500.000000</td>
      <td>22000.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>44.750000</td>
      <td>2403.750000</td>
      <td>4549.750000</td>
      <td>2177.500000</td>
      <td>1778.250000</td>
      <td>0.336026</td>
      <td>42.000000</td>
      <td>3734.75000</td>
      <td>3181.000000</td>
      <td>1013.750000</td>
      <td>2474.750000</td>
      <td>299.500000</td>
      <td>0.050261</td>
      <td>33000.000000</td>
      <td>24000.000000</td>
      <td>41750.000000</td>
      <td>1744.750000</td>
      <td>1594.000000</td>
      <td>336.750000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>87.500000</td>
      <td>3608.500000</td>
      <td>15104.000000</td>
      <td>5434.000000</td>
      <td>8386.500000</td>
      <td>0.534024</td>
      <td>131.000000</td>
      <td>12031.50000</td>
      <td>10073.500000</td>
      <td>3332.500000</td>
      <td>7436.500000</td>
      <td>905.000000</td>
      <td>0.067544</td>
      <td>36000.000000</td>
      <td>27000.000000</td>
      <td>47000.000000</td>
      <td>4467.500000</td>
      <td>4603.500000</td>
      <td>1238.500000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>130.250000</td>
      <td>5503.250000</td>
      <td>38909.750000</td>
      <td>14631.000000</td>
      <td>22553.750000</td>
      <td>0.703299</td>
      <td>339.000000</td>
      <td>31701.25000</td>
      <td>25447.250000</td>
      <td>9981.000000</td>
      <td>17674.750000</td>
      <td>2397.000000</td>
      <td>0.087247</td>
      <td>45000.000000</td>
      <td>33250.000000</td>
      <td>58500.000000</td>
      <td>14595.750000</td>
      <td>11791.750000</td>
      <td>3496.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>173.000000</td>
      <td>6403.000000</td>
      <td>393735.000000</td>
      <td>173809.000000</td>
      <td>307087.000000</td>
      <td>0.968954</td>
      <td>4212.000000</td>
      <td>307933.00000</td>
      <td>251540.000000</td>
      <td>115172.000000</td>
      <td>199897.000000</td>
      <td>28169.000000</td>
      <td>0.177226</td>
      <td>110000.000000</td>
      <td>95000.000000</td>
      <td>125000.000000</td>
      <td>151643.000000</td>
      <td>148395.000000</td>
      <td>48207.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
cleaned_data_count = print(len(recent_grads))
```

    172



```python
ax = recent_grads.plot(x='Sample_size', y='Employed', kind='scatter')
ax.set_title('Employed vs. Sample_size')
```




    Text(0.5, 1.0, 'Employed vs. Sample_size')




![png](/images/ED/output_7_1.png)



```python
ax = recent_grads.plot(x='Sample_size', y='Median', kind='scatter')
ax.set_title('Median vs. Sample_size')
```




    Text(0.5, 1.0, 'Median vs. Sample_size')




![png](/images/ED/output_8_1.png)



```python
ax = recent_grads.plot(x='Sample_size', y='Unemployment_rate', kind='scatter')
ax.set_title('Unemployment_rate vs. Sample_size')
```




    Text(0.5, 1.0, 'Unemployment_rate vs. Sample_size')




![png](/images/ED/output_9_1.png)



```python
ax = recent_grads.plot(x='Full_time', y='Median', kind='scatter')
ax.set_title('Full_time vs. Median')
```




    Text(0.5, 1.0, 'Full_time vs. Median')




![png](/images/ED/output_10_1.png)



```python
ax = recent_grads.plot(x='ShareWomen', y='Unemployment_rate', kind='scatter')
ax.set_title('ShareWomen vs. Unemployment_rate')
```




    Text(0.5, 1.0, 'ShareWomen vs. Unemployment_rate')




![png](/images/ED/output_11_1.png)



```python
ax = recent_grads.plot(x='Men', y='Median', kind='scatter')
ax.set_title('Median vs. Men')
```




    Text(0.5, 1.0, 'Median vs. Men')




![png](/images/ED/output_12_1.png)



```python
ax = recent_grads.plot(x='Women', y='Median', kind='scatter')
ax.set_title('Women vs. Median')
```




    Text(0.5, 1.0, 'Women vs. Median')




![png](/images/ED/output_13_1.png)


Do students in more popular majors make more money?


```python
ax = recent_grads.plot(x='Total', y='Median', kind='scatter')
ax.set_title('Total vs. Median')
```




    Text(0.5, 1.0, 'Total vs. Median')




![png](/images/ED/output_15_1.png)


No does not seem like major with more student attending earn more money afterwards judging from the plot of Median and Men/Women.

Do students that majored in subjects that were majority female make more money?



```python
ax = recent_grads.plot(x='ShareWomen', y='Median', kind='scatter')
ax.set_title('ShareWomen vs. Median')
```




    Text(0.5, 1.0, 'ShareWomen vs. Median')




![png](/images/ED/output_18_1.png)


No. It seems the higher the share of women are in a major the median drops proportionally.

Is there any link between the number of full-time employees and median salary?


```python
ax = recent_grads.plot(x='Full_time', y='Median', kind='scatter')
ax.set_title('Full_time vs. Median')
```




    Text(0.5, 1.0, 'Full_time vs. Median')




![png](/images/ED/output_21_1.png)


A slight downward trend. The more full_time employess the less the salary is.

# Pandas Histograms


```python
recent_grads['Sample_size'].hist(bins=25, range=(0,5000))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11ee30be0>




![png](/images/ED/output_24_1.png)



```python
recent_grads['Median'].hist(bins=100, range=(22000,80000))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11eff9588>




![png](/images/ED/output_25_1.png)



```python
recent_grads['Employed'].hist(bins=50, range=(0,308000))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f187080>




![png](/images/ED/output_26_1.png)



```python
recent_grads['Full_time'].hist(bins=50, range=(11,252000))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f316dd8>




![png](/images/ED/output_27_1.png)



```python
recent_grads['ShareWomen'].hist(bins=2, range=(0,1))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11ef10eb8>




![png](/images/ED/output_28_1.png)



```python
recent_grads['Unemployment_rate'].hist(bins=50, range=(0,0.2))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f524748>




![png](/images/ED/output_29_1.png)



```python
recent_grads['Men'].hist(bins=100, range=(124,150000))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f65edd8>




![png](/images/ED/output_30_1.png)



```python
recent_grads['Women'].hist(bins=100, range=(0,150000))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11f816390>




![png](/images/ED/output_31_1.png)


What percent of majors are predominantly male? Predominantly female?

Looking at ShareWomen there are about 75 out of 172 majors that have more men than women. That adds up to about 43,5%

What's the most common median salary range?

Looking at Median the biggest spike is around 35.000 dollers.

# Scatter Matrix


```python
from pandas.plotting import scatter_matrix
```


```python
scatter_matrix(recent_grads[['Sample_size', 'Median']], figsize=(10,10))
```




    array([[<matplotlib.axes._subplots.AxesSubplot object at 0x11f996b00>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11fa1a438>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x11faad8d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11fad5e48>]],
          dtype=object)




![png](/images/ED/output_36_1.png)



```python
scatter_matrix(recent_grads[['Sample_size', 'Median', 'Unemployment_rate']], figsize=(15,15))
```




    array([[<matplotlib.axes._subplots.AxesSubplot object at 0x11fbd46a0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11fdfdc18>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ffb51d0>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x11ffdb748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x120005cc0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x120034278>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x12005d7f0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x120085da0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x120085dd8>]],
          dtype=object)




![png](/images/ED/output_37_1.png)


# Bar Plots


```python
recent_grads[:10].plot.bar(x='Major', y='ShareWomen', legend=False)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x12025b4e0>




![png](/images/ED/output_39_1.png)



```python
recent_grads[163:].plot.bar(x='Major', y='ShareWomen', legend=False)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x120aa3e80>




![png](/images/ED/output_40_1.png)



```python
recent_grads[:10].plot.bar(x='Major', y='Unemployment_rate', legend=False)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x120ae6c18>




![png](/images/ED/output_41_1.png)



```python
recent_grads[163:].plot.bar(x='Major', y=['Men', 'Women'], stacked=True, legend=False)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x12180ae10>




![png](/images/ED/output_42_1.png)



```python
recent_grads.sort_values("Men", ascending=False).plot.bar(x='Major', y=['Men', 'Women'], stacked=True, legend=True, figsize=(40,15), color=("dodgerblue", "hotpink"))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x132bed208>




![png](/images/ED/output_43_1.png)



```python
my_plot = recent_grads.sort_values("Women", ascending=False).plot.bar(x='Major', y=['Women', 'Men'], stacked=True, legend=True, figsize=(40,15), color=("hotpink","dodgerblue"))
```


![png](/images/ED/output_44_0.png)



```python

```
