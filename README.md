# RHTE Edge Lab

## Description

*From the ESP32 to the OpenShift cluster: discover what Red Hat can offer for the Edge computing!*

*Through the common theme of "shipment tracking", this lab will help you understand what Red Hat can offer for the Edge computing.*

*In this lab, you will be at the head of a "parcel shipment hub" and you will deploy a everything needed to : read the parcel RFID (using arduino and ESP32), send data to a MQTT broker over wifi, transform those data using Camel-K and send relevant events to the headquarter for reporting.*

*An application at the headquarter will display the parcels moving from one hub to another in realtime.*

*Fun is expected !*

## Lab Materials

## Prerequisites

## Schema

![Schema](Schema.png)

Edit the schema [here](https://app.diagrams.net/#HRHTE-2023-Edge-Lab%2Fpreparation%2Fmain%2FSchema.drawio).

## Instructions


## Acknowledgment

This lab is based on the materials provided by this [Validated Pattern](https://redhat-gitops-patterns.io/industrial-edge/).

## Deadline

Lab materials are due for December, the 23rd.

## People

* David Martini
* Florian Even
* Adrien Legros
* Sébastien Lallemand
* Nicolas Massé

## To be validated

* 🟢 Hardware: ESP8266 + RC522
* 🟢 Hardware: Firmware source code
* 🟢 Architecture: MQTT Broker (AMQ Broker)
* 🟢 Architecture: Kafka Broker (AMQ Streams)
* 🔴 Architecture: MQTT-Kafka bridge (Camel Quarkus, Camel-K ?)
* 🟢 Architecture: Kafka replication between warehouse and headquarter (Mirror Maker)
* 🟠 Architecture: World Map Front (Kibana, Grafana, [Quarkus frontend](https://github.com/RHTE-2023-Edge-Lab/worldmap-front), [Python frontend](https://github.com/RHTE-2023-Edge-Lab/worldmap-plotly) ?)
* 🟢 Hosting: ~~HPE~~, **RHPDS**
* 🟢 Hosting: Deployment Method (**ArgoCD**, ~~Ansible~~ ?)
* 🟢 Lab materials: Format ([Markdown](https://github.com/nmasse-itix/api-lifecycle-workshop/tree/master/lab-instructions), ~~Asciidoc~~ ?)
* 🟢 Lab materials: Site generator ([Hugo](https://api-lifecycle-workshop.netlify.app/) on github pages, ~~Athena~~ ?)
* 🔴 Lab materials: Theme ([Learn](https://learn.netlify.app/en/))
