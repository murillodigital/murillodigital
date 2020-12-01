---
title: "KubeCon North America 2020 Virtual"
description: "Community, kubernetes, cloud technology, and the new normal"
featured_image: "/images/kubecon_2020.jpg"
date: "2020-11-24"
author: "Leonardo Murillo"
images:
- "/images/kubecon_2020.jpg"
---
[Click here](#the-technical) if you want to go straight to the technology insights.

# The non-technical

## The new (virtual) normal
I have to say I love virtual conferences.

Yes, there definitely is a void in terms of interaction and hanging out with friends and community. 

Yet, I must admit, I struggle with engaging people that I have not had some form of contact before in person on hallways during on-site conferences. Reaching out to people in Slack or whichever communication platform chosen by the organizers is dramatically friendlier to a person of my character and allows me to have remarkably better results in finding new contacts and growing my network.

The fact I'm not bound to eat airport and hotel food, that I get to workout as usual every day, and that I don't have to say goodnight to my kids on a 6-inch screen, are definitely more arguments to why I do appreciate virtual conferences, if done right.

That's the keyword, _done right_, something quite not yet solidified and that I've seen accomplished with various degrees of success over the last few months.

## Communication, community and the conference experience

So, let's start there, how was my _experience_?

I think the CNCF had mixed success in their choice of tools for the conference, and the ability of the platform to promote a user-friendly and natural conference experience.

Slack was a reasonable choice for text based communication, and the community found ways to fill in the gaps using external alternatives. I would love to see conference organizers finding means to enable the real time interaction of attendees in more dynamic ways other than just text messaging. 

Nevertheless, the cloud native community really stepped up and demonstrated how truly connected, welcoming and warm they are, they took it upon themselves to find and try out tools to allow attendees to talk and interact. Zoom and Rambly served as unofficial venues for people to hangout and share, and albeit far from the in-person experience, definitely served as approximations to a much more personal experience.

Now, as far as the presentation platform itself, if I were part of the team of organizers, I would strongly advocate for not using Intrado on another event.

I had already used Intrado on another Linux Foundation conference, the Open Source Summit, and I was not satisfied then, and I'm still not satisfied. The platform is totally not-mobile-friendly, and I think it presents zero opportunity for participants to engage one another. Add to that the recurring technical problems that surfaced at scale and that basically sums up as deal breaker for me.

As a point of comparison, the platform used by Gremlin for their past ChaosConf was much better, as soon as I find out what that service was, I'll update this post. 

# The Technical

Ok now let's talk about what I think most of you came here for, the technology!

## Chaos Engineering is gaining traction

The number of projects out there promoting the practice of Chaos Engineering is growing and solidifying. I see a strong trend in the Chaos Engineering practice becoming a lot more present across organizations and integrated into SRE teams or otherwise applied, particularly as the complexity of cloud native architectures grows, and the surface of mission critical workloads also increases on top of Kubernetes.

Two projects were presented during the conference and definitely deserve being in technology leader radars.

* [Chaos Mesh](https://litmuschaos.io/): Open source and available at [github.com/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh)
* [Litmus Chaos](https://litmuschaos.io/): Built by MayaData, and also open source. The source code can be found in [github.com/litmuschaos](https://github.com/litmuschaos/litmus)

## Service Mesh and Multicluster/Multicloud

[LinkerD](https://linkerd.io/) with version 2 demonstrates a very interesting alternative for those looking to implement service mesh on their clusters, particularly due to the simplicity of its implementation and the quick access it gives you to deploying workloads to multiple clusters (and thus multiple clouds). During the conference a great demo was shown using LinkerD and Ambassador for deploying multi-cluster workloads, I really encourage you to have a look:

Multicluster with LinkerD and Ambassador:
- [Slides](https://github.com/grampelberg/talks/blob/master/kubecon-11-2020/slides.pdf)
- [Tutorial](https://linkerd.io/2/tasks/multicluster/)
- [Deep Dive](https://linkerd.io/2020/02/25/multicluster-kubernetes-with-service-mirroring/)

It's clear that multicluster and multicloud is also becoming a common challenge for companies maturing their Kubernetes architecture and scaling up. 

The talk by **SIG Multicluster** was interesting, and I learned about [KubeFed](https://github.com/kubernetes-sigs/kubefed) which is currently in Alpha. I strongly encourage for anybody that needs to work with multiple clusters to dig into KubeFed, so here's the [user guide](https://github.com/kubernetes-sigs/kubefed/blob/master/docs/userguide.md) and [general concepts](https://github.com/kubernetes-sigs/kubefed/blob/master/docs/concepts.md).

## MLOps and other innovative use cases

DataOps and MLOps are becoming ever more relevant, as more organizations mature in their use of Kubernetes and want to start leveraging the same platform for their data and machine learning operations.

[Kubeflow](https://www.kubeflow.org/) and [KFService](https://github.com/kubeflow/kfserving) are definitely tools relevant to be in the radar of anybody working with data and/or ML looking to leverage Kubernetes.

## DevSecOps and Security in General

Security remains one of the more critical difficulties in working with containers and Kubernetes, something also validated by the results of CNCF's 2020 Survey, where security keeps the 3rd spot in terms of container challenges.

[Falco Project](https://falco.org/) and [Open Policy Agent](https://www.openpolicyagent.org/) surfaced as relevant technologies to keep up to date with.

Static analysis tools for kubernetes manifests, infrastructure as code and other tooling to incorporate in CI/CD pipelines were also worthy of notice, you can find some links to these in my [dump](#a-dump-of-other-technologies-and-tools-to-track) below.

## A dump of other technologies and tools to track

There's just too much to cover for the time I have available to write this post, so I'm just going to dump a whole bunch of links to interesting projects, tools or otherwise that I picked up during the conference that you likely will be interested on as well, in no particular order:

- [Kuma](https://kuma.io/): Control plane for envoy based service mesh.
- [Apache Guacamole](https://guacamole.apache.org/): Clientless remote desktop server, basically a web server for you to access RDP, SSH and VNC hosts.
- [Remediation as Code](https://www.accurics.com/blog/devops/remediation-as-code/): A paradigm to build self healing infrastructure by continuously and automatically evaluating for code level configuration problems on declarations of immutable deployments.
- [TUF](https://theupdateframework.io/): The update framework, a secure framework for update management.
- [Spiffe and Spire](https://spiffe.io/): A universal identity control plane for distributed systems.
- [Kubernetes Finalizers](https://blog.anynines.com/kubernetes-finalizers-in-custom-resources/): Not commonly used but very powerful means to control object deletion.
- [Apache OpenWhisk](https://openwhisk.apache.org/): This is quite mature and has been in most serverless interested radars for a while, but nevertheless.
- [Contour](https://projectcontour.io/): High performance ingress controller.
- [Keptn](https://keptn.sh/): Event based control plane for CD and automated operations. **This one really drew my attention**
- [Dragonfly](https://d7y.io/en-us/): P2P based image and file distribution.
- [KubeEdge](https://kubeedge.io/en/): An open platform for Edge computing. **This one I was also particularly interested in**
- [Project Sonobuoy](https://github.com/vmware-tanzu/sonobuoy): Diagnostic of kubernetes cluster state.
- [cri-o](https://cri-o.io/): Lightweight container runtime for Kubernetes.
- [OpenTelemetry](https://opentelemetry.io/): What happens when you get census and tracing together? You get OpenTelemetry, an observability framework for cloud native software.
- [KubeAcademy](https://kube.academy/): Free education on Kubernetes.
- [KUTTL](https://github.com/kudobuilder/kuttl): A declarative tool to test kubernetes operators.

I may do a Part 2 of this post with more in-depth coverage as well as more of the references I took note during the event, but I will need some encouragements out of the sheer volume of data that Kubecon provided so if you want to see that happen, reach out or comment in this article.

Which is a meaningful closing remark, it was *definitely* worth it, I had a fantastic time, met a lot of people and got a lot of great technology and uses cases in my radar, next time you should join as well.

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