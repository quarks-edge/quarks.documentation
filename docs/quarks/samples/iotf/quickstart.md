---
layout: docs
title: Quickstart IBM Watson IoT Platform Sample
description: Connecting Quarks to Quickstart service on IBM Watson IoT Platform.
weight: 15
---

# Quarks to Quickstart Quickly!

IoT devices running quarks applications typically connect to back-end analytic systems through a message hub.
Message hubs are used to isolate the back-end system from having to handle connections from thousands to millions of devices.

An example of such a message hub designed for the Internet of Things is
[IBM Watson IoT Platform](https://internetofthings.ibmcloud.com/). This cloud service runs on IBM's Bluemix cloud platform
and [Quarks provides a connector](http://quarks-edge.github.io/quarks/docs/javadoc/index.html?quarks/connectors/iotf/IotfDevice.html).

You can test out the service without any registration by using its Quickstart service and the Quarks sample application:
[IotfQuickstart](http://quarks-edge.github.io/quarks/docs/javadoc/index.html?quarks/samples/connectors/iotf/IotfQuickstart.html).

You can execute the class directly from Eclipse, or using the script `quarks/java8/scripts/connectors/iotf/runiotfquickstart.sh`.


