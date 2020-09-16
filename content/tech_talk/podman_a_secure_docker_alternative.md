---
title: "Podman - Daemonless Containers"
description: "A more reliable and secure alternative to docker"
date: "2020-09-15"
featured_image: "/images/podman.jpg"
author: "Leonardo Murillo"
images:
- "/images/podman.svg"
---
Before we get started on Podman, let's do a quick overview of containers in general.

# The world before and after Docker

For most, hearing the word "containers" immediately brings up Docker to mind. Docker was not the first container initiative in the Linux world, but it did come to breach the gap between containers and developers, and brought the concept to the mainstream.

However, initiatives and technologies to isolate running processes and provide boundaries and resource control inside a linux system exist all the way back to 1979 when chroot was first introduced.

Containers more aligned with the full scope of docker functionality start to appear as early as 2004, and docker finally makes its debut in 2013, revolutionizing the container landscape, more due to the ecosystem Docker came to enable than merely the technical strengths of its container library.

# Open Container Initiative

As the container industry grew, it became evident some standard would be ideal towards enabling a level on interoperability across the various actors in the ecosystem.

In 2015, Docker and other industry leaders established the Open Container Initiative (OCI) to outline some standards as to how to store, unpack and run a container image.

# Some alternatives to Docker

As I've mentioned, docker is not the only option out there to run containers. There are alternatives that take somewhat different approaches towards running workloads inside containers.

We will be looking into `podman` today but let's briefly mention other options:

- **rkt**: Formerly CoreOS Rocket, rkt is one of the more widely accepted alternatives to Docker. It has a growing ecosystem and focuses on security and interoperability, and similar to podman, implements a daemon-less approach to containers.

- **LXC**: A technology sponsored by Canonical and part of LinuxContainers.org, LXC's approach also follows a daemon-less approach. One of the more obvious differences between LXC and Docker is that it's designed to support multiple processes running inside a container, as opposed to Docker where the idea is to run a single process per running container.

# Now lets look into Podman

Podman is open source ([hosted in GitHub](https://github.com/containers/podman)) and, while providing a docker-compatible interface, provides one fundamental benefit: it works without a daemon and forks the container processes as its own children.

Let's talk a little bit in more detail about these two characteristics.

## A docker compatible interface

Podman is written in a way that its command line interface is identical to that of docker, which means you can effectively alias podman as docker and, if you are familiar with docker's command line interface, do the same things you did in exactly the same way you did them without any impact.

All the commands you are familiar with, remain the same:

```bash
# Build and image
podman build -t some_tag:latest ./

# Run an interactive command inside some running container
podman exec -ti ab12 /bin/bash

# See a list of all locally stored images
podman images -a
```

You could effectively `alias podman=docker` and replace docker with podman in a completely transparent way.

## Daemonless operation

This is one of the more meaningful differences between podman and docker. Docker runs as a system service, as a child of init. Init is the first process that gets started in a linux system, and from which all other processes spawn. In most current linux distributions, SystemD is the init process, and every service it runs becomes one of its children.

Docker runs as a privileged service (in its default, and most functional install), it is run by init as root and has wide open access to the system.

Since docker runs as a service, the way users interact with it is by making requests to its API. The issue with this model is that it bypasses a lot of linux's security into its own means of authentication: if you can talk to docker, you effectively now have full access to the underlying system, and if you perform privileged actions in the host via a container, you can completely hide the source of the operation.

The other problem of the daemon model is that it becomes a single point of failure, one process owns all your containers.

Enter podman and it's daemon-less approach to running containers. A few of the more meaningful characteristics of this approach are:

### Rootless

Podman is fully functional when used by a non-root user, and does not require privileged access (except for binding to privileged sockets). Also, by running in user space, you can take full advantage of user namespaces and isolate containers run by different users in the system.

Podman will map root inside a container to your UID, meaning all your containers will be owned by your user in the host process list eventhough the container is running "root" internally.

### User specific configuration paths

Podman will use directories specific for each user running container separate from one another, storage and config configuration files are separate and provide individual customization to the operation of podman.

## Pods are the new thing - thank you K8s

Another important difference between Docker and Podman is that podman (as its name duly implies) can natively work with Pods.

Pods are multi container objects that share resources and must always be scheduled together and located together. Networking, storage and compute resources are available to all containers running inside it.

# A new root-less ecosystem

Podman is part of a growing toolchain focused on enabling containerization without the downsides of root access requirements.

Among the various participants in this evolving ecosystem, you'll find new tools for building and managing container images in user space:

* [Buildah](https://buildah.io/): Allows you to build OCI container images without root access and without a daemon.
* [Skopeo](https://github.com/containers/skopeo): Allows you to copy, inspect, sign and manage container images, also with no daemon nor root access.

# Getting started with Podman

Podman is quite easy to set up and try out. If you want to start experimenting with it (and I think you should), here's their official [getting started guide](https://podman.io/getting-started/), if you want to dig deeper into what podman can do, [the full documentation](https://podman.readthedocs.io/en/latest/index.html) is the place to go.

# Share your experience

As you explore these new tools, please share your learning and insights!

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
 