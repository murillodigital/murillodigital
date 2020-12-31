---
title: "Open Telemetry"
description: "A cockpit for your cloud native solution"
date: "2020-12-21"
featured_image: "/images/opentelemetry.jpg"
author: "Leonardo Murillo"
images:
- "/images/opentelemetry.jpg"
draft: true
---
Before going into Open Telemetry, the CNCF Project, I want to use the opportunity to also discuss some of the more fundamental concepts as to what telemetry and observability mean, as well as elaborate on the recent changes the industry has experienced in terms of evaluating performance, availability and more importantly reliability of digital products.

# What is telemetry and observability?

Telemetry, in its simplest form, means gathering data that can be used to understand performance and utilization of your application across its multiple components, in other words, proactively and programmatically deliver data that can be used to track the behavior of your system as data and requests move throughout services.

Telemetry has been historically used for various purposes, from debugging and troubleshooting to monitoring, however, as cloud native solutions become common, handle ever-increasing scale, and architectures become distributed and stateless, the concept has evolved into the recent trend of _observability_.

Observability as a practice has many practical uses, encompassing both the more traditional use cases I mentioned before (eg. monitoring and troubleshooting), but also serves as the foundation for a new trend into understanding not just the bevahior of your system, but also the impact that behavior has on your users, the value those users are deriving from your solution, and direct the efforts and resources dedicated towards enhancing your product's performance and user experience.

## Culture shift

The nature of cloud native architecture, including concepts such as microservices and functions as a service, have triggered an important change in how failure is evaluated and handled.

Why is cloud native architecture producing this change? I think there are multiple issues to consider: 

- Cloud native infrastructure is inherently dynamic - services scale up and down, new services are added and automatically discovered, data and requests cross all sorts of boundaries.
- Cloud native architecture is highly distributed. To understand systems behavior you must track and correlate data and request flowing through a myriad of components.
- Cloud native deployments are immutable, we don't _fix malfunctioning "things"_ anymore, we _replace malfunctioning things with new, correctly functioning ones_.

The characteristics of this new system paradigm produce a dramatic change in what and how we track behavior and performance, and our approach to failure. From my perspective, there are very positive outcomes of this (still ongoing) shift. For one, we seem to be finally maturing from the simple realization that everything will fail, to actual acceptance, thus enabling a mindset that knows that expecting perfect availability isn't just futile and unrealistic, but also likely much more costly than what satisfactory reliability requires.

On the one hand, the realization that things will always fail has influenced everything, from designing for failure to begin with, all the way to not expecting to avoid failure rather manage and learn from it.

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