---
title: "The Open Application Model and Crossplane - Part 1"
description: "An agnostic way to build application centric platforms and infrastructure in a team centric way, across clouds"
date: "2020-08-26"
featured_image: "/images/crossplane_oam/oam_hero.jpg"
author: "Leonardo Murillo"
---
# What you'll get out of this series

The world of cloud native engineering is reaching ever new heights in terms of flexibility, integration and elegance, all driven by adoption that rapidly gains momentum and the myriad of learning opportunities and challenges that arise from that velocity.

The Open Application Model looks to provide structure, in a vendor agnostic way, towards clearly defining the context of responsibility for each different participant  in a cross functional team as well as across teams as they build service oriented, cloud native solutions. It enables a common language and a predictable way of interaction.

In the meantime, Kubernetes solidifies its place as cornerstone of this evolution (or is it revolution?).

In this series I hope you will come out with two objectives met:

1. Understand the Open Application Model framework concept of [roles](#what-is-the-open-application-model) and [resources](#what-resources-do-the-various-roles-provide)
2. Understand how [Crossplane](#what-is-crossplane-and-the-oam-kubernetes-runtime), a Sandbox Member Project of the Cloud Native Computing Foundation, using both Crossplane as well as their OAM Kubernetes Runtime project, implement the model in Kubernetes.

This Part 1 will focus on general concepts as well as some opinionated observations.

In Part 2 ([coming soon! sign up for the mailing list to be notified when I've released it](#stay-tuned-for-part-2-of-this-series)) I'll walk you through a scenario and build all the code necesary for you to see all the concepts in action.

# What is the Open Application Model

Getting cloud native solutions built and deployed requires a varied set of skills.

I see this every day across many of the projects I'm involved in. It is impossible to expect everybody to know everything, and lacking clear roles and responsibilities leads to decisions made in haste by team members that lack some of the context or expertise necessary to guarantee production readiness and efficiency in a cloud native setting.

OAM proposes a structure that compartmentalizes these concerns onto three different roles:

![Open Application Model Roles](/images/crossplane_oam/oam-high_level.png)

All three roles should be present in cloud native projects, constantly collaborating yet specifically focused. Some of these roles may span across projects, some may be performed by the same person.

# What is each role responsible for?

How do these roles align to the usual titles you find in most projects and when companies are looking for talent? The following is my perspective of how one would map OAM roles to job descriptions.

### Platform Builders
These guys and girls are experts in cloud providers and their services, perhaps hold certifications on some specific cloud and/or set of services in one or more clouds and sometimes have specific areas of focus (eg. data).

Usual titles are _Cloud Infrastructure Engineers_ or _Cloud Architects_. Their responsibility is to abstract cloud specific concepts into generalized functionality, and customize that offering to match the availability, cost and other enterprise or corporate concerns.

They enable teams by providing infrastructure in terms each team can grasp, and doing so securely and aligned with guidelines.

### Application Developers
These are the guys and girls building the software. They are the ones that code out the products we want running on the cloud, algorithms, debugging and user stories are their area of focus.

They target features and value delivery, working with product owners to make sure that users are getting the features and experience they expect from the software they're in charge of.

Usual titles? Plenty, from _Frontend Engineers_, _Backend Engineer_, _Full Stack Developer_, _Systems Analyst_. I would even consider members of this role _QA Automation Engineers_ and _UI/UX Designers_. They are all the people involved in getting the products built.

### Application Operators
And how do you get the application built by the developers continuously integrated and delivered to the platform provided by the platform builders, as well as monitored and properly scaled and configured across multiple environments with varying sets of requirements?

That's where Application Operators come in, these are the usual roles along the lines of _DevOps Engineer_ or _Site Reliability Engineer_, they take applications following the parameters communicated by the developers, and configure them to run reliably on top of platform services.

# What resources do the various roles provide

Now that we understand the various roles lets talk about resources. Resources are types of entities that declare what each role provides to the others, what each of the roles manage towards abstracting and customizing platform, software and runtime.

![Open Application Model Interaction Between Concepts](/images/crossplane_oam/oam_interaction.png)

## Workload Definitions
Platform Builders define Workload Definitions. They represent infrastructure that is available for application developers to build their software on top of any given platform. They basically declare, in a platform agnostic way, what type of service the application developer can choose to use with their application.

An example of Workload Definition is _ContainerizedWorkload_. Application Developers may choose to run their solution in containers, but they should not have to worry as to whether it's Kubernetes or Docker Swarm what ends up running the solution in production, that complexity is abstracted in an agnostic type of workload that can mutate without impacting the applications that use it.

## Components
Components are functional units, for example, one microservice. Components are declared by application developers and specify operational capabilities.

With components, the application developer can indicate what type of workload they want to use, and provide a spec that defines the configuration characteristics it will need to apply.

Note we're talking here about declaring characteristics, not specifically configuring any of those, which will be environment specific and managed by the operators.

For example, the component can indicate that it needs a path available to store files, but it will not indicate what type of volume mount that should be or what type of technology it will use in its various deployment scenarios to get that storage.

## Application Scopes
Application Operators will use Application Scopes to group together logical applications. Imagine a microservice architected solution, each microservice built by a different team and represented as a component.

The Application Operator will group the different components from those different teams into a logical group (the full solution) by putting them together into an application scope.

## Traits and Application Configurations
Traits overlay components and workloads and are used by Application Operators to make specific decisions about the configuration of a component, for example, how autoscaling should behave or how version upgrades should be orchestrated.

Traits attach operational behavior to runtime components and abstract certain configurable characteristics of the underlying runtime platform, for example, if you think of an autocaling trait, the underlying platform may have numerous different options to configure autoscaling behavior, but the trait may expose only the two options that are most relevant to define for efficient runtime operation.

Traits are applied via Application Configurations. Application configurations define the runtime configuration for a specific instance of a component, that is, a specific deployment of a component with a specific name and version.

# DevOps and the Open Application Model
One of the most crucial objectives of the DevOps mindset is communication and collaboration.

Different skills and expertise are indispensable in order to be effective at leveraging technology across a cloud native stack, and those skill-sets should be identified and given a clear scope of action.
 
OAM enables collaboration and visibility, enabling a declarative model to define and communicate requirements at a level compatible with the different concerns of the different members of the team, a common language. From my perspective, OAM is a DevOps enabler, a mechanism to remove silos and enable transparency.

Add to that the increasing complexity of solutions, particularly in the context of service oriented or microservice architectures, and now you have multiple teams requiring infrastructure, configuring and delivering their individual components (or groups of them), testing, etc.

The OAM provides an agnostic way to share, communicate and empower the different roles in their respective responsibilities as they work together, not in silos.

# What is Crossplane and the OAM Kubernetes Runtime

[Crossplane](https://crossplane.io) is a Cloud Native Computing Foundation sandbox project that extends your Kubernetes cluster with Custom Resource Definitions (among other objects) that allow you to provision and manage infrastructure and managed services on underlying public cloud providers using the Kubernetes API and following existing tooling, pipelines, etc.

Currently Crossplane has support for a growing number of services on the Google Cloud, AWS, Azure and Alibaba Cloud.

A sister project of Crossplane is the [OAM Kubernetes Runtime](https://github.com/crossplane/oam-kubernetes-runtime), which provides more CRDs directly aligned with the Resource types from the Open Application Model.

Using Crossplane together with the OAM Kubernetes Runtime, you can manage your cloud based infrastructure as well as enable teams with OAM flow fully inside your existing Kubernetes based ecosystems, which if you ask me, is really awesome!

# Now you know the basics, in Part 2, you'll see the concepts in action

On Part 2 we will do an actual implementation of OAM and Crossplane on Kubernetes. Get notified, join my mailing list!.

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
