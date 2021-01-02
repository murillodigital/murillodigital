---
title: "Observability and OpenTelemetry"
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

Telemetry, in its simplest form, means gathering data that can be used to understand performance and utilization of your application across its multiple components, in other words, proactively and programmatically deliver data that can be used to track the behavior of your system as  requests move throughout services.

Telemetry has been historically used for various purposes, from debugging and troubleshooting to monitoring, however, as cloud native solutions become common, handle ever-increasing scale, and architectures become distributed and stateless, the concept has evolved into the recent practice of _observability_.

Observability as a practice has many uses, encompassing both the more traditional cases I mentioned before (eg. monitoring and troubleshooting), but also serving as the foundation for our increasing need to understand not just the behavior of our systems, but also the impact of that behavior on our users, the value those users are deriving from our solution, and how to direct our efforts and resources towards enhancing our product's performance and user experience.

## Shifting our culture on failure

The nature of cloud native architecture, including concepts such as microservices and functions as a service, has triggered important changes in how failure is evaluated and handled.

Why is cloud native architecture producing these changes? I think there are multiple reasons to consider: 

- Cloud native infrastructure is inherently dynamic - services scale up and down, new services are added and automatically discovered, data and requests cross all sorts of boundaries.
- Cloud native architecture is highly distributed. To understand systems behavior you must track and correlate data and requests flowing through a myriad of components.
- Cloud native deployments are immutable, we don't _fix malfunctioning "things"_ anymore, we _replace malfunctioning things with new, correctly functioning ones_.

The characteristics of this new, much more complex, system paradigm produce a dramatic change in what and how we track behavior and performance, and our approach to failure. The mechanisms that we used for gaining visibility into our monolithic, or on-prem solutions simply no longer make the cut.

From my perspective, there are very positive outcomes to this -still ongoing- shift:

For one, we seem to be finally maturing from the simple realization that everything will fail, to actual acceptance, thus enabling a mindset that understands the expectation of perfect availability isn't just futile and unrealistic, but also likely much more costly than what satisfactory reliability requires.

In the words of Google on user defined reliability:

> Your systems should be reliable enough that users are happy, but not excessively reliable such that the investment is unjustified.

This new _embracing of failure as an inevitability_ has influenced everything, from design and architecture, all the way to how we manage failure and gain insights from it as opposed to trying to simply avoid it.

## New culture breeds new teams, new teams need new indicators

The paradigm shifts we've discussed above turn into a need for a new structure and sets of skills in an organization, and usually, the metrics used to guide that new set of skills also need to be redefined.

I believe there are a couple of other trends, also growing in spread and maturity, that are very much related to observability. One could argue that, without solid observability in place, it would be impossible to accomplish the transformation:

- Trend 1 - **The rise of the SRE**
- Trend 2 - **The use of SLOs and SLIs in addition to SLAs**

Both these trends are strongly dependent on the availability of meaningful data to drive decisions, design and implementation, observability is what makes that data available.

# The verticals of observability

Now that we have a bit of a sense as to where observability is coming from, and what other areas are associated to it, lets go into some detail as to what observability is all about.

In most literature, you will find references to the three pillars or verticals of observability. The pillars are the three primary types of data, each with different characteristics and objectives, that you can use to instrument your solution:

- Logs
- Metrics
- Traces

Although there are other types of observability data that can be gathered, we will focus on these three primary data types. Let's look at each one in a bit more detail:

## Logs

Logs are by far the more established and usual type of observability data that most developers already use. Logs are immutable, timestamped, free-form records of events occurring in the system. They usually provide relatively verbose information associated to process specific behavior. They're very effective in troubleshooting scenarios during development.

The effectiveness of logs decreases with scale, just as the complexity and requirements in managing and storing them increases, which is why log levels are used to increase or decrease verbosity based on the environment where the solution is running.

## Metrics

Metrics are purely quantitative indicators, representing some measured data over a period of time. Metrics are stored in time-series databases, and are enriched with metadata to enable querying and relating them to one another or components of the system. One usually finds some basic metric types in most implementations such as gauges, counters and histograms.

Metrics are very powerful tools to gain visibility as scale increases, and given their mostly numeric nature, enable engineers to perform math and computations with them.

## Traces

In modern distributed architectures, satisfying some user expectation usually involves not one, but various independent systems over a period of time. Understanding system functionality and performance on top of service oriented and microservice based architectures has always been a complicated task, thanks to the fact that user experience is actually bound to numerous systems, any of which may be experiencing degraded performance or be unavailable at some point in time.

Traces solve that problem by building a representation of events related to one another, providing end-to-end visibility of user requests as it traverses a distributed architecture. Each event in the chain of causal events is considered a _span_, a group of _spans_ is deemed a _trace_.

# What is OpenTelemetry

The OpenTelemetry project is a relatively recent initiative born out of the join of two preexisting initiative: OpenTracing and OpenCensus. The project is currently member of the Cloud Native Computing Foundation as a Sandbox initiative.

OpenTelemetry is an observability framework. What that means is, the project aims to provide tools, libraries, agents and other components required to instrument your solution across the full stack, and allow you to capture and publish telemetry data. Another important component is the definition of a common specification that guarantees a level of compatibility for all tools, internal and external to the project, that follow it.

Most OpenTelemetry components are currently in _beta_ are general availability release candidates are already in the horizon, and all the various SDKs, 

You can find the codebase for the various OpenTelemetry projects in their GitHub organization at [https://github.com/open-telemetry](https://github.com/open-telemetry), the organization holds repositories for the quite impressive list of SDKs already available for most current programming languages.

# Why is OpenTelemetry important

There are many important aspects as to why a project such as OpenTelemetry is important in the Cloud Native Computing landscape:

## Interoperability


## Extensibility

## Standardization:

## Community Outreach

# Tools and Ecosystem