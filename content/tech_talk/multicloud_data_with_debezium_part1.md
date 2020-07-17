---
title: "Multicloud data with Debezium - Part 1: The architecture"
date: "2020-07-03"
description: "An Open Source way to deliver your data to any cloud"
featured_image: "/images/debezium.jpeg"
---
When thinking multi cloud, data is usually one of the more complicated concerns. Regardless of whether thinking multi cloud for disaster recovery or to better leverage each cloud's own strengths, there will always be scenarios where you need to make sure data is available and consistent across them.

And that is tricky, and usually expensive.

In this series I will demonstrate a concept in which we leverage Debezium, an _open source distributed platform for change data capture (CDC)_ as our core mechanism to get data across multiple clouds. 

**Part 1** (the article you are reading) will give you a quick introduction to what we will be building.
 
[**Part 2**]({{< ref multicloud_data_with_debezium_part2 >}}) will show you the deployment of our source data tier in AWS, this is where Debezium will be streaming data changes from.  

**Part 3** (coming soon) will demonstrate the consuming side of the architecture in Google Cloud towards data analytics, and finally  

**Part 4** (coming soon) will show you the Microsoft Azure side of the equation with a simple product availability notification service.


Cool, now what is it that we're going to be building?
-----------------------------------------------------

Here's the scenario. Your company is running their inventory database in AWS using a managed PostgreSQL RDS instance. One of your teams decided to use the Google Cloud to build a BigQuery data warehouse for doing prediction and analytics on inventory patterns, and wants to load every historical event of inventory change. Another team is using Azure for a notification engine that sends out e-mails to users who have shown interest in some product that was out of stock, when it comes back in stock.

Since the purpose of this experiment is to demonstrate the value of Debezium inside a relatively complex multi cloud architecture, we're going to mix things up by touching a varied set of services across the three clouds, what that means is, you are not going to see a homogenous set of services across them (say, for example, we could have simply used Kubernetes across all three clouds) --- this would not be an ideal scenario in the real world where you'd actually want some level of consistency.

The other aspect of our strategy, which is one you _will_ want to guarantee in any real world scenario is, everything will be written in code, we will do no manual steps and, if you check out the experiments repo, you should be able to run the experiment against each cloud with no human intervention.

Lastly, we will go for fully managed services across all clouds! Here's our service selection:

* **For compute we will use:**
  * Elastic Container Service on Fargate in AWS
  * Google Cloud Run in GCP
  * Azure Kubernetes Service in Azure
* **Data Repositories**
  * Amazon RDS in AWS using PostgreSQL
  * BigQuery in Google Cloud
* **Event streaming**
  * AWS Managed Streaming for Kafka

![Architecture Diagram - Debezium Multicloud Solution](/images/multicloud_data_with_debezium/part1_diagram.svg)

So Debezium is what again?
--------------------------

In very few words, Debezium tracks changes to data by looking at DB transaction logs, turns those changes into events and pushes them as JSON messages to Kafka for consumption by any number of receivers. This type of technology is commonly known as **CDC**, which stands for **Change Data Capture**. 

Debezium uses a [connector architecture](https://debezium.io/documentation/reference/connectors/index.html) and is built on top of Kafka Connect, their goal is to have a large library of connector for a large variety of DBMSes, and currently support:

* MongoDB
* MySQL
* PostgreSQL
* SQL Server

Plus a handful of others that are currently in preview.

I'm ready! What do I need to get started?
-----------------------------------------

You will need an account in Google Cloud, Amazon Web Services and Azure of course to get started. Bear in mind that this can get expensive if you leave it running or are not in the free tier of the platforms so be careful.

All the code I use throughout the experiment can be found [in my experiments repo in GitHub](https://github.com/murillodigital/experiments) and you will need at least the following tools:

* Terraform
* The psql CLI tools installed in your local machine
* Docker
* Kubectl
* The IDE of your choice

Ready to go? Let's get started
------------------------------

Now before we do anything, here's a disclaimer: **this experiment is not production ready**, we will be using development level resources throughout and no redundancy will be in place, do not go about and try to implement this as is in a production setting unless you're looking for a dramatic way to find a new job.

[Let's move over to part 2 and get the AWS side of the house started]({{< ref multicloud_data_with_debezium_part2 >}}).