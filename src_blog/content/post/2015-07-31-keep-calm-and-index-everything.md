---
title: Keep Calm and Index Everything
date: 2015-07-31
---

This post was originally written on lambda.grofers.com


Currently at Grofers we are growing rapidly. And with this rapid-growth comes a huge amount of data. As we all know the (not so) famous saying:

> With great amount of data comes an itching need to analyse it.

As the size of our data is growing rapidly, we have to analyse the trends that push our product forward.

We use **PostgreSQL** as our primary datastore. We store our data in a pretty normalised way and for doing analysis, we have to perform various aggregations. Since our data is very normalised, we have to do a lot of JOINS for performing the level of aggregations we need which were not RDS friendly.

So we decided to search for alternatives. After pondering over various alternatives , we finally decided that **elasticsearch** will solve most of our problems.

It was decided then, **Elasticsearch** it is. But then came a bigger problem, how can we index so much data onto **Elasticsearch** and that too **realtime**.

We didn’t want to do this just by throwing more servers to the problem. So first we thought of using **celery**.

The flow using **celery** was:

1. On every post-save, we would create an elasticsearch document.
2. We would index that document using an asynchronous task through celery.
3. This would go on and on for all the data we recieved.

But there were a few problems:
* Our elasticsearch kept timing out due to the heavy load.
* This process was also heavy on the database.

After that we decided to use something other than **celery** for our data indexing.

## Hello Gearman
**Gearman** is an application framework that allows you to hand over heavy tasks to different workers.

Gearman has 3 important parts:
* Client Server
* Job/Worker Server
* Daemon

The client server would send data to the job server through its daemon. Then that job server would perform those tasks. The tasks can be done both synchronously and asynchronously.

In our scenario, the client server was our API which was written in **python**. We then built our Job Server around **node.js** .

The flow was pretty much awesome:

1. We would send the `id` and `job_name` to the **gearman** daemon.
2. The daemon would pass on the data to our job server.
3. Our job server would query the required data from our read replica.
4. It will then create a document with that data and indexes it on **elasticsearch** using it’s bulk API.

Now we are processing data at an incredible pace and our analytics team is in the process of generating great outcome.

We used gearman for asynchronous tasks, but it can be used in many ways. It can also be used as an alternative to **Apache Thrift**. You can push your performance critical code to other languages like **C++** or **Golang**.

To conclude all my thoughts, the indexing problem was indeed a good challenge and gearman was a great solution.
