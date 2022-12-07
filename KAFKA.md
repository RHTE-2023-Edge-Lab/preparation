# Kafka

## Deploy Kafka in Warehouse

Create a project named "warehouse-paris".

Install the **Red Hat Integration - AMQ Streams** operator

Create a Kafka resource.

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: warehouse-paris
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
      - authentication:
          type: scram-sha-512
        name: plain
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

Create the users.

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
    strimzi.io/cluster: warehouse-paris
spec:
  authentication:
    password:
      valueFrom:
        secretKeyRef:
          name: camel-user
          key: password
    type: scram-sha-512
---
kind: Secret
apiVersion: v1
metadata:
  name: mirrormaker-user
stringData:
  password: s3cr3t
type: Opaque
---
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: mirrormaker
  labels:
    strimzi.io/cluster: warehouse-paris
spec:
  authentication:
    password:
      valueFrom:
        secretKeyRef:
          name: mirrormaker-user
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
    strimzi.io/cluster: warehouse-paris
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
    strimzi.io/cluster: warehouse-paris
spec:
  config:
    retention.ms: 604800000
    segment.bytes: 1073741824
```

## Deploy Kafka in Headquarter

Create a project named "headquarter".

Install the **Red Hat Integration - AMQ Streams** operator

Create a Kafka resource.

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: headquarter
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
      - authentication:
          type: scram-sha-512
        name: plain
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
  name: warehouse-paris
stringData:
  password: s3cr3t
type: Opaque
---
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: warehouse-paris
  labels:
    strimzi.io/cluster: headquarter
spec:
  authentication:
    password:
      valueFrom:
        secretKeyRef:
          name: warehouse-paris
          key: password
    type: scram-sha-512
```

## Configure MirrorMaker 2.0

See [Documentation](https://strimzi.io/docs/operators/latest/configuring.html#proc-mirrormaker-replication-str).

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: headquarter
stringData:
  password: s3cr3t
  ca-certificate: |
    -----BEGIN CERTIFICATE-----
    MIIFLTCCAxWgAwIBAgIUUXBfQ1fNYuL459ka9h20JX42EakwDQYJKoZIhvcNAQEN
    BQAwLTETMBEGA1UECgwKaW8uc3RyaW16aTEWMBQGA1UEAwwNY2x1c3Rlci1jYSB2
    MDAeFw0yMjEyMDYxNDE5MTdaFw0yMzEyMDYxNDE5MTdaMC0xEzARBgNVBAoMCmlv
    LnN0cmltemkxFjAUBgNVBAMMDWNsdXN0ZXItY2EgdjAwggIiMA0GCSqGSIb3DQEB
    AQUAA4ICDwAwggIKAoICAQC8ctPIYRId1gyZlVzZzQWJF5aVL9fFEk18WI1DReiT
    hl5B2r+b361PeGeHJZy8fZav8txZQ9IpEbLclrWWOTJ9/j3Opn4u9Cdz16cLtm6Y
    3lPDG64omuhEVzbOsOko7ivT5b8de5nN4myG9VefRqT+wYx3xrwmFYjsnhqfXk4/
    k9Eg3gigTN/VT42FWOtFIzPAB2N6hxuvP1uDk4MEOM1AM6KFBuV1NNTEbmzD+OFA
    nrsnbykVT6V4wajt7HoNk2pSAf9Ofnw3hmlXxHk3bbc7cPXc1ZRtdpcGW3IawWo9
    g1yRqkaKNzixc7S6f17w0bgV5ntUgd/NnpUf8hahUhUkluqwtK3x/fHNjW2QBm7Z
    Dm6z9ZEZhq4ZRaHcdq4HjipG+MIhKa5jD2bmzNjp/FyrwezB4zxLvlgUiOyZPNfP
    z35P9thGWRHgdRVTi6ne/LFFgb1DVq68+htrA42aPHV+5u88viV79nFf648WDoMS
    3T+I47hlTor3gFnmrKoD5tVFZOaR3c+6K/S8Z6BTSK1382WhCsXN2Kdv0BqnQYXJ
    3BGtTSHKNC1tvEPWK1sbqd61li1LPH3/7Hl5ImChqK5JY3ONVTMRWzWFHnoYoNX1
    HedZNFI2o38FuQbHrzuywtGcspyNZBg7BSXXqG+nTDitCAiV1Ti2D4Rab9hitw+t
    AwIDAQABo0UwQzAdBgNVHQ4EFgQUs0hMb9jIoXz2ELwswjNaNrE9b+AwEgYDVR0T
    AQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMCAQYwDQYJKoZIhvcNAQENBQADggIB
    AEgSSPiMhyL0d+x4b54iz31UKUQMNBat9aMzz5QMa7/uzvfg9Yu5BZ5JOI/2eTDJ
    R61iR8hJSmPwkW5VIdP+F3BAwDX14T/oL0tTBfH1EW4puSuwg3oO/neyowJ+OP66
    P1DC3odvP1MBhPluv5OHSnivrfBLXy74GM5oNiqZr8H92yAJZzOmM6WybZ57+GSW
    QdImAcJiSekH1xLguEGIjd/MlU7OaGMXFkLngsSym0phoTYG7RApGBVN29l4uIfT
    +zYDATn3x5LahhOCrEODG9cPNSDCHBIGu0ZYgvSMDq8cwwd4EG388lug61Q0+P8b
    meGt+tC1GZdnmULUNzOqxQ8yEwFVV8Vy9aQ5cRF/JVp0hc68n6RGae784EkBrW3F
    +wuE20cRmkiXCt2qi8hr1jepCNdoTd6t6kizn5lPw8BVUiAzQ3eFu59SSZax8KAF
    QG4BlYLKSdCXuu5swlULqLyLitBG1+pzRPbfpVGak8VD2kw3MzfA9rZltBZoHP5i
    uj0IIIOH6qplb84Q5bRhnd+86hsd2kotbvdYykySkwp32S/o8SYyLm4f2DBnYRSo
    3VsrabiRlV74+33+3odpTTXO0R/NrDAmotvAPotlXj2AQo99Dn87Djxv17HE76Kg
    VgAoE89L3gD5aOPOfdgFqyJmlxCdLupy2Bsk/CCLJ+lj
    -----END CERTIFICATE-----
type: Opaque
```

**TODO**: explain how to fetch the CA Certificate above or how to set it in the target Kafka configuration.

Deploy MirrorMaker in each Warehouse.

```yaml
kind: KafkaMirrorMaker2
apiVersion: kafka.strimzi.io/v1beta2
metadata:
  name: to-headquarter
spec:
  version: 3.2.3
  replicas: 1
  connectCluster: headquarter
  clusters:
    - alias: warehouse-paris
      bootstrapServers: 'warehouse-paris-kafka-bootstrap:9092'
      authentication:
        passwordSecret:
          secretName: mirrormaker-user
          password: password
        type: scram-sha-512
        username: mirrormaker
    - alias: headquarter
      bootstrapServers: 'headquarter-kafka-tls-bootstrap-headquarter.apps.appdev.itix.xyz:443'
      config:
        config.storage.replication.factor: -1
        offset.storage.replication.factor: -1
        status.storage.replication.factor: -1
      authentication:
        passwordSecret:
          secretName: headquarter
          password: password
        type: scram-sha-512
        username: warehouse-paris
      tls:
        trustedCertificates:
          - certificate: ca-certificate
            secretName: headquarter
  mirrors:
    - sourceCluster: warehouse-paris
      targetCluster: headquarter
      sourceConnector:
        config:
          replication.factor: 3
          offset-syncs.topic.replication.factor: 1
          sync.topic.acls.enabled: 'false'
      heartbeatConnector:
        config:
          heartbeats.topic.replication.factor: 1
          emit.heartbeats.enabled: false
      checkpointConnector:
        config:
          checkpoints.topic.replication.factor: 1
          sync.group.offsets.enabled: false
      topicsPattern: warehouse.*
```

## Troubleshooting / Validation

File `~/tmp/kcat-hq.conf`

```ini
# Required connection configs for Kafka producer, consumer, and admin
bootstrap.servers=headquarter-kafka-tls-bootstrap-headquarter.apps.appdev.itix.xyz:443
ssl.ca.location=/home/nmasse/tmp/headquarter.pem
security.protocol=SASL_SSL
sasl.mechanisms=SCRAM-SHA-512
sasl.username=warehouse-paris
sasl.password=s3cr3t

# Best practice for higher availability in librdkafka clients prior to 1.7
session.timeout.ms=45000
```

File `~/tmp/kcat-paris.conf`

```ini
# Required connection configs for Kafka producer, consumer, and admin
bootstrap.servers=warehouse-paris-kafka-tls-bootstrap-warehouse-paris.apps.appdev.itix.xyz
ssl.ca.location=/home/nmasse/tmp/warehouse-paris.pem
security.protocol=SASL_SSL
sasl.mechanisms=SCRAM-SHA-512
sasl.username=camel
sasl.password=s3cr3t

# Best practice for higher availability in librdkafka clients prior to 1.7
session.timeout.ms=45000
```

Commands:

```sh
kcat -b warehouse-paris-kafka-tls-bootstrap-warehouse-paris.apps.appdev.itix.xyz:443 -L -F ~/tmp/kcat-paris.conf
kcat -b warehouse-paris-kafka-tls-bootstrap-warehouse-paris.apps.appdev.itix.xyz:443 -P -F ~/tmp/kcat-paris.conf -t warehouse-in
kcat -b warehouse-paris-kafka-tls-bootstrap-warehouse-paris.apps.appdev.itix.xyz:443 -C -F ~/tmp/kcat-paris.conf -t warehouse-in
kcat -b headquarter-kafka-tls-bootstrap-headquarter.apps.appdev.itix.xyz:443 -L -F ~/tmp/kcat-hq.conf
kcat -b headquarter-kafka-tls-bootstrap-headquarter.apps.appdev.itix.xyz:443 -C -F ~/tmp/kcat-hq.conf -t warehouse-paris.warehouse-in
```
