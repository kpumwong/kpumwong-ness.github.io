---
layout: post
title: Data cleaning
---
<img src="/images/fulls/02.jpg" class="fit image">This project was about preparing weekly sales data from wholesalers selling various products. The main problem with the data was that it came as multiple datasets, one after the other in ONE csv file. Between each dataset there was 2 empty lines and 6 rows with info about the data set.

![png](/images/Clean-wrangle/before.png)

Further the information from the 6 rows between the datasets needed to be kept as columns in the final data set.

Latly, the sales for each product is recorded in 6 columns due to 6 KPIs, sales value, sales units, transactions, buying customer, promo sales and average price per sales unit. So the number of columns are 6 x [n of products] plus an aggregate of all the products (first 6 columns).
This wide format we want to change to a long format so we only have 8 columns in the end. One column for week, one for product names and the 6 KPIs.