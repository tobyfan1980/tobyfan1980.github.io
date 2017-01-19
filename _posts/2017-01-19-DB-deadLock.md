---
layout: post
title:  "DB racing queries -- deadlock"
date:   2017-01-19 14:06:05
categories: Database
tags: Database Deadlock
---

* content
{:toc}

When query something not indexed with lock mode, the performance is not scaled. This is a bad query.

for example:
```
session.query(People).filter(_name='Toby').with_lockmode('update').all()
```

We should split the query with two parts. 
1. query by non-index without lock and get the value of the indexed column
2. query by the result of 1) which is indexed with lock mode.

for example:
```
pids = session.query(People._id).filter(_name='Toby').all()
for id in pids:
    session.query(People).filter(_id=id).with_lockmode('update').first()
```

Consequence: for race condition, there may be multiple sessions all getting the same object in the first query. Thus, there will be duplication operations on the object. We should properly handle this.
