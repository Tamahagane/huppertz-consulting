---
title: "Contents of a Data Science Project Contract"
description: "Matters that should b discussed and documented when setting up a data science project contract."
layout: post
toc: true
comments: false
hide: false
search_exclude: false
categories: [project management]
---

When you are facing your first data science project and the accompanying negotiation of a project contract, it can be a little bit overwhelming and it's easy to forget to pen down some important issues.  In this article I will provide a list of some of these issues that should be defined in a data science project contract - and a few things that should not.

## Form and Content of the Result

To manage expectations on the client side, define the deliverables precisely, such as 

- the goal
    - identify factors on the website that influence the average shopping cart volume
    - decide if any of the 5 marketing campaigns of last year had a measurable impact on the revenue and if so, how much
    - analyze if the customer retention measures in the call center are generating positive returns
   - ...


- the format 
    - Powerpoint
    - Word
    - notebook
    - Airflow workflow
    - key-ready application
    - daily mail reports
    - ...


- the content
    - one off analysis
    - model for continuous evaluation
    - table or visualization
    - ...


- the timeframe of data to be analyzed and - if applicable - the timeframes for forecasts
    

- the timing for delivery a.k.a. the deadline

## Wrangling the input data

This is one of the trickiest ones, because part of why you are negotiating a data science project right now might be that your client does not have a very good overview over his own data landscape. But to the extent of your client's knowledge do define the rough size and shape of the data provided to you and their accessibility:

- millions of rows
- GB size
- remote access or on site only
- DB/DWH downtimes
- ETL processes blocking you from access (my last project DWH was down for one hour after lunch break for ETL)
- ...

Also, include in your contract a passage that allows you to re-negotiate or get out of the contract after having an in-depth look at the data. Sometimes it turns out the data is not what is was expected to be in quality and/or size and this might make your project completely infeasible.

## You will be paid for your time, not the impact of your results

There is no guarantee that you will find any result of substance and economical impact in an exploratory data science project. If you want to explore, e.g.,  which factors influence the shopping cart value of a customer, it is entirely possible that the input data set does not contain any answers to your question. Any impactful factors that you do happen to identify might already be at their optimum or cannot be adjusted for reasons outside of the scope of your analysis. Mention in your contract that you do not owe any specific result and that your compensation is for the time and effort of the process, not the efficacy of your results. 

## Do not present preliminary results, avoid presenting your results many times over

Do not agree on meetings to present intermediate results without giving great thought to it. Preparing a shiny presentation to placate a sceptical customer after a week or two of work will take focus away from your final result. Preparing the presentation of the result can take between 30 - 50 % of the overall man hours of a project, so if you want to present results after two weeks, you will barely have a week to do the actual work, if at all. Also this can backfire if you decide to pursue a different angle to solve a problem after presenting preliminary results on a different route.

Oftentimes, the client simply wants to be kept in the loop to avoid wasting the money they invest in you. You can offer to communicate regular status updates via Email or chat and you should keep those memos neutral in terms of results to avoid eating your words later, e.g. write "Explored - among other variables - the customers age in relation to the revenue" instead of "Looks like older customer spend more money in your shop".


Once you find interesting results worthy of presentation, there might be a cascade of "oh, the Head of Marketing needs to see that", "oh, the COO needs to see that", "oh, the CEO needs to see that". This of course leads to you spending more time than expected on the project, because you do not present the results once but 3 or 4 times. Stating in your contract that the results will be presented once and once only will give you leverage, should the client ask you to do more than one. 

## Access to subject matter experts

While I suggest keeping the C-Level of your client on a "need to know"-basis, I do recommend to keep in close contact with the client's side subject matter experts as you will have questions that need answers. Everyone needs a vacation once in a while, though. It is most inconvenient if your contacts within the client's organization are taking that long overdue holiday in the middle of your project. So, make sure you know when they will be present and that they are not constantly beleagured with meetings, but will actually be available to discuss your ideas and findings.

## Hardware provided or BYOD

If your client is providing you with hardware and you don't bring your own device, make sure it's fast and has sufficient RAM. The minimum nowadays should be 16 GB RAM for batch processing of any significant amount of data. Of course the price of RAM is minute compared to the overall budget of the average project and enough RAM to avoid swapping to disk can save you hours if not days over the course of a larger project.

## Third Party Tools

If you want to rely on a 3rd party tools - especially when data is uploaded to their servers - make sure you are allowed to do so and it's within the infosec guidelines of your client.

## Minimize PII

If anyhow possible do not accept any personally identifiable information. You don't want to be involved if at any point there is a data breach and customer information get's leaked. 


## Checklist

- Add a disclaimer about the exploratory nature of the data science project if applicable
- Define a predetermined breaking point after initial look into the data 
- Not agreed on any preliminary result meetings
- Form and content of the final result is defined
- How many times the results will be presented is defined 
- Your compensation for your efforts
- Which hardware will be provided (if applicable)
- Define which third party tools you want to use and get them signed off by infosec
- Define which PII is excluded from the data provided to you
- Document size, shape, and scope of data that will be provided to you
- Access to subject matter experts is defined and any planned absences have been communicated
