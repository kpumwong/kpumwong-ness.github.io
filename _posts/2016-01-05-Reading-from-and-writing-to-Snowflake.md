---
layout: post
title: Reading from and writing to Snowflake
---
<!--<img src="/images/fulls/01.jpg" class="fit image">-->
GraphQL endpoints are widespread and a powerful way of interacting with an API compared to traditional REST API endpoints. It allows the user to query all available data from one single endpoint and choose only what is needed.

## Introduction

In this example I will be extracting the newest data using GraphQL.
Also I will show the use of asyncronous execution which basically means the code will wait for the server onthe other end before continuing.
Lastly, the code will show how I am only fetching the newest data every day an