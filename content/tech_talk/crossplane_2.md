---
title: "The Open Application Model and Crossplane - Part 2"
description: "An agnostic way to build application centric platforms and infrastructure in a team centric way, across clouds"
date: "2020-09-07"
featured_image: "/images/crossplane_oam/oam_hero.jpg"
author: "Leonardo Murillo"
images:
- "/images/crossplane_oam/oam_hero.jpg"
draft: true
---
# Let's get our hands dirty

Welcome to Part 2 of my series on the Open Application Model and Crossplane, if you just got here and want to learn what OAP and Crossplane are in the first plane, start with [part 1]({{< ref crossplane >}}) of this series for an overview of both.

In this article you will get to:

1. Deploy OAM and Crossplane to a Kubernetes cluster
2. Deploy a sample solution following the OAM framework

In this experiment we will take Google's "Online Boutique" and transform it into a OAM / Crossplane framework based solution! The original version on the sample solution can be found in [this GitHub repo](https://github.com/GoogleCloudPlatform/microservices-demo)

# Getting setup



Get notified, join my mailing list!.

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
