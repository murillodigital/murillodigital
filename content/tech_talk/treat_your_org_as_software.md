---
title: "Managing your organization as software"
description: "Applying lessons from software engineering to the iterative craft of evolving an organization"
featured_image: "/images/treat_your_org_as_software.jpg"
date: "2020-11-23"
author: "Leonardo Murillo"
images:
- "/images/treat_your_org_as_software.jpg"
---
# Thinking in terms of systems and cycles

Building and evolving a technical organization has components of both science and art. This description is not unusual when talking about people, and even more so when talking about systems made up of humans.

Humans are quite complex, and none of us came with an instruction manual. The dynamics of human interaction happen at many levels, and to drive people in harmony towards a common objective requires tremendous effort and a bit of magic.

Our ability to become effective in this task is literally the definition of success or failure, of the life or death of the organization.

Interestingly, we have learned a lot from systems of interconnected and highly complex components, it's what we do every day as we build complex distributed systems.

Yet, a lot of the strategies we have identified to build and operate highly complex systems of creatively built components that must hit the right balance in terms of independence and interdependence, have not quite transferred over to the organizations in which those systems of software engineers are embedded.

In this post I will elaborate on analogies inspired by the great wealth of knowledge that has been derived from software engineering, and apply them to the organizational structures and processes.

# Decoupled, well documented and with clear interfaces

There are some very important principles to well-architected distributed and complex systems. One of them is the concept of **decoupling**, meaning, allowing each part of the distributed whole to work and evolve independently of one another, while remaining compatible with the functionality of the global solution.

One of the more relevant strategies to enable this type of structure is to define (and document) clear interfaces for how to handle the input and output of each unit, of each service.

And there exist clear and established patterns as to how these interfaces should be built, depending on the nature of the work handled by each unit (think REST or gRPC).

*How does this apply to an organization?*

There are usually hidden dependencies and unclear interfaces across groups in an organization. I see this happen a lot in terms of decision-making, and across business units that are not necessarily technical in nature.

For example, the way you engage with talent acquisition is different as to how you engage with finances, and sometimes engaging both groups will end up in a common decision maker upstream who creates both a bottleneck and hidden coupling.

The way to solve this is, truly understand and document the scope of responsibility of each group, and release decision-making entirely to remain within the bounds of the group for the scope of that responsibility. If there are dependencies, inject them. In other words, preemptively provide to the team making the decision with the necessary data they may need from other units to make a decision as part of the process.

Build common patterns and means of communication across groups in order to satisfy the objective of clearly defined and common interfaces, and make sure there's well-defined input parameters so that each group does have enough information to make decisions and provide service to others with each request.

# Observable with a clear expectation of performance

Observability is a recent buzzword in the software engineering world, and it has grown in popularity for a reason.

Understanding behavior and evaluating performance when systems span service and environment boundaries is a much more complex endeavor in comparison to was required in understanding the operation of a monolithic system.

The relevance of observability goes far beyond simply "monitoring", and rather becomes the foundation for continuous improvement, and the means to validate whether real value is derived from effort, which is what we're all about, work for the sake of work makes little sense.

Organizations are distributed systems by nature, they have been distributed systems long before software followed suit.

What I find is that many times there is no clearly observable way to understand how groups usually interact with one another, nor clearly defined metrics that can be used to work towards a target level of performance.

Methodologies exist to provide snapshots of visibility, usually preceded by a lot of due diligence, for instance "value stream mapping", where inspection and analysis of processes happens after the fact to gauge the efficacy of a process.

The thing about building observability into your organization is, you want to be able to provide constant feedback and real time metrics as to the expected performance of your different processes and objectives.

This is very much aligned with having clear scope for groups and interfaces of communication, the better the structure and data those human systems can deliver, the easier it will be for you to understand the performance of the interactions.

# Short cycles, bite sized objectives

Organizational change across non-technical business units has a tendency to follow waterfall patterns.

I see sales business units coming up with complex new sales qualification processes, human resource groups launching new team on-boarding run books, marketing teams creating quarterly or even longer marketing plans and budgets.

And all these initiatives are usually the result of months of planning and forecasting, and delivered in bulk as a massive change, with the expectation to see results in an eventual future.

What we have learned in software engineering is that:

1) _Forecasting is by definition inaccurate, and the longer your forecast the worst your precision_. We have a lot of difficulty dealing with randomness and there's a lot more random events out there than we care to admit, and as they say hindsight is 20/20, we have a very strong need to find causality to results in the present and this biases our understanding of the actual nature of events.

2) _Big changes are big risk_. Trying to implement a brand new complex process in a bulk move will make it harder for it to be absorbed, and for the company to adapt to the change, not to mention harder to identify, if things aren't looking as expected, what exactly in the process is a likely culprit for the under-performance.

_What is the takeaway engineering can give the rest of the organization?_

Revamp processes gradually with small objectives focused on a more immediate horizon. Now, don't get me wrong, this does not mean that long term strategy is irrelevant, it of course is, but that long term strategy is a moving target that must be taken for what it is, an imprecise and biased speculation of the future that must be constantly reevaluated.

The closer the timeframe the better the definition, make sure your organizational changes can be delivered iteratively and gradually, and if you have already followed the advice on observability, evaluate the output metrics of each change to have data that validates whether the next step is reinforced or must change.

In terms of long term strategy, don't get boxed into it, be mindful of the fact you are biased by your expectations and the expectations of your current market and customers, both internal and external, and reasonable strategies have many times been the cause of established organizations missing out on opportunities because they did not allow themselves to change quickly enough as new data became clearer as the blurry future sharpened into the present.

# Progressive delivery and a clearly defined roll back

Another big problem with big changes is, those who built the process or championed the change get overly attached.

The concept of progressive delivery and rollback in engineering is built on the knowledge that things may fail, that once something is let loose in the wild our own assumptions and strategy may prove faulty, and that there is nothing wrong with that, and just as we must strategize for success we must plan for failure.

A lot of organizational change happens without a clear idea of what happens if this doesn't go as planned.

And going back to a previous state is sometimes impossible, both because of the dimension of the change, and because there was no strategy to revert, and this becomes a costly exercise.

If you are implementing organizational change, allow yourself the opportunity to deliver it progressively and iterate as you gain more knowledge in the process, each cycle will bring more data that will allow you to patch existing "deployments" as well as deliver newer and better versions to the new targets.

And if it didn't go as planned, that's fine, have clarity and communicate ahead of time what metrics will be used to identify the success in the iteration and have a clear concept of how things would go back to their previous state if things don't go as planned.

_Waht does this mean in terms of the non-technical organization?_

Imagine you are going to promote somebody. Define clearly what criteria will be used to determine the change successful, and provide a path for the team member to revert to her previous role should either party determine that the metrics have not been met.

Marketing a new product offering? Again, define how you will determine if traction is moving in the right direction, and don't get attached to it, if it doesn't work out, have a clear path to step out of it without negative connotations.

# Groom, retro and bugfix.

Once we take all of the above into consideration:
 
- Having a long term strategy constantly re-evaluated and delivered in short iterations (allowing for minor and patch versions and not just major upgrades, while still having a roadmap and high level architecture)
- Defining clear responsibilities to groups and give them full autonomy (decoupling)
- Enabling clear communication and well defined collaboration patterns across business units (clear documentation and interfaces)
- Defining metrics that are evaluated in close to real time to organizational and process changes (observability)
- And delivering organizational changes progressively and with a plan for blame-less failure (progressive delivery and rollback)

You will want to continuously evaluate your plan (groom), look back on every iteration and blamelessly identify what went well and what did not (retro and postmortem), and with clear intent move on to the next iteration.

But beware, there **will be bugs**, sometimes you'll be able to hotfix them, sometimes you'll need to refactor, but that is part of the process of continuous improvement, as you move from Organization 1.0 to Organization 2.0!

Stay up to date with new content, join my mailing list!.

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
