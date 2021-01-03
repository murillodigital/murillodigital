---
title: "Observability and OpenTelemetry"
description: "A cockpit for your cloud native solution"
date: "2020-12-21"
featured_image: "/images/opentelemetry.jpg"
author: "Leonardo Murillo"
images:
- "/images/opentelemetry.jpg"
---
Before going into OpenTelemetry, the CNCF Project, I want to use the opportunity to also discuss some of the more fundamental concepts as to what telemetry and observability mean, as well as elaborate on the recent changes the industry has experienced in terms of evaluating performance, availability and more importantly reliability of digital products.

# What is telemetry and observability?

Telemetry, in its simplest form, means gathering data that can be used to understand performance and utilization of your application across its multiple components, in other words, proactively and programmatically deliver data that can be used to track the behavior of your system as  requests move throughout services.

Telemetry has been historically used for various purposes, from debugging and troubleshooting to monitoring, however, as cloud native solutions become common, handle ever-increasing scale, and architectures become distributed and stateless, the concept has evolved into the more recent practice of _observability_.

**Observability as a practice has many uses, encompassing both the more traditional cases I mentioned before (eg. monitoring and troubleshooting), but more importantly also serving as the foundation for our increasing need to understand not just the behavior of our systems, but also the impact of that behavior on our users, the value those users are deriving from our solution, and how to direct our efforts and resources towards enhancing our product's performance and user experience**.

Now, it's important to be mindful of the fact that **telemetry is not observability**, but my point here is that you can't work towards observability without considering telemetry as a foundational component.

## Shifting our culture on failure

The nature of cloud native architecture, including concepts such as microservices and functions as a service, has triggered important changes in how failure is evaluated and handled.

Why is cloud native architecture producing these changes? I think there are multiple reasons to consider: 

- Cloud native infrastructure is **inherently dynamic** - services scale up and down, new services are added and automatically discovered, data and requests cross all sorts of boundaries.
- Cloud native architecture is **highly distributed**. To understand systems behavior you must track and correlate data and requests flowing through a myriad of components.
- Cloud native deployments are **immutable**, we don't _fix malfunctioning "things"_ anymore, we _replace malfunctioning "things" with new, correctly functioning ones_.

The characteristics of this new, much more complex system paradigm produce a dramatic change in what and how we track behavior and performance, and our approach to failure. The mechanisms that we used for gaining visibility into our monolithic, or on-prem solutions simply no longer make the cut.

From my perspective, there are very positive outcomes to this -still ongoing- shift:

For one, we seem to be finally maturing from the simple realization that everything will fail, to actual acceptance, thus enabling **a mindset that understands the expectation of perfect availability isn't just futile and unrealistic, but also likely much more costly than what satisfactory reliability requires**.

In the words of Google on user defined reliability:

> Your systems should be reliable enough that users are happy, but not excessively reliable such that the investment is unjustified.

This new _embracing of failure as an inevitability_ has influenced everything, from design and architecture, all the way to how we manage failure and gain insights from it as opposed to trying to simply avoid it.

## New culture breeds new teams, new teams need new indicators

The paradigm shifts we've discussed above turn into a need for a new structure and sets of skills in an organization, and usually, the metrics used to guide that new set of skills also need to be redefined.

I believe there are a couple of other trends, also growing in spread and maturity, that are very much related to observability. One could argue that, without solid observability in place, it would be impossible to accomplish the transformation:

- Trend 1 - **The rise of the SRE**
- Trend 2 - **The use of SLOs and SLIs in addition to SLAs**

**Both these trends are strongly dependent on the availability of meaningful data to drive decisions, design and implementation, and telemetry is what makes that data available.**

# The verticals of observability

Now that we have a bit of a sense as to where observability is coming from, and what other areas are associated to it, lets go into some detail as to what observability is all about.

In most literature, you will find references to the three pillars or verticals of observability. The pillars are the three primary types of data, each with different characteristics and objectives, that you can use to instrument your solution:

- Logs
- Metrics
- Traces

Although there are other types of observability data that can be gathered, we will focus on these three primary data types. Let's look at each one in a bit more detail:

## Logs

Logs are by far the more established and usual type of telemetry data that most developers already use. Logs are immutable, timestamped, free-form records of events occurring in the system.

They usually provide relatively verbose information associated to process specific behavior, and are very effective in troubleshooting scenarios during development.

The effectiveness of logs decreases with scale, just as the complexity and requirements in managing and storing them increases, which is why log levels are used to increase or decrease verbosity based on the environment where the solution is running.

## Metrics

Metrics are purely quantitative indicators, representing some measured data over a period of time.

Metrics are stored in time-series databases, and are enriched with metadata to enable querying and relating them to one another or components of the system. One usually finds some basic metric types in most implementations such as gauges, counters and histograms.

Metrics are very powerful tools to gain visibility as scale increases, and given their mostly numeric nature, allow for engineers to perform math and computations with them.

## Traces

In modern distributed architectures, satisfying some user expectation usually involves not one, but various independent systems over a period of time.

Understanding system functionality and performance on top of service oriented and microservice based architectures has always been a complicated task, due to the fact that user experience is actually bound to numerous systems, any of which may be experiencing degraded performance or be unavailable at some point in time.

Traces solve that problem by building a representation of events related to one another, providing end-to-end visibility of user requests as they traverse a distributed architecture. Each event in the chain of causal events is considered a _span_, a group of _spans_ is deemed a _trace_.

# What is OpenTelemetry

The OpenTelemetry project is a relatively recent initiative born out of the union of two preexisting initiative: OpenTracing and OpenCensus. The project is currently member of the Cloud Native Computing Foundation as a Sandbox initiative.

**OpenTelemetry is an observability framework. What that means is, the project aims to provide tools, libraries, agents and other components required to instrument your solution across the full stack, and allow you to capture and publish telemetry data. Another important component is the definition of a common specification that guarantees a level of compatibility for all tools, internal and external to the project, that follow it.**

Most OpenTelemetry components are currently in _beta_ and general availability release candidates are already in the horizon.

You can find the codebase for the various OpenTelemetry projects in their GitHub organization at [https://github.com/open-telemetry](https://github.com/open-telemetry), the organization holds repositories for the quite impressive list of SDKs already available for most current programming languages.

# Why is OpenTelemetry important

There are many aspects as to why a project such as OpenTelemetry is important in the Cloud Native Computing landscape.

Instrumenting a product is something that every team *should* do, which means the aggregated effort into getting the basics in place as well as long term supporting "homegrown" solutions would be tremendously expensive to the industry were there no available alternatives to provide a more effective starting point.

It's also important to mention that this lowering of the entry barrier will support the adoption of best practices, and support developers in "doing the right thing" amidst mounting release and project pressures.

Other more technical reasons a project such as OpenTelemetry is relevant also come to mind:

## Interoperability

The wide variety of SDKs and the availability of a common set of concepts exposed by those development tools allows for solutions built by different teams and even using different technologies to still be able to provide telemetry data consistently.

This is particularly important in the world of microservices and distributed solutions, as well as towards supporting the progressive evolution of technology, where services can be independently deprecated and replaced with newer versions built using different stacks and development models, while still being able to provide telemetry data transparently to existing backends. 

## Extensibility

The [OpenTelemetry Registry](https://opentelemetry.io/registry/) serves as a clear example of the extensibility benefits of a common framework. In the registry you will be able to find as well as contribute _libraries, plugins, integrations and other useful tools_ for a wide variety of programming languages, all compatible with the OpenTelemetry specification.

## Standardization

Of course, a lot of the benefits of interoperability and extensibility stem from the fact that the framework defines a common standard for protocols, semantics and vocabulary.

The [OpenTelemetry Specification](https://github.com/open-telemetry/opentelemetry-specification) _"... describes the cross-language requirements and expectations for all OpenTelemetry implementations ..."_. The specification is quite comprehensive and properly versioned, not too mention open for community feedback and directed by a committee.

## Community Outreach

Of course, being an open source initiative, the [community](https://github.com/open-telemetry/community) aspect is fundamental, and IMHO one of the best characteristics of projects under the CNCF umbrella. I am a true believer in the power of community for innovation and technological progress.

# The OpenTelemetry Framework

As mentioned, the OpenTelemetry Framework looks to build tools and libraries for a wide variety of languages. To close up I'm going to link to some of the more relevant ones, taken out of the [official documentation](https://opentelemetry.io/docs/)

## SDKs

There are SDKs for all current major high-level languages, including [Javascript](https://opentelemetry.io/docs/js/), [.NET](https://opentelemetry.io/docs/net/), [Go](https://opentelemetry.io/docs/go/), [Java](https://opentelemetry.io/docs/java/) and [Python](https://opentelemetry.io/docs/python/)

## The Collector

[The Collector](https://opentelemetry.io/docs/collector/) is an important component of the OpenTelemetry Framework, as it receives telemetry data published or exposed by any compliant service, and takes care of receiving, processing and pushing the data into one or more backends. It's worthy of notice that the benefit of using the collector is that your solution no longer needs to worry about sending its data to multiple, proprietary backends, rather to a single endpoint that then makes the data available to the backend of your choice.

## Limitations

There is currently one important limitation in the OpenTelemetry Framework, which is its lack of support for logs. I think this is understandable considering the much longer history and available solutions both open source and commercial to handle log data, not to mention the nature of the data itself introduces some interesting challenges to be able to build a spec around it.

Nevertheless, there is some expressed interest to incorporate logging into the framework in the future. In the meantime, there are well established solutions out there, including the widely used [ELK](https://www.elastic.co/what-is/elk-stack) and [EFK](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes) stacks.

---

On my next article in this series we will do an actual demo implementation by instrumenting two services written using different languages. Stay tuned for updates! Join my mailing list!.

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

