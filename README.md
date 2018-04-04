# Confluent Kafka on Kubernetes

Deploying Confluent Kafka on Kubernetes

## Components

We are only using a few of the containers/applications that Confluent offer for our Kafka cluster.

1. Zookeeper - https://hub.docker.com/r/confluentinc/cp-zookeeper/
1. Kafka - https://hub.docker.com/r/confluentinc/cp-kafka/
1. Kafka Connect - https://hub.docker.com/r/confluentinc/cp-kafka-connect/

## Version

We are currently using version 3.3.1 because of a .Net library dependency


## Commands to cleanup

```
kubectl delete statefulsets/zookeeper
kubectl delete statefulsets/kafka
kubectl delete svc/zookeeper
kubectl delete svc/kafka
kubectl delete pvc/kafka-kafka-0
kubectl delete pvc/zookeeper-zookeeper-0
kubectl delete pv/kafka
kubectl delete pv/zookeeper
```

## References

- https://github.com/Yolean/confluent-quickstart-kubernetes
