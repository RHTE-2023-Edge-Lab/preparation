# Kafka

## Deploy Kafka in Warehouse

Install the **Red Hat Integration - AMQ Streams** operator

Create a Kafka resource.

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: warehouse
spec:
  kafka:
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
      inter.broker.protocol.version: '3.2'
    storage:
      type: ephemeral
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - authentication:
          type: scram-sha-512
        name: tls
        port: 9093
        type: route
        tls: true
    version: 3.2.3
    replicas: 3
  entityOperator:
    topicOperator: {}
    userOperator: {}
  zookeeper:
    storage:
      type: ephemeral
    replicas: 3
```

Create the user.

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: camel-user
stringData:
  password: s3cr3t
type: Opaque
---
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: camel
  labels:
    strimzi.io/cluster: warehouse
spec:
  authentication:
    password:
      valueFrom:
        secretKeyRef:
          name: camel-user
          key: password
    type: scram-sha-512
```

Create the topics.

```yaml
kind: KafkaTopic
apiVersion: kafka.strimzi.io/v1beta2
metadata:
  name: warehouse-in
  labels:
    strimzi.io/cluster: warehouse
spec:
  config:
    retention.ms: 604800000
    segment.bytes: 1073741824
---
kind: KafkaTopic
apiVersion: kafka.strimzi.io/v1beta2
metadata:
  name: warehouse-out
  labels:
    strimzi.io/cluster: warehouse
spec:
  config:
    retention.ms: 604800000
    segment.bytes: 1073741824
```
