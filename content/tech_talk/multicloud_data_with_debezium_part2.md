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

# Getting set up

## Clone the repository

All the code you need to deploy this solution can be found at [github.com/murillodigital/experiments](https://github.com/murillodigital/experiments) inside a directory named `multicloud_data_with_debezium` and there you will find a subdirectory for every cloud we will be working on, today we're going to be looking into the `aws` folder.

```bash
$ git clone git@github.com:murillodigital/experiments
Cloning into 'experiments'...
remote: Enumerating objects: 54, done.
remote: Counting objects: 100% (54/54), done.
remote: Compressing objects: 100% (35/35), done.
remote: Total 54 (delta 15), reused 46 (delta 10), pack-reused 0
Receiving objects: 100% (54/54), 11.61 KiB | 11.61 MiB/s, done.
Resolving deltas: 100% (15/15), done.
$ cd experiments/multicloud_data_with_debezium/aws/terraform/
$ ls
bastion.tf	ecs.tf		msk.tf		outputs.tf	variables.tf
database.tf	main.tf		network.tf	templates
```

## Setup Terraform to use your AWS account

My preferred method for enabling Terraform to authenticate against AWS is creating an Access Key and making it available to Terraform as an environment variable. Amazon has very good documentation on their website, [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) you can see how to create the necessary tokens. Once you're done with that, you will have two pieces of information, an `Access Key ID` and a `Secret Access Key`. On the terminal where you will be running terraform, use those to export the following variables:

```bash
$ export AWS_ACCESS_KEY_ID=yourAccessKeyId
$ export AWS_SECRET_ACCESS_KEY=yourSecretAccessKey
$ export AWS_DEFAULT_REGION=us-east-1
```

As you can see you can also define a default region, by exporting the variable `AWS_DEFAULT_REGION`.

## Create an SSH key for the bastion host

Follow [these instructions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#prepare-key-pair) to create a keypair for our bastion host, you don't have to do this if you already have a keypair in AWS you want to use, all we will need is the key pair name.

![AWS SSH Key Pair Name](/images/multicloud_data_with_debezium/part2-screenshot1-keypair.png)

# Let's look into the terraform code

For simplicity, I'm not using modules rather have split all resources into individual files depending on their purpose.

```bash
-rw-r--r--  1 leonardomurillo  staff   2099 Jul 16 13:52 bastion.tf
-rw-r--r--  1 leonardomurillo  staff   1113 Jul 16 06:12 database.tf
-rw-r--r--  1 leonardomurillo  staff   2665 Jul 16 06:28 ecs.tf
-rw-r--r--  1 leonardomurillo  staff     41 Jul 15 19:14 main.tf
-rw-r--r--  1 leonardomurillo  staff   1330 Jul 16 05:26 msk.tf
-rw-r--r--  1 leonardomurillo  staff   1340 Jul 16 06:53 network.tf
-rw-r--r--  1 leonardomurillo  staff    702 Jul 16 05:50 outputs.tf
drwxr-xr-x  5 leonardomurillo  staff    160 Jul 15 05:56 templates
-rw-r--r--  1 leonardomurillo  staff    255 Jul 16 05:43 variables.tf
```

`main.tf` holds only the provider definition, the rest of the files should be self-explanatory by their name. Let's do a quick review of each:

### network.tf

We are using AWS default VPC and subnets for creating our infrastructure, this is not ideal in a production setting but it dramatically simplifies the amount of network _wiring_ we need to create. Terraform includes some handy resources that do not create rather lookup these default resources so you can refer them elsewhere in your code

```hcl
resource "aws_default_vpc" "default_vpc" { }

resource "aws_default_subnet" "default_az1" {
  availability_zone = "us-east-1a"
}

resource "aws_default_subnet" "default_az2" {
  availability_zone = "us-east-1b"
}

resource "aws_default_subnet" "default_az3" {
  availability_zone = "us-east-1c"
}
```

The only other two resources we create here are both `internal` and `external` security groups.

### msk.tf

> _MSK Configurations cannot be deleted, this is a limitation by AWS. To go around that problem, we're adding a random string to its name._

> _The default MSK configuration will not work with Debezium. Auto topic creation must be enabled for Debezium to dynamically generate topics based on the schema of the database. A replication factor must also be defined, otherwise you will run into a **NOT_ENOUGH_REPLICAS** error_


# Ready to go! Deploy!
That's it, now you are ready to deploy the terraformed infrastructure. From the `aws/terraform` subdirect