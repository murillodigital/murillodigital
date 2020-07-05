---
title: "Multicloud data with Debezium"
date: "2020-07-03"
description: "An Open Source way to deliver your data to any cloud"
featured_image: "/images/debezium.jpeg"
---
When thinking multi cloud, data is usually one of the more complicated concerns. Regardless of whether thinking multi cloud for disaster recovery or to better leverage each cloud's own strengths, there will always be scenarios where you need to make sure data is available and/or consistent across them.

And that is tricky, and usually expensive.

In this article I will demonstrate a concept in which we leverage Debezium, an _open source distributed platform for change data capture_ as our core mechanism to get data across multiple clouds.

So Debezium is what again?
--------------------------

In very few words, Debezium tracks changes to data by looking at DB transaction logs, turns those changes into events and pushes them as JSON messages to Kafka for consumption by any number of receivers. 

Cool, now what is it that we're going to be building?
-----------------------------------------------------

Here's the scenario. Your company is running their inventory database in AWS using a managed PostgreSQL RDS instance. One of your teams decided to use the Google Cloud to build a BigQuery data warehouse for doing prediction and analytics on inventory patterns, and wants to load every historical event of inventory change. Another team is using Azure for a notification engine that sends out e-mails to users who have shown interest in some product that was out of stock, when it comes back in stock.

Now to make this interesting, we are going to use a few additional services that I'm sure you'll all appreciate. For Kafka we will actually use AWS Managed Streaming for Kafka (who wants to manage a full Kafka cluster?), in Azure we are going to be using Azure Kubernetes Service, and in GCP we're going to use Kafka Connect with the BigQuery Connector. Excited yet? This is going to be a really fun experiment.

![Architecture Diagram - Debezium Multicloud Solution](/images/debezium_diagram.svg)

I'm ready! What do I need to get started?
-----------------------------------------

You will need an account in Google Cloud, Amazon Web Services and Azure of course to get started. Bare in mind that this can get expensive if you leave it running or are not in the free tier of the platforms so be careful.

All the code I use throughout the experiment can be found [in my experiments repo in GitHub](https://github.com/murillodigital/experiments) and you will need at least the following tools:

* Terraform
* Docker
* Kubectl
* The IDE of your choice

Ready to go? Let's get started
------------------------------

Now before we do anything, here's a disclaimer: **this experiment is not production ready**, we will be using development level resources throughout and no redundancy will be in place, do not go about and try to implement this as is in a production setting unless you're looking for a dramatic way to find a new job.

Before we start provisioning any infrastructure, let's consider data structures.

**Our Inventory on AWS**

Our inventory system will have a very basic data base of products:

```
+-------------------------------------------------------+
| Name                 |   Type             | Default   |
+----------------------+--------------------+-----------+
| id                   |  uuid              | uuid()    |
| sku                  |  char(15)          |           |
| name                 |  varchar(150)      |           |
| description          |  text              |           |
| price                |  double precision  |           |
| available            |  int               |           |
+----------------------+--------------------+-----------+
```