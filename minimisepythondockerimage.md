# Minimizing python docker images

###### tags: `blogs`

During the transition to a microservice based approach at Qualislabs I saw new faces joining my team. There was definitely a learning curve involved and generally it reduces the effectiveness of the team. A newcomer should not feel stranded and burdened with a lot to work with.


## Dependency and Environment Hell
Originally we had a monolithic application either in python or node and I must say we were able to debug and fix issues easily even on each others machine.

However as our number increased, the number of differing environments became an issue which meant that every machine had to have the correct dependencies and environment.


## 12 Factor Applications

If you have worked on making applications cloud friendly then you may have encountered the [12 factors](https://12factor.net/). We were trying to ensure that every service within our application adhered to all of these factors in order to easily scale and deploy our services with very little to no manual effort.

## The Issue

Setting appropriate dependencies and environment variables for all members was becoming a bit of a burden and there were times when tests were run against production resources. This was due to developers forgetting to set environment variables back to development values.

## Docker to our rescue

I later found out about docker. I must admit it felt like love at first sight. With containerization we were able to overcome most of the challenges we had but still missed our former days of monoliths. Developers now only had to build their code with a well documented README file then I would later do the system intergration on Friday evening. This was hard since my friends had a better plan for me that evening but I have stayed faithful to docker. We have had our ups and downs but we still care for each other 😁. We later converted our apps to microservices written in different languages from the wrath C to the lovely python and Go.

## Minimising images
So this is what led me to write this. Most of the developers starting out follow what every Indian guy tells them on Youtube like a Church mass. But they encouter many problems which they don't know how to solve. One of these problems is when you have large images you have a slower runtime and also it takes alot of time to build and test to production. Large images may hog your memory if you write simple apps with a base image of ubuntu or debian yet it would have been better to convert the system to a monolith and they would have shared the requirements.


### 101

Docker is a set of platforms as a service products that uses OS-level virtualization to deliver software in packages called containers.

Docker image is a template to create docker container. It is built using Dockerfile and can be shared to others through a repository.

Dockerfile is a set of instructions to build an image. It starts from base image and every instruction creates a layer which is cached for future builds.

Docker container is created from docker images. We add a writable layer to it and the allocated resoucres.

Registry is where we store docker images remotely.

Lets create a python Flask app for demonstration purposes:
```python=
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "Hello world from container"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

```

We create a virtualenv as we used to do in the monolith days to track our dependencies.
```shell
virtualenv venv --python=python3
```
We activate the virtual environnet.
```shell
source venv/bin/activate
```
We install flask in order to run our web application.
```shell
pip3 install flask
```
We finally write the dependencies to a requirements file.
```shell
pip freeze > requirements.txt
```

The requirements consist of :
```=
click==7.1.1
Flask==1.1.1
itsdangerous==1.1.0
Jinja2==2.11.1
MarkupSafe==1.1.1
Werkzeug==1.0.0
```

We write a first Dockerfile:
```dockerfile=
FROM ubuntu
LABEL Maintainer="Rodney Osodo"
WORKDIR /app
RUN apt-get update   
RUN apt-get install python3-pip
COPY . /app
RUN pip3 install -r requirements.txt  
WORKDIR EXPOSE 5000
CMD ['python3','app.py']
```
and finally build the images,
```shell
docker build -t flask_test:1.0.0 .
```
Our docker images size is 1.28GB, which isn't that bad but is extremely bad. 😂

### Build process

Lets make another dockerfile from python as the base image:
```Dockerfile=
FROM python
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```
Our image size has now reduces to 1GB. Isn't that great?

__Optimization objectives__
1. Reduce image size
2. Reduce build time

__Priorities__
1. Fast builds during development
2. Smaller image for production

Base image
| Image  | Size  | Notes |
| -------- | -------- | -------- |
| python:3-7      | 918MB    | Supports many wheels     |
|    python:3-7-stretch|   929Mb  |Supports many wheels |
|    python:3-7-slim-stretch|  155MB  | Doesnot Supports many wheels |
|    python:3-7-alpine|   98.8MB | Doesnot Supports many wheels |

Use slim-stretch as base when you care about build time.
Use alpine as base when you care about image size.

Lets try reducing the images. Ahh we should all be happy now, right?
```Dockerfile=
FROM python:3.7-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```
Image size is 209MB

```Dockerfile=
FROM python:3.7-slim-stretch
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```
Image size is 186MB

Definitely slim-stretch is better than slim.
What about our alpine base image?
```Dockerfile=
FROM python:3.7-alpine
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```
The image size is 127MB

First take home is that build dependancies are not needed and only include the necessary files, not all your project files. Unfortunately we had only three files. When copying use dockerignore to exclude files that are not needed.

__Tips__
1. Combine multiple RUN statements into a single one then delete them in the same layer.
2. To benefit from caching arrange statements in ascending order depending on system level dependencies.
3. Do not save dependencies
        - pip3 --no-cache-dir
        - apk --no-cache

```Dockerfile=
FROM python:3.7-alpine
WORKDIR /app
COPY . .
RUN pip3 install --no-cache-dir -r requirements.txt

CMD ["python", "app.py"]
```
Size 107MB

__Docker multi stage__
- Build a docker image with all your deps then install your app.
- Copy the result to a fresh image and label is as the final image.
```Dockerfile==
# Stage 1 - Install build dependencies
FROM python:3.7-alpine AS builder
WORKDIR /app
RUN python -m venv .venv && .venv/bin/pip install --no-cache-dir -U pip setuptools
COPY requirements.txt .
RUN .venv/bin/pip install --no-cache-dir -r requirements.txt && find /app/.venv ( -type d -a -name test -o -name tests \) -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) -exec rm -rf '{}' \+
```

```Dockerfile=
# Stage 2 - Copy only necessary files to the runner stage
FROM python:3.7-alpine
WORKDIR /app
COPY --from=builder /app /app
COPY app.py .
ENV PATH="/app/.venv/bin:$PATH"
CMD ["python", "app.py"]
```
Results a smaller image with no build deps.

Bind mount source code instead of COPY in local dev environment.

## Our benchmark
Our benchmark app was a go app with the same functionality.
The beauty of using a compiled code is that it runs faster than interpreted code. The compiled program was checked for errors during compilation.

```go=
package main
import (
    "fmt"
    "net/http"
)
func main(){
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request){
        fmt.Fprintf(w,   "Hello from conatiner")
    })
    http.ListenAndServe(":5000",   nil)
}

```

We compile the program then it will generated an executable file which we can ran from a scratch image:
```Dockerfile
FROM scratch
COPY app /
CMD ["/app"]
```
This results to an image size of 25Mb.

Hopefully you found this story both entertaining and enlightening! I’m hoping this helped to highlight the benefits of minimising docker images and how you could utilize it for everyday applications to improve developer workflow.

If you found this useful then please let me know in the comments section or by tweeting me @[b1ackd0t](https://twitter.com/b1ackd0t). I’d be more than happy to hear any feedback!

