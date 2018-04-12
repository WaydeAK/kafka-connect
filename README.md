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

## <a name="issues">Issues</a>

None at this time

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