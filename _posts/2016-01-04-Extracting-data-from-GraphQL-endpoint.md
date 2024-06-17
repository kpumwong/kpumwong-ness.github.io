---
layout: post
title: Extracting data from a GraphQL endpoint
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
In this example I will be extracting data from a GraphQL endpoint.
Also, I will show the use of asyncronous execution, enabling the code to wait for the API server before continuing. Lastly, the code will show how I am only fetching only the newest data since the last run and appending this to the database.

## Set up




GraphQL endpoints are widespread and a powerful way of interacting with an API compared to traditional REST API endpoints. It allows the user to query all available data from one single endpoint and choose only what is needed.
