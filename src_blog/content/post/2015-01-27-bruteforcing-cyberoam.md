---
title: Bruteforcing Cyberoam
date: 2015-01-27
---

This post is about how me and [Avijit](https://twitter.com/526avijit) retrieved multiplie cyberoam username/password combinations using python. This also highlights how vulnerable the system is if you are not using a more secure password . Even the default password given to us by the College are being recycled for many years.

### DISCLAIMER
Before you read this, I want to clarify that we do not intend to use our findings in any unethical way. No username-password combinations will be disclosed. The purpose of writing this blog was to notify possible victims to prevent their ID from being used by someone else.

### What is Cyberoam ?
Its a network gateway software which requires students to login to use the internet.

## How we did it ?
I wrote a simple python script which basically sends multiple requests to the server with varying username & password combinations. It checks whether the combination is correct and if it is correct , the combination is stored in a file. I wrote that script to iterate over 2000 users and check 16 passwords, basically sending around 32,000 requests.

### How we got the usernames ?
It was simple. The college keeps a record of the marks obtained by the students with their enrollment number(username) and keeps it in the file-server (which is open to all the students).

Avijit did a little search around and was able to compile a list of all the 1st year students by going through the data available in different department’s servers.

### How we got the passwords ?
We found out that the default passwords given by the college have a pattern, also the same passwords( a set of around 150–300 passwords) were being used every year.
So we asked around and compiled a list of those passwords for our script. We could acquire around 10 passwords out of those.

## Running the script
![A Screenshot of the script in action](/images/bruteforce-cyberoam-shell.png)

Before running the script we didn't have any high hopes, but on executing it, the outcome surprised us. We managed to get **62 combinations**.

Its sad to see that our user IDs are so easy to crack.

## Conclusion

Our College uses Cyberoam. They control and monitor what we see and what we do not on the Internet. They give us a very very crappy speed for a Computer Science College. They block even the most harmless sites like [IMDb](http://www.imdb.com/).

The least they can do is to add some security. I am tired of hearing my other friends praising their college’s internet speed, saying how they can download a movie in minutes , where as I can’t even download online lectures without exceeding my limited quota.
