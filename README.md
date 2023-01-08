## FileScale: Fast and Elastic Metadata Management for Distributed File Systems

Distributed database systems have been shown to be effective for scaling metadata management in scalable file systems. These systems can handle billions of files, while traditional systems that store metadata on a single machine or shared-disk abstraction struggle to scale. However, distributed database systems perform worse than single-machine systems at low scales, where metadata can fit in memory. 

FileScale is a three-tier architecture that addresses this issue by incorporating a distributed database system while maintaining comparable performance to single-machine systems at small scales and enabling linear scalability as metadata increases.

## Deploy Database Layer

Currently, our system only provides technical support to [VoltDB](https://www.voltactivedata.com/) and [Apache Ignite](https://ignite.apache.org/).

### VoltDB

1. Download VoltDB from https://www.voltactivedata.com/. Then, unpack your downloaded `VoltDB tar.gz` file into a folder "voltdb-ent".

```shell
voltdb-docker>$ mkdir voltdb-ent
voltdb-docker>$ tar -xzf voltdb-ent-8.4.2.tar.gz
voltdb-docker>$ mv voltdb-ent-8.4.2 voltdb-ent
```

2. Prepare VoltDB

If we want to use 3 nodes, we need to create a script that will generate a deployment file and start VoltDB using this file. To do this, we will create a new file called deploy.py in the directory that contains our dockerfile. The script will include the following code:

```python
import sys, os

deploymentText = """<?xml version="1.0"?>
<deployment>
    <cluster hostcount="##HOSTCOUNT##" kfactor="##K##" />
    <httpd enabled="true"><jsonapi enabled="true" /></httpd>
</deployment>
"""

deploymentText = deploymentText.replace("##HOSTCOUNT##", sys.argv[1])
deploymentText = deploymentText.replace("##K##", sys.argv[2])

with open('/root/voltdb-ent/deployment.xml', 'w') as f:
    f.write(deploymentText)

os.execv("/root/voltdb-ent/bin/voltdb",
        ["voltdb",
        "create",
        "--deployment=/root/voltdb-ent/deployment.xml",
        "--host=" + sys.argv[3]])
```

Next, create a file named `Dockerfile` in the directory with the following contents:

```Dockerfile
# VoltDB on top of Docker base JDK8 images
FROM java:8
WORKDIR /root
COPY voltdb-ent/ voltdb-ent/
COPY deploy.py voltdb-ent/
WORKDIR /root/voltdb-ent
# Ports
# 21212 : Client Port
# 21211 : Admin Port
#  8080 : Web Interface Port
#  3021 : Internal Server Port
#  4560 : Log Port
#  9090 : JMX Port
#  5555 : Replication Port
#  7181 : Zookeeper Port
EXPOSE 21212 21211 8080 3021 4560 9090 5555 7181
CMD /bin/bash
```

3. Build Docker Image

```shell
$ docker build -t hawaii/voltdb:8.4.2 .

$ docker image

REPOSITORY            TAG         IMAGE ID            CREATED             SIZE
hawaii/voltdb       8.4.2       d08f1b9a1569        6 minutes ago       773MB
```

4. Deploy VoltDB Cluster

Now we can start a 1 node (k=0) VoltDB cluster in this way:

```shell
docker run --name=volt1 --hostname=volt1 -d -p 8080:8080 -p 21212:21212 \
hawaii/voltdb:8.4.2 /root/voltdb-ent/deploy.py 3 1 volt1
```

Find the IP of the first container using:

```shell
$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' volt1

172.17.0.3
```

We can use that IP as the leader for the second and third nodes:

```shell
$ export LEADERIP=172.17.0.3
$ docker run --name=volt2 --hostname=volt2 -d -p 8081:8080 \
    hawaii/voltdb:8.4.2 /root/voltdb-ent/deploy.py 3 1 $LEADERIP
$ docker run --name=volt3 --hostname=volt3 -d -p 8082:8080 \
    hawaii/voltdb:8.4.2 /root/voltdb-ent/deploy.py 3 1 $LEADERIP

$ docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS            PORTS                    NAMES
a3a4a96b2222        hawaii/voltdb:8.4.2   "/root/voltdb-ent/de…"   3 seconds ago       Up 3 seconds        0.0.0.0:8082->8080/tcp   volt3
75e9b9190ce5        hawaii/voltdb:8.4.2   "/root/voltdb-ent/de…"   12 seconds ago      Up 11 seconds       0.0.0.0:8081->8080/tcp   volt2
015db7c2dccb        hawaii/voltdb:8.4.2   "/root/voltdb-ent/de…"   3 minutes ago       Up 3 minutes        0.0.0.0:8080->8080/tcp   volt1
```


## License

FileScale resources in this repository are released under the Apache License 2.0.
