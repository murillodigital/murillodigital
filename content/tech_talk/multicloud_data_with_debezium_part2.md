---
title: "Multicloud data with Debezium - Part 2: Data source in AWS"
date: "2020-07-10"
description: "Our data source, the inventory database running in Amazon Web Services"
featured_image: "/images/debezium.jpeg"
---
Welcome to Part 2 of this 4-part experiment on Multicloud Data with Debezium - in this article we will be building the **source of our data, and will be implementing Debezium for tracking and streaming data changes in AWS**.

For those that just got here and need a quick intro or if you need to know which tools you'll need installed to deploy this solution, take a look at [part 1]({{<  ref multicloud_data_with_debezium_part1 >}}) of this article.

Before we do anything, the same disclaimer you will see across all four parts: **this experiment is not production ready**, we will be using development level resources throughout and no redundancy will be in place, do not go about and try to implement this as is in a production setting unless you're looking for a dramatic way to find a new job.

# Prerequisites and Codebase

To deploy this experiment yourself, you will need the following:

* An AWS account, with an active billing method (MSK is not available in AWS Free Tier)
* Terraform
* A SSH key for the bastion host

You can find the full codebase in the following repository:   
[github.com/murillodigital](https://github.com/murillodigital/experiments/tree/master/multicloud_data_with_debezium)

# Architecture

![Architecture Diagram - AWS Architecture](/images/multicloud_data_with_debezium/part2_aws_diagram.svg)


# Resource Walkthrough

We will be working on a single AWS region, and to make our lives easier, will be using AWS's default VPC and subnets. We will be using only two subnets and as mentioned above, are not going for a fully HA implementation.

A two node Managed Streaming For Kafka (MSK) cluster will be deployed, each node in a separate AZ. A single RDS instance and a Bastion host will be created in a single AZ all within the same VPC. The bastion host will be used both for bootstrapping the Database and registering the Debezium Connector as well as to be able to work against the database. We need this bastion host since, as I'll explain below, neither the database nor the Kafka cluster are available over the public internet - even though they are in a public subnet, they're not configured for public access and additionally are protected by a security group which allows connections only from services that share the same SG.

Debezium will be running as a container inside a Elastic Container Service cluster, which will be using Fargate for capacity - this is ideal as we will not have to manage any sort of autoscaling or instance registration against the cluster. A single service definition will spin up a single task that is Debezium, logs will be shipped over to CloudWatch logs and, in order for us to reach the task on a friendly name, an Application Load Balancer will target the task and send traffic to it.

In terms of security, although this is not quite ready for production, we are taking some measures to provide a basic level of protection. Two security groups will be created: one `external` which will allow access via SSH to the Bastion host as well as allow connections from anywhere to the application load balancer on port 80 and one `internal` that enables all services inside AWS to communicate with one another, but not to be reached from anything public. 
