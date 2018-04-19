# Confluent Kafka on Kubernetes

# Table of Content
1. Component
1. Version
1. Kubernetes Resources
1. Commands to cleanup
1. Issues
1. Bash Alias
1. References

Deploying Confluent Kafka on Kubernetes

## <a name="component">Components</a>

We are only using a few of the containers/applications that Confluent offer for our Kafka cluster.

1. Zookeeper - I was not able to get the Confluent Zookeeper working in a StatefulSet so I switched it to use the 
    Google one - Image: k8s.gcr.io/kubernetes-zookeeper:1.0-3.4.10
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
svc/connect      ClusterIP   172.168.1.101 <none>        8083/TCP    17m
svc/kafka-broker ClusterIP   172.168.1.100 <none>        29092/TCP   17m
svc/kubernetes   ClusterIP   172.168.1.1   <none>        443/TCP     8d
svc/zookeeper    ClusterIP   None          <none>        32181/TCP   17m
```

## Create Topics

```
/usr/bin/kafka-topics --create --zookeeper zk-cs:2181 --replication-factor 3 --partitions 3 --topic quickstart
/usr/bin/kafka-topics --create --zookeeper zk-cs:2181 --replication-factor 3 --partitions 3 --topic quickstart-config
/usr/bin/kafka-topics --create --zookeeper zk-cs:2181 --replication-factor 3 --partitions 3 --topic quickstart-offsets
/usr/bin/kafka-topics --create --zookeeper zk-cs:2181 --replication-factor 3 --partitions 3 --topic quickstart-status
```


## <a name="cleanup">Commands to cleanup</a>

```
kubectl delete statefulsets/zk
kubectl delete statefulsets/kafka
kubectl delete statefulsets/connect
kubectl delete svc/zk-cs
kubectl delete svc/zk-hs
kubectl delete svc/kafka-broker
kubectl delete svc/connect-rest
kubectl delete pvc/kafka-kafka-0
kubectl delete pvc/kafka-kafka-1
kubectl delete pvc/kafka-kafka-2
kubectl delete pvc/zookeeper-zk-0
kubectl delete pvc/zookeeper-zk-1
kubectl delete pvc/zookeeper-zk-2
kubectl delete pvc/connect-connect-0
kubectl delete pvc/connect-connect-1
kubectl delete pvc/connect-connect-2
kubectl delete pv/kafka0
kubectl delete pv/kafka1
kubectl delete pv/kafka2
kubectl delete pv/zk0
kubectl delete pv/zk1
kubectl delete pv/zk2
kubectl delete pv/connect0
kubectl delete pv/connect1
kubectl delete pv/connect2
```

If you want to clean the directories where some data and configuration might be you should execute:

```
sudo rm -Rf /kafka/zookeeper/zk-*/data/log/*
sudo rm -Rf /kafka/zookeeper/zk-*/data/version-2
```

## Zookeeper Commands

On a zookeeper server you can connect to the cluster with:

zkCli.sh

To list the Kafka brokers:

ls /brokers/ids


## <a name="issues">Issues</a>

Kafka Connect is not able to communicate with the Kafka cluster:

```
+ cub kafka-ready 1 40 -b kafka-broker:9092
Error while getting broker list.
java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.TimeoutException: Timed out waiting for a node assignment.
	at org.apache.kafka.common.internals.KafkaFutureImpl.wrapAndThrow(KafkaFutureImpl.java:45)
	at org.apache.kafka.common.internals.KafkaFutureImpl.access$000(KafkaFutureImpl.java:32)
	at org.apache.kafka.common.internals.KafkaFutureImpl$SingleWaiter.await(KafkaFutureImpl.java:89)
	at org.apache.kafka.common.internals.KafkaFutureImpl.get(KafkaFutureImpl.java:213)
	at io.confluent.admin.utils.ClusterStatus.isKafkaReady(ClusterStatus.java:149)
	at io.confluent.admin.utils.cli.KafkaReadyCommand.main(KafkaReadyCommand.java:150)
Caused by: org.apache.kafka.common.errors.TimeoutException: Timed out waiting for a node assignment.
Expected 1 brokers but found only 0. Trying to query Kafka for metadata again ...
Expected 1 brokers but found only 0. Brokers found [].
```

The Kafka Connect pod are not able to read from the service and they can't connect directly to the pod:

```
root@connect-2:/# nc kafka-0 9092                                                                                                                                                                                                                                                                                            
kafka-0: forward host lookup failed: Host name lookup failure : Resource temporarily unavailable
root@connect-2:/# nc kafka-broker 9092
test
root@connect-2:/# 
```

This error was caused when I applied a new k8s yaml for the services:

```
[2018-04-13 14:35:20,793] INFO Kafka Connect started (org.apache.kafka.connect.runtime.Connect)
[2018-04-13 14:36:41,939] ERROR Uncaught exception in herder work thread, exiting:  (org.apache.kafka.connect.runtime.distributed.DistributedHerder)
org.apache.kafka.connect.errors.ConnectException: Error while attempting to create/find topic(s) 'quickstart-offsets'
	at org.apache.kafka.connect.util.TopicAdmin.createTopics(TopicAdmin.java:247)
	at org.apache.kafka.connect.storage.KafkaOffsetBackingStore$1.run(KafkaOffsetBackingStore.java:99)
	at org.apache.kafka.connect.util.KafkaBasedLog.start(KafkaBasedLog.java:126)
	at org.apache.kafka.connect.storage.KafkaOffsetBackingStore.start(KafkaOffsetBackingStore.java:109)
	at org.apache.kafka.connect.runtime.Worker.start(Worker.java:146)
	at org.apache.kafka.connect.runtime.AbstractHerder.startServices(AbstractHerder.java:99)
	at org.apache.kafka.connect.runtime.distributed.DistributedHerder.run(DistributedHerder.java:194)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.NotControllerException: This is not the correct controller for this cluster.
	at org.apache.kafka.common.internals.KafkaFutureImpl.wrapAndThrow(KafkaFutureImpl.java:45)
	at org.apache.kafka.common.internals.KafkaFutureImpl.access$000(KafkaFutureImpl.java:32)
	at org.apache.kafka.common.internals.KafkaFutureImpl$SingleWaiter.await(KafkaFutureImpl.java:89)
	at org.apache.kafka.common.internals.KafkaFutureImpl.get(KafkaFutureImpl.java:213)
	at org.apache.kafka.connect.util.TopicAdmin.createTopics(TopicAdmin.java:227)
	... 11 more
Caused by: org.apache.kafka.common.errors.NotControllerException: This is not the correct controller for this cluster.
[2018-04-13 14:36:41,941] INFO Kafka Connect stopping (org.apache.kafka.connect.runtime.Connect)
[2018-04-13 14:36:41,943] INFO Stopping REST server (org.apache.kafka.connect.runtime.rest.RestServer)
[2018-04-13 14:36:41,957] INFO Stopped ServerConnector@1259b26b{HTTP/1.1}{0.0.0.0:8083} (org.eclipse.jetty.server.ServerConnector)
[2018-04-13 14:36:41,979] INFO Stopped o.e.j.s.ServletContextHandler@103a845c{/,null,UNAVAILABLE} (org.eclipse.jetty.server.handler.ContextHandler)
[2018-04-13 14:36:41,980] INFO REST server stopped (org.apache.kafka.connect.runtime.rest.RestServer)
[2018-04-13 14:36:41,980] INFO Herder stopping (org.apache.kafka.connect.runtime.distributed.DistributedHerder)
[2018-04-13 14:36:46,981] INFO Herder stopped (org.apache.kafka.connect.runtime.distributed.DistributedHerder)
[2018-04-13 14:36:46,982] INFO Kafka Connect stopped (org.apache.kafka.connect.runtime.Connect)
```

This happened a few times but not consistently. Kafka Connect seems to have issues retrieving its data from the topic:

```
[2018-04-16 14:18:14,213] WARN Error while fetching metadata with correlation id 8863 : {quickstart-status=LEADER_NOT_AVAILABLE} (org.apache.kafka.clients.NetworkClient)
[2018-04-16 14:18:14,314] WARN Received unknown topic or partition error in ListOffset request for partition quickstart-status-4. The topic/partition may not exist or the user may not have Describe access to it. (org.apache.kafka.clients.consumer.internals.Fetcher)
[2018-04-16 14:18:14,414] WARN Received unknown topic or partition error in ListOffset request for partition quickstart-status-4. The topic/partition may not exist or the user may not have Describe access to it. (org.apache.kafka.clients.consumer.internals.Fetcher)
[2018-04-16 14:18:14,514] WARN Received unknown topic or partition error in ListOffset request for partition quickstart-status-4. The topic/partition may not exist or the user may not have Describe access to it. (org.apache.kafka.clients.consumer.internals.Fetcher)
[2018-04-16 14:18:14,523] ERROR Uncaught exception in herder work thread, exiting:  (org.apache.kafka.connect.runtime.distributed.DistributedHerder)
org.apache.kafka.common.errors.TimeoutException: Failed to get offsets by times in 305000 ms
[2018-04-16 14:18:14,525] INFO Kafka Connect stopping (org.apache.kafka.connect.runtime.Connect)
[2018-04-16 14:18:14,526] INFO Stopping REST server (org.apache.kafka.connect.runtime.rest.RestServer)
[2018-04-16 14:18:14,531] INFO Stopped ServerConnector@1259b26b{HTTP/1.1}{0.0.0.0:8083} (org.eclipse.jetty.server.ServerConnector)
[2018-04-16 14:18:14,547] INFO Stopped o.e.j.s.ServletContextHandler@103a845c{/,null,UNAVAILABLE} (org.eclipse.jetty.server.handler.ContextHandler)
[2018-04-16 14:18:14,548] INFO REST server stopped (org.apache.kafka.connect.runtime.rest.RestServer)
[2018-04-16 14:18:14,548] INFO Herder stopping (org.apache.kafka.connect.runtime.distributed.DistributedHerder)
```

## Bash Alias

I have these because I don't like to type too much:

```
alias k="kubectl"
alias kd="kubectl delete"
alias kg="kubectl get"
alias kga="kubectl get all"
alias kl="kubectl logs"
alias ke="kubectl exec -it"
alias ka="kubectl apply"
```

## <a href="references">References</a>

- https://github.com/Yolean/confluent-quickstart-kubernetes
- https://kubernetes.io/docs/tutorials/stateful-application/zookeeper/
- https://github.com/confluentinc/schema-registry/issues/689
- https://github.com/kubernetes/kubernetes/issues/40651
- https://github.com/confluentinc/cp-docker-images/wiki/Getting-Started
- https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/
- https://docs.confluent.io/current/installation/docker/docs/configuration.html
- https://github.com/darrenfu/bigdata/issues/6