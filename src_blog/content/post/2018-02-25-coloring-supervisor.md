---
title: Coloring supervisor
date: 2018-02-25
type: post
---

### What is supervisor

[Supervisor](http://supervisord.org/) is a client/server system that allows its users to monitor and control a number of processes on UNIX-like operating systems.

### Why I colored it

I use supervisor a lot (the organization I work for uses supervisor a lot), so this meant, time to time I had to do `supervisorctl status` and read the output.

![Current supervisor output](/images/supervisor-old.png)

I was always jealous with the JS folks as they had their **pm2**[link it] which made things colorful and pretty by default.

![Current pm2 output](/images/pm2.png)

So, this weekend, I spent around 1 hour to do justice by supervisor.

I used `tabulate` and `crayons` to convert the output in a tabular format and color it respectively. I also used `psutil` to get CPU and memory usage as well. So, here is the final outcome.

![Current supervisor output](/images/supervisor-new.png)

### How to use it

`pip install git+https://github.com/yashmehrotra/supervisor.git`

