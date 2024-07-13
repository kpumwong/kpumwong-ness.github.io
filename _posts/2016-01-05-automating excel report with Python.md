---
layout: post
title: Automate Excel Reports with SQL & Python - Handling Different Configurations
---

In this chapter, I will demonstrate how to use Python to automate multiple Excel reports based on different configurations. This approach ensures efficiency and consistency when tailoring reports to meet the specific requirements of different clients.

### Example Scenario
Imagine you have created a base report in Excel. This report needs to be shared with five different clients, each requiring minor adjustments to the data structure. The Python code provided below streamlines this process, allowing you to generate all necessary reports with a single execution.

## Step 1 : Setup a master file

This master file should contain data specification for each of the client. In this exmaple this report tell category performance base on the category and channel that each client subscribe, each client has their brand logo of different size which the logo needs to be put in the cover page. Hence, data specification contains Client name, category name to put in Cover Page,  Logo Width (Inches), Logo Height (Inches) and channel names

For example

Client | Category (to put in Cover Page) | Logo Width (Inches) | Logo Height (Inches) | Channel Total | Channel Online | Channel Offline | 
--- | --- | --- | ---| ---| ---| ---
A | Soft Drink |3.42 | 1.24 | Y | Y | Y
B | Meat | 4.13 | 0.79 | Y |  | Y
C | Fruits | 3.84 | 1.1 | Y |  | 
D | Snacking | 2.51 | 1.18 | Y | Y | 
E | HBA | 5.04 | 1.16 | Y | Y | Y


## Step 1 : Setup an SQL code to get 