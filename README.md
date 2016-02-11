# How to create a Docker image with MongoDB and another with Cloudera QuickStart to use Pig over Mongo

#### 1. Let’s say we have a Mongo database in a USB device

#### 2. Create your onw MongoDB images: 

From here: https://docs.docker.com/engine/examples/mongodb/

Let's create a file that will help us to create our image for MongoDB
$ nano Dockerfile

```
# Dockerizing MongoDB: Dockerfile for building MongoDB images
# Based on ubuntu:latest, installs MongoDB following the instructions from:
# http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/

FROM ubuntu:latest
MAINTAINER Docker
# Installation:
# Import MongoDB public GPG key AND create a MongoDB list file
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
RUN echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release -sc)"/mongodb-org/3.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-3.0.list

# Update apt-get sources AND install MongoDB
RUN apt-get update && apt-get install -y mongodb-org

# Create the MongoDB data directory
RUN mkdir -p /data/db

# Expose port #27017 from the container to the host
EXPOSE 27017

# Set /usr/bin/mongod as the dockerized entry-point application
# install wget
RUN yum install wget

# mongo hadoop drivers:
RUN wget https://github.com/mongodb/mongo-hadoop/releases/download/r1.4.0/mongo-hadoop-pig-1.4.0.jar
RUN wget https://github.com/mongodb/mongo-hadoop/releases/download/r1.4.0/mongo-hadoop-hive-1.4.0.jar
RUN wget https://github.com/mongodb/mongo-hadoop/releases/download/r1.4.0/mongo-hadoop-core-1.4.0.jar

# mongo java driver:
RUN wget https://oss.sonatype.org/content/repositories/releases/org/mongodb/mongo-java-driver/2.14.1/mongo-java-driver-2.14.1.jar

ENTRYPOINT ["/usr/bin/mongod"]
```

Once you have create this file, you have to build the image (wait):
```
# Format: docker build --tag/-t <user-name>/<repository> .
# Example:
$ docker build --tag yourRepo/mongo .
````
When this task is finished you can do a *$ docker images*
```
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
yourRepo/mongo          latest              17635816c2b0        4 hours ago         1.027 GB
```

And then you can run your image:
```
# Dockerized MongoDB, lean and mean!# Usage: docker run --name <name for container> -d <user-name>/<repository> --noprealloc --smallfiles
$ docker run -p 27017:27017 --name mongo_instance_001 -privileged /dev/bus/usb:/dev/bus/usb -d yourRepo/mongo --smallfiles
```

And now you have a running docker container:
```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS                          PORTS               NAMES
0ff9e649bc01        17635816c2b0        "/usr/bin/mongod --sm"   3 minutes ago        Exited (0) About a minute ago                       mongo
```

#### 2. Download and start hadoop container:
From here: https://blog.cloudera.com/blog/2015/12/docker-is-the-new-quickstart-option-for-apache-hadoop-and-cloudera/

```
$ docker pull cloudera/quickstart:latest
$ docker images # note the hash of the image and substitute it below
$ docker run --privileged=true --hostname=quickstart.cloudera -t -i ${HASH} /usr/bin/docker-quickstart
````

