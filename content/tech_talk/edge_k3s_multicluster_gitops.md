---
title: "GitOps on the edge with CoreOS, K3s and ArgoCD"
description: "Deploying a consistent state across an edge of Kubernetes clusters"
date: "2020-09-26"
featured_image: "/images/crossplane_oam/oam_hero.jpg"
author: "Leonardo Murillo"
images:
- "/images/crossplane_oam/oam_hero.jpg"
draft: true
---
This is going to be a really awesome experiment, we'll get to play around with lots of cool tech. Because of the complexity of the experiment, I'll split it in n parts:

1. In Part 1 we are going to talk about some general concepts and have a look at the high level, full system architecture.
2. In Part 2 we will set up our central control plane, running ArgoCD on top of a GKE cluster running on GCP.
3. In Part 3 we will build our edge device automation using CoreOS and K3s
4. Finally, in Part 4, we will see the full system in operation

# The scenario
You have selected Kubernetes as your orchestrator of choice for your containerized workloads, you are already using it for all your solutions running in the cloud. But now, you've decided to deploy a network of edge devices, running on hardware with limited capacity, and you want to extend your Kubernetes surface to the edge as well.

You have also discovered the benefits of GitOps, and want to make sure you can easily guarantee a consistent, stable state across your entire fleet of edge devices with the littlest possible effort.

Finally, you are looking for everything to be as automated as possible and be able to get full fleet visibility from a single interface.

So, here's what we will be building.

## High Level Architecture

**Control Plane**: We are going to use a Kubernetes cluster running on GCP as our Control Plane, running ArgoCD to handle the GitOps nature of our implementation.

**The Edge**: Since we're using resource constrained devices on the edge, we're going to go with very minimalistic and lightweight options, I've chosen Fedora CoreOS and K3s for Linux and Kubernetes distribution to run on these edge devices.

**State**: Considering we're going the GitOps route, I'll be using GitHub to hold the manifested state of our edge environment.

# Some concepts first

We're talking about a set of leading technologies and concepts that we'll be exploring in this experiment, let's take a few minutes to make sure we're all on the same page.

## GitOps

The GitOps model aims to guarantee that the full state of a system is version controlled and represented as code.

This model enables a new perspective to continuous delivery - before GitOps, CD was usually driven by pipelines that provided instructions to a target system so it reaches a new state, GitOps takes a different approach, where the state of the system, not the instructions to reach it, are effectively declared in code.

Another important component of the GitOps mindset it, considering we have a full declaration of the system state, we can constantly monitor for divergence and revert the system to the expected stable state should any drift occur.

The nature of GitOps makes it a very developer centric model, after all it is all about "everything as code" and uses the usual lifecycle for code integration most developers are already used to (eg. Pull Requests, branching, tagging, versioning, etc) and, being fully declarative in nature, it is a model very much in alignment to the way Kubernetes and cloud native solutions work.

This article is not _about_ GitOps, so for those looking to learn more, [here's a link](https://www.weave.works/blog/gitops-operations-by-pull-request) for you to follow up.

# K3s: Kubernetes for the Edge

Kubernetes has become the de facto platform for container orchestration, however, a lot of the more common distributions have some hefty requirements as a baseline. In the past, it was complicated using Kubernetes in environments where compute capacity is limited, such as local development environments and of course, the edge.

[Kind](https://kind.sigs.k8s.io/) and [minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/) have solved the local development problem, but they're not quite ideal for running any sort of actual production workload.

K3s comes to fill that gap, by providing a solution that is certified and production ready, but very lightweight and built for running on the edge and on internet connected devices with limited compute capacity. For more details on K3s visit their [official website](https://k3s.io/).

Some of the cool features I'd like to point out about K3s that makes it such a friendly alternative for edge use cases:

- K3s does away with etcd, and uses the extremely lightweight sqlite DB for persistence.
- K3s can run on top of ARM architecture without any additional effort.

Being able to run Kubernetes on the edge allows us many benefits, for example, extending our service mesh and control plane everywhere we run workloads.

# ArgoCD - Continuous Deployment of a declared state

ArgoCD is an awesome continuous delivery tool and control plane focused exclusively on the GitOps model. It is an extremely powerful system that allows you to deploy to any number of clusters from a centralized platform, supporting helm, kustomize and many other config management and templating tools, making it extremely flexible.

It not only syncs your state to a target cluster but also monitors that state to identify drift and revert any changes that deviate from the desired cluster state.

I think there is no other tool in the market so much built for GitOps as ArgoCD. To learn more, and I guarantee you there's a lot to learn about ArgoCD you can check out [its official documentation](https://argoproj.github.io/argo-cd/). Argo is an incubating project under the Cloud Native Computing Foundation.

# kubeprod - The Bitnami Kubernetes Production Runtime

There's a lot of work that needs to be done in getting the most fundamental pieces inside a barebones Kubernetes cluster running.

Getting an ingress controller, observability tooling, tls issuers and authentication in place takes a lot of piping and groundwork. This is the beauty of [bkpr](https://kubeprod.io/). With the Bitnami Kubernetes Production runtime installer you get nginx for ingress controller, a full EFK stack, cert-manager, external-dns and auth-proxy wired for you and ready to go.

The caveat is, you get an opinionated install that abstracts away some level of control saving you huge amounts of time and effort to begin with.

# The architecture

In this experiment we'll set up a central control plane running ArgoCD in a public cloud, we'll be running ArgoCD in a GKE cluster on GCP.

We want the lightest possible solution to run on the edge, and we want to be able to automatically bootstrap an edge device with K3s, part of the automation is to automatically register the cluster aganst ArgoCD once the system becomes operational.

My distribution of choice for the edge device is Fedora CoreOS, due to its minimalistic and container centric design.

Due to the fact I don't have a Raspberry Pi laying around, I will simulate an edge device simply by running a VM inside my Virtual Box. I will most likely update this article in the future once I get me a couple of Pis to play around with, there are some caveats to getting CoreOS running in a Raspberry, Robert Grimm has put together a [great set of instructions](https://fwmotion.com/blog/operating-systems/2020-09-09-coreos-on-raspi4/) on how to get that done.

# PART 2
# Setting up our central control plane

We need to start with our centralized control plane, we will run this in GCP in a GKE cluster.

We're going to use the handy terraform modules provided by Google to create the VPC and subnets as well as a GKE cluster to work as our control plane

Run start.sh to enable APIs

Terraform init and apply to get the cluster in place

The amount of effort that usually goes into setting up the minimum expected services in a K8s cluster is non-trivial, gladly, there's a tool by bitnami called the kubernetes production runtime, which is a relatively opinionated way to get a lot of what you will always need in a kubernetes production environment running in a public cloud in place.



Next we'll install ArgoCD using Helm. Thanks to kubeprod we already have a lot of piping in place so we can quickly expose it via a https secured ingress. Note that to deploy ArgoCD inside a GKE cluster an additional step is required to add a clusterrolebinding.

```bash
kubectl create clusterrolebinding cluster-admin-binding   --clusterrole=cluster-admin   --user=${GCLOUD_USER}
```

Now that we have our control plane running on GKE, lets look getting our edge device running on part 3.

# PART 3


```bash
/murillodigital/experiments/k3s_iot/ignition$ podman run --privileged --pull=always --rm -v .:/data -w /data quay.io/coreos/coreos-installer:release download -f iso

/murillodigital/experiments/k3s_iot/ignition$ podman run -i --rm quay.io/coreos/fcct:release --pretty --strict < k3s-autoinstall.yaml > k3s-autoinstall.ign

#
# You will need to take the contents of the k3s-autoinstall.ign file generated 
# in the previous command, and paste those contents inline in the coreos-autoinstall.yaml
# file before runnning the following command
#

/murillodigital/experiments/k3s_iot/ignition$ podman run -i --rm quay.io/coreos/fcct:release --pretty --strict < coreos-autoinstall.yaml > coreos-autoinstall.ign

/murillodigital/experiments/k3s_iot/ignition$ podman run --privileged --pull=always --rm -v .:/data -w /data     quay.io/coreos/coreos-installer:release iso ignition embed -i coreos-autoinstall.ign ./fedora-coreos.modified2.iso
```

# Prerequisites

1. In order to install CoreOS you will need `podman` running on an existing machine, in order to create the iso image we will use to bootstrap our edge devices. `podman` installation instructions can be [found here](https://podman.io/getting-started/installation)

2. 

# Step 1: Setup your CoreOS image for automated install

As you all know, I'm all for automation, so we will make sure the full process of installing CoreOS and get K3s running inside the machine is automated. This process is what we'll want to run on any device that we want to automatically connect to our central ArgoCD management control plane.

CoreOS uses [Ignition](https://github.com/coreos/ignition) to automate any install tasks on both RHEL and Fedora's distros. Ignition differs from, say `cloudinit`, in that it manipulates disks at a very early stage of the boot process, which means you can do things such as partition and format disks in an automated fashion (really cool!).

When creating ignition files, the simplest way is to create a human friendly FCC (Fedora CoreOS Configuration) yaml manifest, and convert that to an `.ign` Ignition file. `.ign` files are JSON transpilations of your YAML `.fcc`

There are some caveats to consider in order to get this process done:

1) When you boot up the CoreOS ISO's live version, you need to run coreos-installer to actually get CoreOS into the persistent partition of the machine

2) CoreOS is an immutable OS, which means every time you modify the state of the OS, you're actually creating a new version of the system. Along these lines, Fedora CoreOS does not use `rpm` in its usual incarnation, rather `rpm-ostree`, which layers new packages on top of the previous system state, creating a new version. What's the trick? For the new version of your system state to load, you will need to restart so the new version of your OS gets loaded. The reason I point this out is, the default K3s install script requires some RPM prerequisites to be installed, therefore it will not work "as is" and we need to split the process in two: first install the rpm dependencies using `rpm-ostree`, reboot, and then run the k3s install script.



```bash
$ podman run -i --rm quay.io/coreos/fcct:release --pretty --strict < k3s.yaml > k3s.ign
```

Once we have our `k3s.ign` file we'll have to embed it in the installation ISO image. Note that there is an alternative method using PXE, but we will not go through that process in this article. For anyone interested, [this post](https://dustymabe.com/2020/04/04/automating-a-custom-install-of-fedora-coreos/) was of great help as I identified the best mechanism for edge device provisioning automation.

