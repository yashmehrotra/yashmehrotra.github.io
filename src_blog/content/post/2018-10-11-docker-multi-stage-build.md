---
title: Building minimal docker images using multi-stage builds
date: 2018-10-11
type: post
---

With the advancements in the world of containers, it is now much easier and feasible for you to deploy containerized applications.

Docker is a utility to efficiently create, ship, and run containers. They enable isolation and portability of application. However, one of the challenges of building docker images is to hold down the image size. One of the most effective ways to optimise the image sizes is to use multi stage builds.

It is in everybody's best interest to keep the size of docker images small as:

1. Smaller images are pulled/pushed faster
2. Smaller images take up less disk space

## Multi-stage builds

In 2017, docker introduced multi-stage builds as a way of optimizing the docker images.

> Multi-staged builds are useful where space is a constraint, and whilst it is always better to build small concise containers, it is easy to get carried away trying to shave off a few MBs. The main focus should be improving the workflow.

With multi-stage builds, you use multiple `FROM` statements in your Dockerfile. Each FROM instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image.

Now, we should look at some practical use cases for this

## Deploying applications

Now, let's come to the practical part. How to write Dockerfiles that build minimal container images.

### Golang
So you have a basic web server in Golang written using gin framework.

```
// main.go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080
}
```

Lets write the dockerfile for this

```
FROM golang:latest
COPY . /go/src/github.com/org/helloworld
WORKDIR /go/src/github.com/org/helloworld
RUN go get -u github.com/gin-gonic/gin
ENV CGO_ENABLED 0
RUN go build -o HelloWorld main.go
EXPOSE 8080
ENTRYPOINT ["./HelloWorld"]
```

After building it, the size of the images comes up to 829 MB. You must be wondering why it is so much, after all it is a feature of golang to have static binaries. Well, the reason is this image has a lot of bloatware that is not required while running the golang application. Since it is a compiled binary, even the go package isn't required.

Now, using Mult-stage builds, we'll build a minimal container that only has the software required by the application at runtime.

```
FROM golang:latest as base
COPY . /go/src/github.com/org/helloworld
WORKDIR /go/src/github.com/org/helloworld
RUN go get -u github.com/gin-gonic/gin
ENV CGO_ENABLED 0
RUN go build -o HelloWorld main.go

FROM scratch
COPY --from=base /go/src/github.com/org/helloworld/HelloWorld /usr/bin/HelloWorld
EXPOSE 8080
ENTRYPOINT ["HelloWorld"]
```
The size of this image is just 15.3 MB. This is because in case of golang, we just need the compiled binary to run our application. See how the base image is used to compile the binary and then a new scratch image is created which just has the executable binary we need.

Since golang generates a binary, it is very straightforward to create a minimal image as it does not need anything else at runtime. Now, let's see how we create an image for a python application which dynamically accesses it's libraries at run time.

### Python

For python, let's build a simple Flask application which runs via gunicorn. So, our app.py will be
```
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

and the requirements.txt file will look like
```
Flask==1.0.2
gunicorn==19.9.0
```

We shall start by building a normal docker image, using the following Dockerfile
```
FROM python:3.6-alpine
RUN apk update && apk add build-base

COPY . /code/
WORKDIR /code
RUN pip install -r requirements.txt

ENTRYPOINT ["gunicorn", "-b", "0.0.0.0:8000", "app:app"]
```

Note that we are using the alpine image which itself is lightweight. The size of this image is 242 MB. This size is still small as the number of dependent libraries are less, but for a production application the size of the image can go upto ~400 MB.

Building a multi-stage build is not as direct in python compared to golang. In python, you need your dependencies to be available at runtime. Also, since gunicorn is an executable here, we need to copy that too. This is how the Dockerfile would look like:

```
FROM python:3.6-alpine as base
RUN apk update && apk add build-base

COPY . /code/
WORKDIR /code
RUN pip install -r requirements.txt

FROM python:3.6-alpine

COPY --from=base /code/ /code
COPY --from=base /usr/local/lib/python3.6 /usr/local/lib/python3.6
COPY --from=base /usr/local/bin/gunicorn /usr/local/bin/gunicorn
WORKDIR /code

ENTRYPOINT ["gunicorn", "-b 0.0.0.0:8000", "app:app"]
```

The size of this image is 84.5 MB. You can see that we are just using the libraries and the gunicorn executable from the previous image.

## Takeaway

This is how multi-stage builds stack up against normal builds

| Language | Normal | Multi-stage |
|----------|--------|-------------|
| Python   | 274 MB | 84.5 MB     |
| Golang   | 829 MB | 15.3 MB     |

One importnant thing to keep in mind is that using multi-stage will not impact the build time of your container, the time difference between the build times is negligible

Multi-stage builds are useful where space is a constraint, and whilst it is always better to build small concise containers, it is easy to get carried away trying to shave off a few megabytes. Even though they are great to use, they shouldn't be abused, the effort should always spent be towards improving the workflow.
