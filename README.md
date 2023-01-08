## FileScale: Fast and Elastic Metadata Management for Distributed File Systems

Recent work has shown that distributed database systems are a promising solution for scaling metadata management in scalable file systems. This work has shown that systems that store metadata on a single machine, or over a shared-disk abstraction, struggle to scale performance to deployments including billions of files. In contrast, leveraging a scalable, shared-nothing, distributed system for metadata storage can achieve much higher levels of scalability, without giving up high availability guarantees. However, for low-scale deployments – where metadata can fit in memory on a single machine – these systems that store metadata in a distributed database typically perform an order of magnitude worse than systems that store metadata in memory on a single machine. This has limited the impact of these distributed database approaches, since they are only currently applicable to file systems of extreme scale.

FileScale is a three-tier architecture that incorporates a distributed database system as part of a comprehensive approach to metadata management in distributed file systems. In contrast to previous approaches, the architecture described in the paper performs comparably to the single-machine architecture at a small scale, while enabling linear scalability as the file system metadata increases.

## Deploy Database Layer

Currently, our system only provides technical support to [VoltDB](https://www.voltactivedata.com/) and [Apache Ignite](https://ignite.apache.org/).

### VoltDB

1. Download VoltDB from https://www.voltactivedata.com/. Then, unpack your downloaded `VoltDB tar.gz` file into a folder "voltdb-ent".

```shell
voltdb-docker>$ mkdir voltdb-ent
voltdb-docker>$ tar -xzf voltdb-ent-8.4.2.tar.gz
voltdb-docker>$ mv voltdb-ent-8.4.2 voltdb-ent
```

2. Deploy VoltDB

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

## License

FileScale resources in this repository are released under the Apache License 2.0.
