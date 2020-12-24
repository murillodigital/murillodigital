---
title: "Open Telemetry"
description: "A cockpit for your cloud native solution"
date: "2020-12-21"
featured_image: "/images/opentelemetry.jpg"
author: "Leonardo Murillo"
images:
- "/images/opentelemetry.jpg"
---
Before going into Open Telemetry, the CNCF Project, I want to use the opportunity to also discuss some basic 

# What is telemetry and observability?

Telemetry, in its simplest form, means gathering data that can be used to understand performance and utilization of your application across its multiple components, in other words, programmatically track the behavior of your system as data and requests move throughout services.

Telemetry has been historically used for various purposes, from debugging and troubleshooting to monitoring, however, as cloud native solutions become common, handle ever-increasing scale, and architectures become distributed and stateless, the concept has evolved into the recent trend of _observability_.

My personal opinion is that, in the cloud native world, observability should go beyond systems-thinking and into the user

## Culture shift

Culture shift: not try to avoid failure, since it is unavoidable, rather handle failure transparently and gain insights from it.

The concerns of what we track are also changing, as we progress from bare metal, to virtualization and now into the realm of the cloud and orchestrated containers. In the past we worried a lot about whether we had provisioned enough capacity, whether our fail-over was getting traffic, or other such concerns, but going into the cloud and containers, we have different concerns, are we scaling efficiently?

It has become much more of a concern whether we have enough capacity provisioned as opposed to capping the autoscaling capacity of our solutions and gaining the necessary insights to utilize the seemingly infinite scale available to us effectively.

Another important component in this shift is how, given microservices, failures are harder to see as they usually become (as desired) very much localized.

## A new set of key performance indicators

Evolution of SLAs into SLOs and SLIs

# The verticals of observability

In the modern cloud native world, three core verticals have been identified as critical for effective systems observability:

- Logs
- Metrics
- Traces

# Deriving value from telemetry data

# The importance of telemetry in cloud native architecture

# OpenTelemetry and the Cloud Native Community

# How to start instrumenting

# Tools and ecosystem