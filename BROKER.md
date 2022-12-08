# Mise en place du broker mqtt

## Installation de l'operateur

Connectez vous au cluster openshift est creez un namespace dedie a l'atelier 

```shell
oc new-project esp32-rhte
```


- Naviguez dans la console Web vers la page Opérateurs → OperatorHub.
- Tapez "AMQ broker" dans la zone Filtrer par mot-clé pour trouver l'Opérateur selectionnez "Red Hat Integration - AMQ Broker for RHEL 8 (Multiarch)" et cliquez sur Install
- Sur la page Installer l'opérateur, sélectionnez A specific namespace on the cluster et selectionnez le namespace "esp32-rhte".
- Cliquez sur "installer"

## Creation du broker

Une fois l'operateur installe creer le broker. 

```shell
cat > broker.yaml <<EOF
apiVersion: broker.amq.io/v1beta1
kind: ActiveMQArtemis
metadata:
  name: mqtt-3
  namespace: esp32-rhte
spec:
  acceptors:
  - connectionsAllowed: 5
    expose: true
    name: mqtt
    port: 1883
    protocols: mqtt
    sslEnabled: false
    adminPassword: public
    adminUser: admin
  adminPassword: public
  adminUser: admin
  console:
    expose: true
  deploymentPlan:
    image: placeholder
    jolokiaAgentEnabled: false
    journalType: nio
    managementRBACEnabled: true
    messageMigration: false
    persistenceEnabled: false
    requireLogin: true
    size: 1
EOF
```

```shell
oc apply -f broker.yaml
```

On creer ensuite une address 

```shell
cat > AMQaddress.yaml <<EOF
kind: ActiveMQArtemisAddress
apiVersion: broker.amq.io/v1beta1
metadata:
  name: address
  namespace: esp32-rhte
spec:
  addressName: myAddress0
  queueName: myQueue0
  routingType: anycast
EOF
```

```shell
oc apply -f AMQaddress.yaml
```

## Exposition du broker

Le routeur OpenShift ne pourra pas acheminer une connexion MQTT en clair, car le protocole MQTT ne porte pas de nom d'hôte de destination comme, par exemple, HTTP le fait dans son en-tête "Host:". Pour exposer notre broker nous pouvons utiliser une route en mode passthrough ou une exposition en NodePort. Dans cette exemple le nodePort est utilise.

Afficher les services creer lors de la creation du broker.

```shell
oc get svc -n esp32-rhte
```
```shell
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
mqtt-3-hdls-svc       ClusterIP   None            <none>        8161/TCP,61616/TCP   37s
mqtt-3-mqtt-0-svc     ClusterIP   172.30.44.180   <none>        1883/TCP             37s
mqtt-3-ping-svc       ClusterIP   None            <none>        8888/TCP             37s
mqtt-3-wconsj-0-svc   ClusterIP   172.30.19.143   <none>        8161/TCP             37s
```

On passe le service "mqtt-3-mqtt-0-svc" en type NodePort

```shell
oc patch svc mqtt-3-mqtt-0-svc --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'
```

## test

Dans un premier terminal creer une subscription a votre broker

```shell
mosquitto_sub -t esp32 -h '10.6.82.226' -p 31053 -u admin -P public
```
NOTE: L'address Ip dois etre celle de l'un de vos noeuds et le port celui exposer via Nodeport

Dans un second terminal envoyez un message qui dois etre recu dans le premier
```shell
mosquitto_pub -t esp32 --host '10.6.82.226' -p 31053 -u admin -P public -m 'hello cest flo'
```