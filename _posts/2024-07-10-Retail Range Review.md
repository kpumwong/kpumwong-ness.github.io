---
layout: post
title: Range Review Analysis
---

This post provides an overview of common retail practices and highlights my contributions to the range review project. All customer information is a mockup to ensure data privacy.

## What is retailer annual range review?

The annual range review in retail is a strategic process where retailers evaluate and adjust their product assortments to optimize sales, profitability, and customer satisfaction for the upcoming year. This process involves analyzing various types of data, such as year-on-year performance, current market trends, and customer preferences.

### **My Contributions** 
There are three main parts where I supported this project:
- Category Overview Analysis
- Customer Decision Tree (CDT)
- Product Recommendations

### **Category Overview Analysis**

- **Project Overview:** Provide an overview of category performance year on year.
- **Data Analysis:** Conduct an analysis of transactional data to understand category performance over time, for example, looking at sales drivers, brand performance, customer segmentation, and source of volume analysis.

*Sales Drivers*
<img src="/images/Cate Overview/sales tree ppt.png" alt="sales tree ppt" class="fit image">

*Customer Segmentation Profile*
<img src="/images/Cate Overview/HH Profile.png" alt="HH Profile" class="fit image">

*Source of Volume Analysis*
<img src="/images/Cate Overview/SOV.png" alt="SOV" class="fit image">

- **Customer Insights:** Identify the key factors driving sales growth or decline for the focus timeframe, along with benchmark comparison.
- **Strategic Recommendations:** Provide top-level insights and opportunities to improve category performance which serves as a foundation for further analysis and strategic decision-making.

### **Customer Decision Tree (CDT) Analysis**

- **Project Overview:** The CDT analysis helps in understanding how consumers shop a particular category or sub-category. This involves identifying key decision drivers, grouping products with the same need state, and making strategic recommendations for category management and store layout.
- **Data Analysis:** 
    - Identify the category to focus.
    - Establish which products in the category fulfill the same need by calculating distance scores which show how closely each pair of the products are connected.
    - Perform clustering based on a distance score between the products.

*Dendogram as a result from CDT*
<img src="/images/Cate Overview/CDT.png" alt="CDT" class="fit image">

    - Validate clusters and identify need states by applying business logic to the cluster solutions.

*Example of CDT Interpretation*
<img src="/images/Cate Overview/CDT interpret.png" alt="CDT interpret" class="fit image">

- **Customer Insights:** Identify key decision drivers and patterns in customer behavior using the CDT framework.
- **Strategic Recommendations:** Combining with category overview to provide actionable insights and recommendations by applying CDT insights to Blockogram to optimize product adjacencies and aisle flow. This helps enhance category performance and customer experience.

### **Product Recommendation**

- **Project Overview:** Product recommendation provides the criteria to keep or remove the products from shelf based on sales performance and loyalty metrics, along with providing the substitution products.
- **Data Analysis:**  
    - Define product universe or use the result from CDT.
    - Generate a product flag to flag low-performing or essential items based on consistency.
    - Create product ranking based on sales performance and loyalty metrics for items that pass the consistency review.
    - Calculate walk rate, expected sales loss, and percentage of sales switching to substitutes if delisted.
- **Customer Insights:** Recommend adjustments to the product mix to enhance overall category performance.
- **Strategic Recommendations:** Combining this with CDT results helps review items in the same cluster, leading to strategic decisions on which products to keep, discontinue, introduce, or promote for assortment optimization.
