---
title: "OpenSource IoT with SiteWhere"
date: "2020-07-21"
description: "Open Source, Kubernetes Native platform for IoT Device Management"
featured_image: "/images/debezium.jpeg"
draft: true
---
# Managing IoT devices while staying cloud lock-in free

Most public cloud offer some managed service towards handling internet of things devices' usual concerns, including registration, monitoring and management.

AWS IoT Device Management
Azure IoT Hub
GCP Cloud IoT Core
IBM Cloud Watson IoT Platform
Oracle Cloud IoT Cloud Service

The primary problem with any of those services is cloud lock-in, once you've built your architecture on top of them, switching requires huge effort, particularly when you have a fleet of devices in the field that may need to be reconfigured or perhaps even refactored.

# SiteWhere is a Kubernetes native, truly open source alternative

I ran into SiteWhere looking for alternatives to this issues. My question was, _is there any free, truly open source platform, capable of production, enterprise grade operation of a large scale IoT devices fleet?_

SiteWhere checked all the boxes, built with microservices, using extremely scalable and proven technologies to handle data, built with cloud native concepts in mind, and native to Kubernetes!

This experiment looks to validate SiteWhere's effectiveness as a truly open source alternative to IoT management, and is my first experience with the tool.

Note - I'll be looking into SiteWhere 3.0 which is currently in Beta but nearing General Availability, so it may be a bit of a bumpy ride.

The community seems quite active and one of the key maintainers (Derek Adams) is very much supportive on it.

# What are we looking to validate

SiteWhere will work in two public clouds, and switching devices from one to the other will be a seamless process.

In order to accomplish this, there are two primary areas of concern.

We need a way to migrate the data from the IoT cluster over from one cloud to the other, and we need to make sure registered devices can find the new cloud


# The setup and our scenario

I'm going to start by using a GKE cluster in the Google Cloud for this experiment, and then I'll be migrating over to an AKS service in Azure. To demonstrate the IoT side of things, I'll be using a handful of devices I have laying around, including a couple of old iPhones and an Android device.

# The codebase

As usual, I will automate the whole provisioning process, use the code in the terraform directory to create the required infrastructure in GCP and Azure, and start.sh to bootstrap the services.