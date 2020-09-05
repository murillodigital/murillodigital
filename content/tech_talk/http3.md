---
title: "HTTP/3 and the QUIC protocol"
date: "2020-09-04"
description: "A protocol for an insatiable internet"
author: "Leonardo Murillo"
featured_image: "/images/http3_hero.jpg"
images:
- "/images/http3_preview.png"
---
# An insatiable internet

Five years ago in 2015, HTTP/2 was officially launched. Based on earlier developments by Google and their SPDY protocol, it looked to solve known issues present in the previous HTTP/1.1 standard as well as enable lower latency, data compression, and other features necessary for an internet growing ever larger and hungrier for speed and data.

Fast forward to today, and the scale of the internet has outgrown all expectations, and so have the needs of users relying on technologies built on top of the HTTP protocol, which represent a large percentage of traffic on the internet.

A growing availability of broadband as well as mobile 5G technology contribute to an ever-increasing thirst for speed and instant data availability.

# A protocol for the next generation internet

HTTP/3 is the answer the internet community has been working on towards this new world of relentless need for speed and data. This new standard is also based on developments pioneered by Google, known as the QUIC protocol.

The new iteration of the HTTP standard looks to both solve known issues in the HTTP/2 version as well as to take advantage of the increasing amount of bandwidth available.

The protocol is considerably more efficient in handling parallel requests, and by leveraging UDP as opposed to TCP, it is not constrained by connections nor does it suffer trom the latency and overhead of TCP's handshake.

# Technical concepts

HTTP/3 uses QUIC as its underlying transport protocol - this is not a trivial change. 

To understand the dimension of the change, it is relevant to notice that QUIC is not an application layer protocol, it is a transport protocol, on top of which HTTP sits, but it could effectively serve for transporting other application layer protocols.

The use of UDP allows the protocol to multiplex using connection-less streams as well as a dramatically reduce latency, once QUIC has established the initial negotiation, the rest of the packets can be delivered with 0 round trips!

Compare to the SYN, SYN-ACK, ACK three way handshake of TCP and you get a sense of the dramatic reduction in time to first byte.

## Building TCP-like features on top of UDP

You may rightfully wonder how that would work out considering the relatively unreliable nature of the UDP protocol?

The QUIC protocol implements mechanisms to provide reliability on top of UDP, such as congestion control and loss recovery.

QUIC also implements a type of connection on top of UDP, which is used to uniquely define and identify a conversation between two endpoints, and it does so by negotiating a unique connection id on its first packet delivery, and then referring to that connection id on every additional packet of the stream.

The use of a unique connection id also enables interesting benefits, such as maintaining a connection regardless of network interface changes. A simple example of what this means to the user is, the user will be able to remain on the same connection, as they switch between WiFi networks or from a mobile to a wifi connection, allowing for uninterrupted downloads.

Another relevant advantage is QUIC's ability to parallelize streams, guaranteeing in order delivery of data over each stream, but not establishing in order delivery dependencies between data streams.

## Security by default

QUIC is secure by definition, TLS 1.3 is required to negotiate a QUIC connection and can not be disabled, which guarantees always-on encryption.

The amount of unencrypted data sent to establish the initial negotiation is minimal.

## How different is HTTP/3 to its predecessor

A lot of the concepts we are familiar with when we think of HTTP remain present in HTTP/3, for example: headers, body, request, response, verbs, cookies, cache, etc.

The primary changes incorporated in the HTTP/3 standard revolve around how to transfer data, not the structure of that data.

# Limitations and Concerns

There are concerns to using UDP as an underlying protocol, which mostly arise from the momentum of the TCP protocol, that being the more widely used transport protocol on the internet, has gotten most of the attention.

Most software is not optimized for UDP traffic, this results in an increased computational cost to processing the traffic. A similar issue happens at the network level, most networks are optimized for TCP traffic, sometimes even blocking UDP traffic altogether except for the few current widespread use cases where UDP is the transport, DNS and VoIP.

It is likely that as UDP becomes more prominent in terms of percentage of total network traffic, these patterns will change, however until then any implementation should have a graceful fallback to TCP.

# Adoption and support

The adoption of the HTTP/3 protocol is still early, but growing. Some estimates count around 300,000 sites already serving their content using http/3. In the big scheme of things of course, albeit a growing number, it still remains small.

In terms of system support, on the client side the latest versions of Chrome, Firefox and curl support the protocol, mostly experimentally and behind a feature flag.

On the server side, the latest version of nginx supports the protocol; some progress is taking place in terms of IIS adoption of HTTP/3 however very much on early stages, and Apache has not yet expressed any commitment in supporting the protocol yet.

Managed cloud service support for http/3 is minimal, with one notable exception, CloudFlare is spearheading HTTP/3 and has enabled the protocol across their entire network.

# What should a company or engineer do?

It is still early in the game for HTTP/3 adoption, currently I see little value to most organizations dedicating efforts and resources in preparing their systems towards supporting this new protocol, but that will likely change sooner than later.

As engineers we should definitely remain aware and informed of the advances in terms of technology and adoption of HTTP/3 and QUIC, understand the eventual impact it may have on our systems and infrastructure, and become familiar and comfortable with the technology.

Daniel Stenberg, lead developer of the famous `curl` wrote a [small online book](https://http3-explained.haxx.se/en) I strongly suggest reading as a next step for anybody looking to dig a little deeper into the protocol, as we have really just scratched the surface in this article. For those wanting to go really in the weeds, you can find the various IETF specs involved in the protocol [here](https://http3-explained.haxx.se/en/specs).

Hope you learned something new today and I do encourage you do a little bit of playing around and see the protocol in action, CloudFlare has a [quick tutorial](https://blog.cloudflare.com/how-to-test-http-3-and-quic-with-firefox-nightly/) on how to test and see QUIC for yourself.

To be notified of new content please, join my mailing list!.

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


