---
layout: post
title:  "C++ vector copy"
date:   2019-03-21 12:00:00 -0800
categories:    Programming
tags:    C++
---

we can use brace to do copy. e.g.
```
std::vector v2 = {v1.begin(), v1.end()}
```

the performace is 100 faster than iteration. Then following code just copies 10000 items.

```
  std::vector<int> vv = {};
  int total = 10000;

  auto start = std::chrono::steady_clock::now();
  for(int ii=0; ii<total; ii++){
    vv.push_back(ii*2);
  }

  auto time1 = std::chrono::steady_clock::now();

  std::vector<int> l = {vv.begin(), vv.end()};

  auto time2 = std::chrono::steady_clock::now();
  
  std::chrono::duration<double> elapsed_seconds = time1 - start;

  INFO("pushback %d cost %f", vv.size(), elapsed_seconds.count());

  std::chrono::duration<double> elapsed_seconds2 = time2 - time1;

  INFO("copy constructor %d cost %f", l.size(), elapsed_seconds2.count());
  
```

the output is 
```
[INFO] pushback 10000 cost 0.000278
[INFO] copy constructor 10000 cost 0.000002
```

