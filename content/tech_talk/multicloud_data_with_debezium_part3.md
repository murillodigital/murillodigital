---
title: "Multicloud data with Debezium - Part 3: Data Change History in BigQuery"
date: "2020-07-21"
description: "Using BigQuery in GCP as our historical data change repository"
featured_image: "/images/debezium.jpeg"
draft: true
---
Welcome to Part 3 of this series on multicloud data with Debezium. For those just joining us, in [part 1]({{< ref multicloud_data_with_debezium_part1 >}}) you can learn about the high level architecture of what we will be building and get a quick introduction to Debezium as a change data capture tool.

In [Part 2]({{< ref multicloud_data_with_debezium_part2 >}}) we start building our solution, specifically the AWS side of things, where our data will originate.

We are now going to look in this 3rd part into consuming this data from a different cloud, the Google Cloud Platform!

Before we do anything, the same disclaimer you will see across all four parts: **this experiment is not production ready**, we will be using development level resources throughout and no redundancy will be in place, do not go about and try to implement this as is in a production setting unless you're looking for a dramatic way to find a new job.

# Prerequisites and codebase

We are now getting ready to start working with multiple clouds at the same time, so all prerequisites defined on [part 2]({{< ref multicloud_data_with_debezium_part2 >}}) are also required, plus these new requirements:

* A Google Cloud account with billing enabled.
* The gcloud command line tools. [Click here](https://cloud.google.com/sdk/install) for install instructions.
* A Google Cloud Project attached to a billing account.
* The Compute API enabled in your GCP project ([gcp getting started instructions](https://cloud.google.com/apis/docs/getting-started))
* A service account with the necessary rights for Terraform to create resources inside the GCP project

# Some initial preparations

The focus of this article is enabling the secure connectivity between GCP and AWS, and consuming data from AWS Kafka into BigQuery, therefore I will not look into setting up the basic account details in GCP. If you need directions on how to create a project in GCP and attach it to a billing account, [this page](https://cloud.google.com/resource-manager/docs/creating-managing-projects) will give you details on how to manage projects and [here](https://cloud.google.com/billing/docs/how-to/modify-project) you can learn how to manage your project's billing.

# Architecture

