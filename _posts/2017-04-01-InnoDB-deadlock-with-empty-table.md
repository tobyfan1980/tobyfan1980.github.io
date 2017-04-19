---
layout: post
title:  "InnoDB deadlock when table is empty"
date:   2017-04-01 11:00:00 -0800
categories:    Database
tags:    MySQL, InnoDB
---

Database deadlock can happen during race condition. This post gives an interesting example. 

When we query a table for update, it will lock the rows in the table, thus other query (for update) will be blocked.

However, if the table is empty, there is no row to be locked, all concurrent transactions can get through and acquire the X lock. As a result, the insert intention of the first transaction will be blocked as another transaction have the X lock; then the insert intention of the second transaction will cause deadlock.

explained in https://dev.mysql.com/doc/refman/5.6/en/innodb-next-key-locking.html

```
"Note that if you use SELECT FOR UPDATE to perform a uniqueness check before an insert, you will get a deadlock for every race condition unless you enable the innodb_locks_unsafe_for_binlog option. A deadlock-free method to check uniqueness is to blindly insert a row into a table with a unique index using INSERT IGNORE, then to check the affected row count."
```
