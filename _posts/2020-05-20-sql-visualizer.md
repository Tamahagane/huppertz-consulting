---
title: "Simple Visualizer for Teradata/t-SQL Queries"
description: "A quick tool to help debugging complicated CTE expressions."
layout: post
toc: false
comments: false
hide: false
search_exclude: false
categories: [tooling, python]
---

# A Simple Visualizer for SQL scripts

A while ago, I was working on a project where the client used an expensive high-performance database cluster in order to do demand forecasting for a German supermarket chain.

What the client did not have:

- Automated testing
- CD/CI-Pipelines
- Coding guidelines
- Documentation

What the client **did** have however, was a codebase of about 30,000 lines of pure T-SQL code and a team that learned "on the job", i.e. had never written a functioning application in this SQL dialect. This resulted in a about 10 scripts of varying length and written in different styles by different people, most of whom had left the project in frustration already by the time I joined.

Combine that with requirements which were subject to change on daily (sometimes hourly) basis, arbitrary restrictions from operations (indexes take up too much space) and being about 100% over budget, it was a recipe for disaster.

Debugging could mean scrolling through a single or multiple scripts each consisting of 3,000 lines of code and dozens of SQL statements and manually recalculating steps along the way. This would of course take hours within which priorities of implementation could (and frequently did) change again.

I was in desperate need for some kind of support and wanted to use [queryscope](https://app.sqldep.com/demo/) to get at least a graphical representation of the queries in question but of course the source code was property of the client and I was not allowed to submit it to an unvetted, unapproved vendor for analysis.

So, I took to recreating parts of queryscope's features from scratch in python to keep all source on premise, the result of which you can see here:


Running the script will yield results like so:

| ![]({{ site.baseurl }}/images/sql_visualizer/result.png) | 
|:--:| 
|**Output of the query below**|

**Caveat**: This code was only tested  with the very specific way the developers on this project wrote their SQL queries, i.e. using many a CTE to create complex temporary tables and then persisting them into intermediary tables, a pattern which is replicated with mock statements in the following SQL snippet:

```sql

WITH cte_category_counts (
    category_id, 
    category_name, 
    product_count
)
AS (
    SELECT 
        c.category_id, 
        c.category_name, 
        COUNT(p.product_id)
    FROM 
        production.products p
        INNER JOIN production.categories c 
            ON c.category_id = p.category_id
    GROUP BY 
        c.category_id, 
        c.category_name
),
cte_category_sales(category_id, sales) AS (
    SELECT    
        p.category_id, 
        SUM(i.quantity * i.list_price * (1 - i.discount))
    FROM    
        sales.order_items i
        INNER JOIN production.products p 
            ON p.product_id = i.product_id
        INNER JOIN sales.orders o 
            ON o.order_id = i.order_id
    WHERE order_status = 4 -- completed
    GROUP BY 
        p.category_id
)
INSERT INTO categories (category_id, category_name, product_count,sales) 
SELECT 
    c.category_id, 
    c.category_name, 
    c.product_count, 
    s.sales
FROM
    cte_category_counts c
    INNER JOIN cte_category_sales s 
        ON s.category_id = c.category_id
ORDER BY 
    c.category_name;



WITH cte_business_division_counts (division_id, counts) AS (
    SELECT    
        d.division_id, 
        SUM(c.counts)
    FROM    
        categories c
        INNER JOIN organization.divisions d 
            ON c.category_id = d.category_id
    GROUP BY 
        d.division_id
)
cte_business_division_sales (division_id, division_name, sales) AS (
    SELECT    
        d.division_id,
        d.division_name, 
        SUM(c.sales)
    FROM    
        categories c
        INNER JOIN organization.divisions d 
            ON c.category_id = d.category_id
    GROUP BY 
        d.division_id
)
INSERT INTO business_divisions (division_id, division_name, product_count,sales) 
SELECT 
    s.division_id, 
    s.division_name,
    s.sales,
    c.counts 
FROM
    cte_business_division_counts c
    INNER JOIN cte_business_division_sales s 
        ON s.division_id = c.division_id
ORDER BY 
    c.division_name;
```

And this here is the source code for the actual visualizer:

```python
import re
import os
from collections import Counter
from graphviz import Digraph
import networkx as nx
# can be used to generate JSON format of graph
# from networkx.readwrite import json_graph

basedir = '.'

filenames = ['test.sql']

with_deletes = False

dot = Digraph(comment='Structure', engine='dot', strict=True)

G = nx.Graph()

# graphically highlight tables and their incoming connections
highlight_nodes = []

# do not draw these
ignore_list = []

# collect all nodes here
complete_list = []

for filename in filenames:
    f = open(os.path.join(basedir, filename))
    f_string = f.read()
    sep = ';'
    f_list = [x+sep for x in f_string.split(sep)]
    f_list = [x.replace("\n", " ") for x in f_list]
    f_list = [x for x in f_list if x.strip() != "COMMIT;"]
    complete_list += f_list


# define regular expressions for SQL statements
re_create = re.compile(
    r"CREATE\s+(?:MULTISET)?\s+TABLE\s+([a-zA-Z0-9_\.]+)\s*,",
    re.MULTILINE | re.IGNORECASE)
re_insert = re.compile(
    r"INSERT\s+INTO\s+([a-zA-Z0-9_\.]+)\s*\(", re.MULTILINE | re.IGNORECASE)
re_from = re.compile(
    r"(?<!DELETE)\s+FROM\s+([a-zA-Z0-9_\.]+)[;\s]+", re.S | re.I)
re_cte = re.compile(
    r"(?:WITH)?\s+([A-Za-z0-9_]+)\s+AS\s?\(", re.MULTILINE | re.IGNORECASE)
re_join = re.compile(
    r"JOIN\s+([A-Za-z0-9_\.]+)\s", re.MULTILINE | re.IGNORECASE)
re_update = re.compile(
    r"UPDATE\s([\w])+\sSET\s[\w\,\'\=_]+", re.MULTILINE | re.IGNORECASE)
re_delete = re.compile(
    r"DELETE\sFROM\s([\d\w\.\'\=_]+.*;)", re.S | re.IGNORECASE)

node_list = []
delete_nodes = []
create_nodes = []

# go through all statements and check if they match a regex
for i, statement in enumerate(complete_list):
    statement = statement.replace("\n", " ")

    to_nodes = []
    from_nodes = []

    for match in re.findall(re_create, statement):
        create_nodes.append(match)

    for match in re.findall(re_delete, statement):
        delete_nodes.append(match)

    for match in set(re.findall(re_insert, statement)):
        to_nodes.append(str.lower(match))

    for match in set(re.findall(re_update, statement)):
        print(match)

    for match, count in Counter(re.findall(re_cte, statement)).items():
        print("%s : %i" % (match, count))

    for match, count in Counter(re.findall(re_from, statement)).items():
        if match not in re.findall(re_cte, statement):
            from_nodes.append(str.lower(match))

    # print(5*'-' + 'Joins (CTEs removed)' + 5*'-')
    for match, count in Counter(re.findall(re_join, statement)).items():
        if match not in re.findall(re_cte, f_string):
            # print("%s : %i" % (match,count))
            from_nodes.append(str.lower(match))

    from_nodes = [x for x in from_nodes if x not in ignore_list]
    to_nodes = [x for x in to_nodes if x not in ignore_list]

    for to_node in to_nodes:
    	# for every node switch between picking colors from the "left" and "right" end 
    	# the spectrum so similar colors do not appear next to each other
        if i % 2 == 0:
            edge_color = str(round(i/float(len(complete_list)), 2))+" 1.0 " +\
               str(round(i/float(len(complete_list)), 2))
        else:
            edge_color = str(round((len(complete_list)-i)/float(len(complete_list)), 2))\
                 + " 1.0 " + str(round((len(complete_list)-i)/float(len(complete_list)), 2))

        if to_node in highlight_nodes:
            dot.node(to_node, color=edge_color, style='filled',
                     fillcolor=edge_color)
        else:
            dot.node(to_node, color=edge_color)

        for from_node in from_nodes:
            dot.node(from_node, shape='box')
            if (to_node in highlight_nodes) or (from_node in highlight_nodes):
                dot.edge(from_node, to_node, color=edge_color, penwidth='3')
            else:
                dot.edge(from_node, to_node, color=edge_color)

# graphically represent deletes within the query
if with_deletes:
    delete_nodes = [str.lower(del_node) for del_node in delete_nodes]
    delete_label = '<<B>DELETES</B><br/>' + '<br/>'.join(delete_nodes) + '>'
    dot.node('DELETES', label=delete_label, shape='box')

# print(5*'-' + 'All participating Tables' + 5*'-')
# for match in set(re.findall(re_from,f_string)+re.findall(re_insert,f_string)+re.findall(re_join,f_string)):
#    if match not in re.findall(re_cte,f_string):
#        print (match)


dot.render('output/'+filename.replace('.', '_')+'.gv', view=True)

# print(dot.source)
# dot_graph = G.read_dot()
# print (json_graph.dumps(dot_graph))

```


As you can see, there are also ways to highlight certain nodes, as well as omitting them from the output. There is also an automatic random assignment of a color to any persistent table and all edges going into the respective node are also color-coded accordingly. Nodes that have no downstream dependents within the query are displayed as an ellipse as opposed to a rectangle.

This whole thing is a quick draft that took me about 4-5 hours to write, but it saved me countless hours in debugging.

This script could also be integrated into a CD/CI-pipeline in order to create a constantly updated visual documentation of the project.