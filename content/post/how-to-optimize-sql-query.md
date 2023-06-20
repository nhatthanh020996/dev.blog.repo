---
layout:     post
title:      "How to optimize sql query"
description: "Discover practical tips for optimizing network performance, query execution, and scaling hardware in a database system. Enhance efficiency and prepare for growth"
date:    2023-06-14
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Database
    - PostgreSQL
URL: "/how-to-optimize-sql-query/"
categories: [ Tech ]
---

## Optimize network

- Keep connection.

- Create connection pooling.

- Integrate many query in a connection.

## Optimize query execution

- Avoid n+1 problem.

- Using EXPLAIN query to inspect the query plan.

- Indexing.

- Partition.

- Sharding.

## Scale hardware
- Vertical scale (scale up)

- Horizontal scale using Master-Slave architecture (scale out).