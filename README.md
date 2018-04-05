# Confluent Kafka on Kubernetes

# Table of Content
1. [Component](#component)
1. [Version](#version)
1. [Kubernetes Resources](#resources)
1. [Commands to cleanup](#cleanup)
1. [Issues](#issues)
        1. [Kafka Connect](#connect)
        1. [Zookeeper](#zookeeper)
        1. [Kafka](#kafka)
1. [References](#references)

Deploying Confluent Kafka on Kubernetes

## <a name="component">Components</a>

We are only using a few of the containers/applications that Confluent offer for our Kafka cluster.

1. Zookeeper - https://hub.docker.com/r/confluentinc/cp-zookeeper/
1. Kafka - https://hub.docker.com/r/confluentinc/cp-kafka/
1. Kafka Connect - https://hub.docker.com/r/confluentinc/cp-kafka-connect/

## <a name="version">Version</a>

We are currently using version 3.3.1 because of a .Net library dependency

## <a name="resources">Kubernetes Resources</a>

This is what it should look like once you have deployed all the resources with the
YAML file:

```
[mcharette@kn2vmd808 ~]$ kubectl get all
NAME                     DESIRED   CURRENT   AGE
statefulsets/connect     3         3         17m
statefulsets/kafka       3         3         17m
statefulsets/zookeeper   3         3         17m

NAME             READY     STATUS             RESTARTS   AGE
po/connect-0     0/1       Running            0          17m
po/connect-1     0/1       Running            0          17m
po/connect-2     0/1       Running            0          17m
po/kafka-0       1/1       Running            0          17m
po/kafka-1       1/1       Running            0          16m
po/kafka-2       1/1       Running            0          16m
po/zookeeper-0   1/1       Running            0          17m
po/zookeeper-1   1/1       Running            0          16m
po/zookeeper-2   1/1       Running            0          16m

NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)     AGE
svc/connect      ClusterIP   None          <none>        32181/TCP   17m
svc/kafka        ClusterIP   None          <none>        29092/TCP   17m
svc/kubernetes   ClusterIP   172.21.82.1   <none>        443/TCP     8d
svc/zookeeper    ClusterIP   None          <none>        32181/TCP   17m
```

## <a name="cleanup">Commands to cleanup</a>

```
kubectl delete statefulsets/zookeeper
kubectl delete statefulsets/kafka
kubectl delete statefulsets/connect
kubectl delete svc/zookeeper
kubectl delete svc/kafka
kubectl delete svc/connect
kubectl delete pvc/kafka-kafka-0
kubectl delete pvc/kafka-kafka-1
kubectl delete pvc/kafka-kafka-2
kubectl delete pvc/zookeeper-zookeeper-0
kubectl delete pvc/zookeeper-zookeeper-1
kubectl delete pvc/zookeeper-zookeeper-2
kubectl delete pvc/connect-connect-0
kubectl delete pvc/connect-connect-1
kubectl delete pvc/connect-connect-2
kubectl delete pv/kafka0
kubectl delete pv/kafka1
kubectl delete pv/kafka2
kubectl delete pv/zookeeper0
kubectl delete pv/zookeeper1
kubectl delete pv/zookeeper2
kubectl delete pv/connect0
kubectl delete pv/connect1
kubectl delete pv/connect2
```

## <a name="issues">Issues</a>

### <a name="connect">Kafka Connect</a>

Containers are not starting properly.

The logs are:

```
. /etc/confluent/docker/apply-mesos-overrides
+ . /etc/confluent/docker/apply-mesos-overrides
#!/usr/bin/env bash
#
# Copyright 2016 Confluent Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# Mesos DC/OS docker deployments will have HOST and PORT0
# set for the proxying of the service.
#
# Use those values provide things we know we'll need.
[ -n "${HOST:-}" ] && [ -z "${CONNECT_REST_ADVERTISED_HOST_NAME:-}" ] && \
        export CONNECT_REST_ADVERTISED_HOST_NAME=$HOST || true
++ '[' -n '' ']'
++ true
[ -n "${PORT0:-}" ] && [ -z "${CONNECT_REST_ADVERTISED_PORT:-}" ] && \
        export CONNECT_REST_ADVERTISED_PORT=$PORT0 || true
++ '[' -n '' ']'
++ true
# And default to 8083, which MUST match the containerPort specification
# in the Mesos package for this service.
[ -z "${CONNECT_REST_PORT:-}" ] && \
        export CONNECT_REST_PORT=8083 || true
++ '[' -z '' ']'
++ export CONNECT_REST_PORT=8083
++ CONNECT_REST_PORT=8083
echo "===> ENV Variables ..."
+ echo '===> ENV Variables ...'
env | sort
+ env
+ sort
echo "===> User"
+ echo '===> User'
id
+ id
echo "===> Configuring ..."
+ echo '===> Configuring ...'
/etc/confluent/docker/configure
+ /etc/confluent/docker/configure
dub ensure CONNECT_BOOTSTRAP_SERVERS
+ dub ensure CONNECT_BOOTSTRAP_SERVERS
CONNECT_BOOTSTRAP_SERVERS is required.
Command [/usr/local/bin/dub ensure CONNECT_BOOTSTRAP_SERVERS] FAILED !
```

### <a name="zookeeper">Zookeeper</a>

It is running in standalone mode:

```
[2018-04-04 13:48:39,251] WARN Either no config or no quorum defined in config, running  in standalone mode (org.apache.zookeeper.server.quorum.QuorumPeerMain)
[2018-04-04 13:48:39,266] INFO Reading configuration from: /etc/kafka/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
[2018-04-04 13:48:39,273] INFO Server environment:host.name=zookeeper-0.zookeeper.default.svc.cluster.local (org.apache.zookeeper.server.ZooKeeperServer)
```

### <a name="kafka">Kafka</a>

The containers are connecting to 1 Zookeeper node
It is working but probably not very HA.
Not sure what happens if that zookeeper node was to die or restart


## <a name="references">References</a>

- https://github.com/Yolean/confluent-quickstart-kubernetes
- https://kubernetes.io/docs/tutorials/stateful-application/zookeeper/
