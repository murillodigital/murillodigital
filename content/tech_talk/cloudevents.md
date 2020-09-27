---
title: "The CloudEvents Specification"
description: "Interoperability and standardization in an event driven world"
date: "2020-09-21"
featured_image: "/images/cloudevents_hero.jpg"
author: "Leonardo Murillo"
images:
- "/images/cloudevents-icon-color.png"
---
# An event driven world
The world of systems architecture has been through accelerated change.

The cloud, containers and container orchestration, as well as entirely new paradigms for how you build and deploy software such as serverless architectures, have dramatically changed the patterns that govern how systems and components within those systems interact with one another.

The shift away from monoliths, and the ever-increasing scale that software needs to endure, together with an also growing amount of data as driver to a lof of this, have directed plenty of solution and systems architects towards embracing an event driven approach to their designs.

# A "Tower of Babel"
The problem is, every architect has come up with their own opinion of what the schema of an event should look like. Sometimes even across services in the same solution!

Systems must consume and publish events that vary in terms of structure and available data. It is no surprise that this diversity of structures has many downsides:

- Mapping and transformations are ever present, this impacts code reliability as well as compute capacity requirements in handling the event streams
- Developers have to write new event handling logic for every source they're consuming 
- No portability and no common libraries.

# Enter CloudEvents
[CloudEvents](https://cloudevents.io/) is an effort focused on defining a standard specification to solve these issues.

The specification considers the schema of the event, providing defined patterns as to how common as well as custom attributes should be expressed.

The specification also looks into encoding and transport of these messages, something very important towards true interoperability as it considers both binary (eg. Avro) and text based (eg. JSON) serialization and handles common transport providers such as Kafka or AMQP.

The other quite relevant objective of the initiative, is to promote the development of libraries and interfaces that expose and consume messages that satisfy the specification, in order to reduce development and integration efforts as well as to enable faster system interoperability.

CloudEvents is another of the many initiatives by the Cloud Native Computing Foundation and has received contributions from engineers working across most of the largest representatives of the industry, including Google, Amazon, Alibaba, Microsoft and Oracle.

# Integrations
Of course, for a standard such as CloudEvents to fulfill its promise, widespread adoption is indispensable, which means the spec must be fully integrated into plenty of platforms and systems.

To that end, CloudEvents has broken some initial ground with integrations across public clouds including Alibaba, Oracle Cloud and Azure, as well as being supported by important open source solutions such as OpenFaaS, Debezium and Knative.

# The specification

Now that we know what the problem is and how CloudEvents looks to solve it, let's take a quick look as to how some of the more relevant definitions in the specification actually look like. You can find the [full specification here](https://github.com/cloudevents/spec/blob/master/spec.md).

## Type system and naming constraints

The first thing we have to look into is type specification and constraints in terms of attribute names. It is possible that consuming systems may not all handle case sensitivity identically, therefore, the specification defines that al attribute names must be lower case and be strictly alphanumeric, and not longer than 20 characters.

CloudEvents allows for the following data types:

- Boolean
- Integer
- String
- Binary
- URI
- URI Reference
- Timestamp

All attributes in the message must have a type from the list above, no additional data types should be present neither in data nor metadata of the event.

## Message size limits

CloudEvents messages may be forwarded across a variety of systems and consumers, including resource constraint devices. Size limitations are very important to guarantee interoperability across this potential diversity.

The maximum size of a CloudEvents compatible message must be 64KB. This implies that publishers should always keep event payloads compact and link to data, not include data.

## Security and privacy

There is an important component of linking to data vs including data in the message: it enhances security. If a message becomes exposed to unauthorized consumers, the publisher of the data will still be able to control access to the data itself via the usual access control mechanisms.

Finally, encryption is strongly encouraged although not enforced by the specification.

## Expressing data

The standard includes a `dataschema` attribute, that can optionally refer to the schema that the data adheres to.

## SDKs

The availability of SDKs is critical towards enabling development teams to adopt the standard, and to that end CloudEvents has done a great job in building officially supported software development kits for many of the more relevant languages in use today, including JavaScript, GO, Rust, Java and Python.

## Example CloudEvent serialized as JSON:

```json
[
  {
      "specversion" : "1.x-wip",
      "type" : "com.example.someevent",
      "source" : "/mycontext/4",
      "id" : "B234-1234-1234",
      "time" : "2018-04-05T17:31:00Z",
      "comexampleextension1" : "value",
      "comexampleothervalue" : 5,
      "datacontenttype" : "application/vnd.apache.thrift.binary",
      "data_base64" : "... base64 encoded string ..."
  },
  {
      "specversion" : "1.x-wip",
      "type" : "com.example.someotherevent",
      "source" : "/mycontext/9",
      "id" : "C234-1234-1234",
      "time" : "2018-04-05T17:31:05Z",
      "comexampleextension1" : "value",
      "comexampleothervalue" : 5,
      "datacontenttype" : "application/json",
      "data" : {
          "appinfoA" : "abc",
          "appinfoB" : 123,
          "appinfoC" : true
      }
  }
]
```

# What is my opinion?

I think CloudEvents is a great initiative and one which is truly needed in the cloud development world. My experience with plenty of teams building microservice based solutions speaks to the fact that something as fundamental as the structure and exchange mechanisms of messages usually consume a lot more effort than it really should.

The fact it's reached a v1.0 release gives me some confidence it has good opportunity to reach further adoption.

However, I think the current ecosystem of integrations is still small, and the constraints the standard enforces to guarantee interoperability will be tricky for some teams and solutions to adopt. Ideally, this is a standard that should be adopted early on in designing a solution.

The one criticism I have is having `dataschema` being an optional attribute. For any consumer to be able to effectively leverage the payloads in the message in a truly independent way, a description of the data in the message is indispensable.

The metadata of the message may remain pretty consistent over time but the schema of the data itself will change, which also makes the declaration of versioned data schemas as a component of every message critical.

I for one will definitely encourage the teams I lead to explore and look towards adopting the standard, after all, as much as it may be a challenge for some of them to embrace it, the specification promotes what I consider very wise decisions in terms of message constraints and data security.

Keep up to date with cloud native technologies, join my mailing list!

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