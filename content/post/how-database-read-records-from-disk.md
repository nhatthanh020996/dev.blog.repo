---
layout:     post
title:      "How database read records from disk"
description: "Isolation là tính chất bị underated nhất trong ACID, trong bài viết này
mình xin trình bày rõ hơn về tính chất này cũng như đưa ra các trường hợp cần setup đúng
isolation level trong thực tế"
date:    2021-11-04T12:00:00
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Database
    - PostgreSQL
    - Pooling
URL: "/how-database-read-records-from-disk/"
categories: [ Tech ]
---

# Dadabase disk page