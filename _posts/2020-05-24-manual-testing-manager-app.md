---
title: "Creating a tool for manual testing in Django"
description: "In absence of a proper testing process, we create a Django app to scale out testing to a manual testers."
layout: post
toc: false
comments: false
hide: false
search_exclude: false
categories: [tooling, python, django, testing]
---

Automated testing is still not as prevalent in software development as one would wish. Testing is apparently very expensive, but experience tells us that the absence of testing is oftentimes even more expensive. Unfortunately, budget deciders do not share these experiences with software developers and therefore decide against the implementation of automatic testing in a project. 

The project I was working on as as project manager had exactly this setup. Manually testing the software artifacts produced by the devs was one of my jobs as was the documentation of the testing results.

This was obivously not an optimal solution due to various reasons:

- as PM I was fairly expensive, junior members would have been more suitable to conduct the user tests
- as the software framework was new to the dev team, a lot of functions had to be refactored after their first implementation, which led to features breaking again and again
- Communication overhead for all the issues slowed down the development of new features
- Just using issues in JIRA was impractical as the constant regression through refactoring would have meant we could never close any issues and user stories and the information fields required for the development workflow were not the same as for the testing workflow

So in order to remedy this problems I needed an application where I could:

- delegate the testing process to junior team members
- repeatedly test the same issues again and again
- document progression and regression permanently in oredr to provide feedback to the dev team
- ensure sufficient test coverage and records about how recently a certain feature was tested

I decided to create a Django app that would help me realise these goals.


## Features

We start with an overview page listing all the issues and their status as well as an aggregate overview over all existing issues faceted according to their priority.

Also in the header there are buttons allowing to choose issues to be tested at random.

| ![]({{ site.baseurl }}/images/test_manager/overview.png) | 
|:--:| 
|**Homepage - high level overview and list of all issues**|

When adding a new issue, it is possible to set a priority and an artifact the issue relates to. Artifacts are configurable in the admin backend of Django.

Images that help to describe the issue can be attached and one or multiple sets of credentials like test user account access can be attached.

| ![]({{ site.baseurl }}/images/test_manager/new_issue.png) | 
|:--:| 
|**Create new issue**|

Once an issue has been created it is possible to add comments, images and set a status of the issue ("okay", "unsure", "cannot test", "not working"). The history of status changes will be recorded on the right so it is easy to visually identify how stable an issue is and how carefully it needs to be tested again.

| ![]({{ site.baseurl }}/images/test_manager/existing_issue.png) | 
|:--:| 
|**Issue and testing status**|

A simple faceted search is pretty much included OOTB with Django and just need a little bit of configuration.


| ![]({{ site.baseurl }}/images/test_manager/search.png) | 
|:--:| 
|**Simple search function**|

A lot of communication overhead goes into sharing credentials for testing, such a login data for the admin backend or test user credentials. So in order to simplify this process, all credentials that can be shared among the testing team without compromising the security of any live environment can be collected and shared on an overview page as well as linked to the specific issues the are connected with. 

| ![]({{ site.baseurl }}/images/test_manager/credentials.png) | 
|:--:| 
|**Credentials overview**|

Last but not least, all comments attached to issues are chronolgically displayed on the comments overview page to gather the most recent feedback on issues for the project manager and allows for this feedback to be realyed to the dev team.

| ![]({{ site.baseurl }}/images/test_manager/comments.png) | 
|:--:| 
|**Comments section**|
