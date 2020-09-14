---
title: "FinOps"
description: "Collaborative management of the finances of the cloud"
date: "2020-09-13"
featured_image: "/images/finops.jpg"
author: "Leonardo Murillo"
images:
- "/images/finops.jpg"
draft: true
---
# The financials peril of infinite capacity

I see a very common and dangerous pattern across many of the teams I support, with regard to their cloud utilization habits: there isn't a clear sense of the financial impact of their development output.

What I mean is, teams working with the cloud have no fiscal responsibility as to the expenses and budget they're working with - and this unawareness triggers bad behaviors:

- A simplistic "throw more compute at it" solution to most problems really caused by sloppy engineering practices.
- A usual compromise of "order for speed". As they test and iterate, resources get left behind, experiments become abandoned but left partially running slowly adding up to gigantic cloud spend.

To aggravate this, companies struggle to have the right visibility into their cloud spend, and have a hard time truly identifying what are reasonable capacity numbers and resource constraints for the different deployment stages of their products.

Which constraints control spend and which limit agility? What is the real capacity I need in production? Is the organization paying for technical debt with cloud resources?

These are all common questions that produce ongoing friction between business and financial stakeholder and the teams building technical solutions.

# How can you focus on something you can't see?

The good thing is, the problems I mention are not really caused by ill intention, it really is an issue of awareness.

We all have a tendency of missing the forest for the trees, and this is particularly true when you have teams marching on for features and getting very clearly defined expectations in terms of functionality and time but a very poor definition of financial constraints.

The cloud feels, to most engineers building products, as infinite and quasi-free, while time and features are very clearly defined and have a very clear sense of "cost" to them: build XYZ within this specific timeframe - period. There is usually never a consideration for budget in any spec.

If you have a perception of infinite resources on one hand and a very much finite specification on the other, what do you think is bound to happen?

# Collaboration and ownership - a proven model

Solving these problems is the focus of FinOps. Following a similar model to DevOps and _"{Fill In The Acronym}Ops"_, FinOps looks to create a cultural change in the organization by getting the various stakeholders in the operation of cloud solutions to collaborate, and for cross-functional teams to take ownership of cloud utilization.

Cloud financials become everybody's concern, driven by a central effort with participation from financial, business and technical stakeholder, to define best practices that allow teams to make intelligent decisions as to how to balance their resources.

# The three phases of FinOps

As with most OtherOps, a reiterating cycle drives continuous improvement towards allowing teams to be more accountable and efficient with their cloud investment. The phases of FinOps are:

## Inform

Knowledge is power, and in the case of FinOps this knowledge flows in multiple direction:

- Get visibility into your cloud costs: if you don't know where your cloud budget is going you will not be able to provide proper direction to your teams nor manage your spend effectively. Those with financials visibility must communication with technical stakeholders, so they understand the ROI expectations, budget, etc.

- Understand the capacity requirements and come up with a mechanism to identify reasonable costs. Technical teams should build a clear framework to determine true capacity requirements, and have the maturity to look hard at their engineering practices and determine whether they're paying for technical debt or poor engineering with cloud currency. I personally think the definition of cost/load metrics is ideal to support this type of decision.

If you think in terms of data processing solutions, can you identify how much it costs you to process each GB of data? If you are taking orders, what is your total infrastructure cost for number of hours per day? This type of thinking doesn't always come naturally to the technical teams, but it is part of the effort of informing to support them in building up that type of mindset.

After all, it's a mindset that resonates well with the consumption based pricing model of most cloud services.

## Optimize

You've got your teams on a path to better understanding their cloud spend, they're now more fiscally responsible for their running environments, and you have better visibility into your cloud resources.

Now it's time top optimize! The cloud allows for many opportunities to reduce cost, however, one thing is very important: given the cloud's elastic nature, real time visibility is a critical component to successful optimization.

Some relatively simple ways to reduce cost will require a clear understanding of the pricing model of each cloud, as well as having a good sense of capacity:

- Most cloud providers offer dramatic cost savings when you commit to resources over a period of time - once you are able to identify your ongoing minimum capacity, you can start reserving it and pay much less.

- Clean up! I'm sure in most organizations left-over resources account for a non-trivial amount of spend, but regular (ideally automated) spring cleaning will start lowering your cloud bill with relative ease.

- Use autoscaling effectively. This may sound simpler than it is, depending on how you've built your applications, but autoscaling is a simple way to move from heavy ongoing costs to dynamic resource allocation. Be careful because this can become a two sided sword: if not properly managed or if your software is not well written to support it, autoscaling can end up having the exact opposite effect and cost you huge sums due to runaway capacity. Learn to use it well! 

## Operate

Now it's time to run with it, and track the metrics and objectives you built during the first two phases. Operation is a combination of measurement and culture. As you run your solutions maintain a clear view of the evolution of the financial impact of its operation, and dedicate focused effort in cementing and evolving the necessary team culture that guarantees continuous improvement.

# Don't mistake FinOps as just about saving money

The objective of FinOps is not (just) to save money - it is quite the opposite, it is to make money! To improve cloud utilization financials means better return on investment, having proper visibility and metrics in place results in your teams having less constraints, they're able to move faster because they feel safe and empowered in that what they're building is indeed not just technically but also financially great.

It will show new opportunities for delivering better solutions and encourage teams to explore new technologies that will make their software faster and more scalable. FinOps is a driver to a cloud native mentality.

# FinOps is a component of security

Information, real-time visibility and automated constraints become ever more critical when there is ill will. I mentioned above most of the time, teams are over spending and sub optimizing not out of malicious intent, rather lack of information - but that will not always be the case.

A disgruntled employee can cause tremendous financial damage if let to run wild on a poorly managed cloud environment.

# You will need the help of tools

Managing costs and spend across regions and zones, on top of resources that are absolutely elastic and can go from 0 to breaking the bank in seconds, is not a job for humans alone. The advent of multi-cloud doesn't make this any easier, since now you will need to worry about multiple pricing models and deploy control mechanisms to completely different cloud platforms.

The landscape of solutions that provide visibility is growing, although I'd say it is still not as rich as into other areas of the cloud.

## Open source, commercial and cloud specific

A good open source alternative to start building some control into your cloud costs is [Cloud Custodian](https://cloudcustodian.io/). It supports AWS, Azure and GCP and uses a yaml DSL to declare your rules and constraints.

Every public cloud has some native tools to provide visibility, forecasting and basic budget related alerts. Here are some references:

- [Google Cloud Cost Management](https://cloud.google.com/cost-management)
- [Azure Cost Management](https://azure.microsoft.com/en-us/blog/azure-cost-management-now-generally-available-for-pay-as-you-go-customers/)
- [AWS Cost Management](https://aws.amazon.com/aws-cost-management/)

In 2019 an initiative called the [FinOps Foundation](https://www.finops.org) was established, under the umbrella of the Linux Foundation. The FinOps foundation's members represent multiple commercial solutions towards gaining control and better value from cloud spend, and even provide a certifications for individuals, services and platforms that follow the FinOps guidelines.

# A requirement towards cloud maturity

In closing, I believe the FinOps model is a clear representation of the growing maturity of the cloud ecosystem, and all organizations would be wise to start working towards a better way of managing their cloud finances.

Stay up to date with more cloud native information. Join my mailing list!.

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
