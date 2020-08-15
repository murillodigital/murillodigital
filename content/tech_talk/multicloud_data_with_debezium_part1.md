---
title: "Multicloud data with Debezium - Part 1: The architecture"
date: "2020-07-03"
description: "An Open Source way to deliver your data to any cloud"
featured_image: "/images/debezium.jpeg"
---
## Multicloud Data with Debezium

When thinking multi cloud, data is usually one of the more complicated concerns. Regardless of whether thinking multi cloud for disaster recovery or to better leverage each cloud's own strengths, there will always be scenarios where you need to make sure data is available and consistent across multiple cloud platforms.

In this series I will demonstrate a concept leveraging Debezium, an _open source distributed platform for change data capture (CDC)_ as our core mechanism to get data across multiple clouds. 

**Part 1** (the article you are reading) will give you a quick introduction to what we will be building.
 
[**Part 2**]({{< ref multicloud_data_with_debezium_part2 >}}) will show you the deployment of our source data tier in AWS, this is where Debezium will be streaming data changes from.  

[**Part 3**]({{< ref multicloud_data_with_debezium_part3 >}}) will demonstrate the consuming side of the architecture in Google Cloud, by storing all changes to our data in BigQuery.


Great! Let's define the scenario
--------------------------------

Your company is running its inventory database in AWS using a managed PostgreSQL RDS instance. One of your teams decided to use the Google Cloud to build a BigQuery data warehouse for doing prediction and analytics on inventory patterns, and wants to load every historical event of inventory change.

Since the purpose of this experiment is to demonstrate the value of Debezium inside a relatively complex multi cloud architecture, we're going to mix things up by touching a varied set of services across both clouds.

The other aspect of our strategy, which is one you _will_ want to guarantee in any real world scenario is, everything will be written in code, we will do no manual steps to deploy the infrastructure and, if you check out the experiments repo, you should be able to run the experiment against each cloud with no human intervention assuming you have created the required AWS and GCP accounts.

One final requirements we'll be aiming for is to leverage managed service as much as possible. Here's our service selection:

* **For compute we will use:**
  * Elastic Container Service on Fargate in AWS
* **Data Repositories**
  * Amazon RDS in AWS using PostgreSQL
  * BigQuery in Google Cloud
* **Event streaming and Data Pipelines**
  * AWS Managed Streaming for Kafka
  * Apache Beam Running on GCE

![Architecture Diagram - Debezium Multicloud Solution](/images/multicloud_data_with_debezium/part1_diagram.svg)

A quick revisit to Debezium
--------------------------

In very few words, Debezium tracks changes to data by looking at DB transaction logs, turns those changes into events and pushes them as JSON messages to Kafka for consumption by any number of receivers. This type of technology is commonly known as **CDC**, which stands for **Change Data Capture**. 

Debezium uses a [connector architecture](https://debezium.io/documentation/reference/connectors/index.html) and is built on top of Kafka Connect, their goal is to have a large library of connector for a large variety of DBMS, and currently support:

* MongoDB
* MySQL
* PostgreSQL
* SQL Server

Plus a handful of others that are currently in preview.

Here's what you need to get started
-----------------------------------------

You will need accounts in both Google Cloud and Amazon Web Services to get started. Bear in mind that this can get expensive if you leave it running or are not in the free tier of the platforms so be careful.

All the code I use throughout the experiment can be found [in my experiments repo in GitHub](https://github.com/murillodigital/experiments) and you will need at least the following tools:

* Terraform
* The AWS CLI
* The gcloud SDK
* The IDE of your choice

> Two important caveats: In AWS, you may run an issue with the required IAM roles for ECS on Fargate not existing the first time you run the code if you have never used the service before, if that happens, just wait a few minutes and try again. On GCP you will need to have multiple APIs enabled, including Compute Engine and BigQuery, if running the code throws any errors due to APIs disabled, simply go into the GCP Console and enable the required API.

Ready to get started? Move on to Part 2
------------------------------

Now before we do anything, here's a disclaimer: **this experiment is not production ready**, do not go about and try to implement this as is in a production setting unless you're looking for a dramatic way to find a new job.

[Let's move over to part 2 and get the AWS side of the house started]({{< ref multicloud_data_with_debezium_part2 >}}).

Stay up to date with new experiments, join my mailing list!.

<!-- Begin Mailchimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/horizontal-slim-10_7.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background:#fff; clear:left; font:14px Helvetica,Arial,sans-serif; width:100%;}
	/* Add your own Mailchimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="https://murillodigital.us10.list-manage.com/subscribe/post?u=c12ff1afa71003663de3762cc&amp;id=4cff0f72fe" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<label for="mce-EMAIL">Subscribe</label>
	<input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required>
    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_c12ff1afa71003663de3762cc_4cff0f72fe" tabindex="-1" value=""></div>
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
    </div>
</form>
</div>

<!--End mc_embed_signup-->