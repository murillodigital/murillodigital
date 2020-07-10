---
title: "Multicloud data with Debezium - Part 2"
date: "2020-07-10"
description: "Our data source, the inventory database running in Amazon Web Services"
featured_image: "/images/debezium.jpeg"
---
Welcome to Part 2 of this 4-part experiment on Multicloud Data with Debezium - in this article we will be building the source of our data, and will be implementing Debezium for tracking and streaming data changes in AWS.

For those that just got here and need a quick intro, take a look at [part 1]({{<  ref multicloud_data_with_debezium_part1 >}}) of this article.

Before we do anything, the same disclaimer you will see across all four parts: **this experiment is not production ready**, we will be using development level resources throughout and no redundancy will be in place, do not go about and try to implement this as is in a production setting unless you're looking for a dramatic way to find a new job.

