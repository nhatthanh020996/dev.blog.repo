---
layout:     post
title:      "Isolation property in ACID"
description: "Discover read phenomena and isolation levels in PostgreSQL databases, ensuring data consistency and concurrency control."
date:    2023-06-13
author:     "Admin"
image: "/img/posts/isolation/title_image.jpeg"
tags:
    - Database
    - PostgreSQL
URL: "/isolation/"
categories: [ Tech ]
---
## Read Phenomena

- Dirty reads: if you are in an inflight transaction, you can read something that some other transaction has written but didn’t really commit yet.
- Non-repeatable reads: if you read a row once, then read it again, the second read could see different data if another transaction has modified it.

- Phantom reads: if you run a query twice, the second query could see different rows if another transaction has inserted or deleted them.

- Lost updates: if two transactions try to update the same row, the second transaction might overwrite the changes made by the first transaction, causing the first transaction to lose its changes.

## Isolation Levels
- Read Uncommitted

- Read Committed: The transaction can see the rows that have been committed by other transactions.

- Repeatable Read: The transaction ensures that when a query reads a row, that row will remain unchanged while it's running.

- Snapshot: Each query in a transaction only sees changes that have been committed up to the start of the transaction. It’s like a snapshot version of the database at that moment.

- Serializable: Transactions are run as if they are serialized one after the other.
## Isolation Level vs Read Phenomena

## Isolation in PostgreSQL