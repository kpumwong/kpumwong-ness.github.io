---
layout: post
title: Mapping with Regex
---
<img src="/images/fulls/01.jpg" class="fit image">This is an example of mapping different values in the results of an encoded questionnaire. In stead of the actualy brand name it only states brand_0014 for example. Also, the answers based on some scale only has numbers where we for example want "Adults over 18", "Teenagers", etc. in stead.

This could have been done with Dataiku as well using a simple Find and replace function, however it would have been much more complex and time consuming, as you would have had to do it one column at the time.
Using Python we can easily search across the whole dataset and do the mapping.

