---
title: "Confidential Computing"
date: "2020-08-16"
description: "Securing data's last frontier, and further enabling edge computing and the cloud"
featured_image: '/images/confidential-computing.jpg'
---
If you haven't heard the buzz about *confidential computing*, I'm sure you will very soon. I hope with this post you will:

- Get a [bit of background](#what-is-confidential-computing) of what confidential computing is, and how we got here.
- Understand some [technical concepts](#technical-concepts) you will need to be aware of, to understand confidential computing.
- Get an overview of the current features available across the [major public clouds](#public-cloud-service-comparison) supporting confidential computing.

# What is confidential computing
Data moves through three different states:

1. In storage: also known as **data at rest**
2. Moving from one place to another over some connecting media: **data in transit**
3. Loaded into memory and actively used for computations: **data in use**.

Encryption as a means to protect data is widely used when we look at two out of those three states: **we currently use encryption to store our data, and encryption to transmit it**.

> The use of HTTPS to securely deal with web based traffic over the internet (aka. data in transit) went from around 50% in 2015, to over 95% in 2020.

However, there's one glaring deficiency to all this security. As soon as the data is loaded up into memory, all the benefits from encryption become moot, as soon as we start *using* our data, it is exposed.

### This huge vulnerability is the last frontier in data security and the aim of confidential computing: _to guarantee that data and application code can be trusted and are only accessible by the people and systems authorized to do so during runtime._

It remains as the last frontier of security for a clear reason, **it is a complicated problem** that requires hardware level support and used to require important changes to source code and code delivery.

The use of trusted execution environments, which are secure areas in a processor that guarantee data confidentiality and integrity, is one of the key technologies under development to overcome this deficiency. **These challenges are precisely the focus of current confidential computing initiatives.**

# Why is confidential computing a hot topic?

This will not be a surprise: **the internet is a hostile environment**.

The amount and complexity of attacks demonstrates a dangerous uptrend, fueled by the also exponentially increasing amount of data held in digital systems _everywhere_.

The meaning of _everywhere_ in terms of data and compute is far-reaching: on enterprise premises, in the cloud, in users homes, pockets and on the edge. There is virtually no place where there is no data being held **and actively processed**

## Increasing Regulation

Many might say that the industry alone has not done enough to protect customer's data by their own initiative. Usually when that pattern emerges, **governments step in to do a little bit of convincing**.

**GDPR** (General Data Protection Regulation) in the European Union, **CCPA** (California Consumer Privacy Act) in the USA, **DPC** (Data Protection Committee) in India and **DPA** (Data Protection Act) in the UK are just a few of the regulations that have been born out of this urgent need to hold companies accountable for their responsibility in holding the data of millions of people.

## Numerous and advanced recent attacks

From data theft to ransonware, a month doesn't go by without some company showing up in the headlines, announcing they've exposed confidential data from millions of people to unauthorized parties. **Cyber threats are now handled by powerful, organized groups - including some governments eager to manipulate the decisions of entire countries.**

- Marriot
- Cognizant
- Nintendo
- Adobe
- Equifax
- LinkedIn
- Zynga

The list could go on, exposing hundreds of millions of users to cyber criminals.

## The cloud and a data-and-software-is-everywhere world

A driving factor to this growing number of attacks is of course the related growth in the amount of data that is captured and held by companies related to users, and this growth is enabled in large part by big data and the cloud.

The cost per gigabyte of storage has raced from half a million in 1981 to under $0.03 today, bandwidth availability is growing at a similar rate, and the clouds have enabled cost-effective mechanisms to reach virtually infinite scale, to companies of any size.

Add to that the massive amount of smart devices in everybody's homes and pockets, and you end up with the conditions for a perfect storm: **cheap storage and big internet pipes, readily access to user's every move, and companies of all sizes with access to infinite compute resources eager to find ways to derive value and profit from the combination**.

# The Confidential Computing Consortium

The concern for data security is present in the minds of technology companies large and small. It is a blatant Achille's heel that spares no one - all companies currently dealing with technology, software and data are all equally exposed.

Hence, massive industry interest is adding up, and many [big names in technology](https://confidentialcomputing.io/members/) are looking to actively participate both for protection as well as to influence the decisions that will define how this next frontier of data security is addressed.

Gladly, The Linux Foundation has stepped in to act as governing body of these discussions and aims to guarantee a level of openness and community inclusion. 

> The concern for the security of our data should not matter to businesses alone - it is your data as well as my own which is held by corporations, and it is our duty as technologists to be involved in the conversation.

Hence the creation of the **[Confidential Computing Consortium](https://confidentialcomputing.io)**, an initiative with a goal to _Define and enable confidential computing, and accelerate its acceptance and adoption in the market_.

# Technical Concepts

It is important to grasp a few technical concepts to understand, at a high level, the requirements and implications of confidential computing.

## Trusted Execution Environments (TEE)

A key piece of confidential computing technology is the concept of **trusted execution environment**.

`TEE`s use hardware specific support to provide application code and data isolation during runtime with higher levels of security than other previous alternatives (such as TPM - the Trusted Platform Module developed by the Trusted Computing Group).

Implementations exist for many current platforms: **Intel Software Guard Extension** (SGX) being one of the more widely available implementations, and **AMD Platform Security Processor** (PSP) and **ARM TrustZone** are close competitors.

#### The Trusted Execution Environment runs parallel to the untrusted operating system, and uses a private key embedded in the hardware to prevent hardware simulation attacks.

`TEE`s offer two guarantees for the code executed and data used inside of them: **confidentiality and integrity, which means no one can access the data outside the environment, and the code that the environment is running using that data cannot be modified.**

## Enclaves and SDKs

A critical barrier to adoption is the complexity associated in taking advantage of the various platform features that enable isolated execution. **SDKs are indispensable in the ecosystem to fuel widespread adoption.**

**SDks provide developer friendly ways to run code inside enclaves, which take advantage of trusted execution environments functionality.**

Multiple projects exist, aiming at different perspectives to the same objective, enabling non-cryptographic-experts nor low-level code writers to take advantage of the features associated to confidential computing.

Interesting alternatives include:

- [Scone](https://scontain.com/)
- [Graphene](https://github.com/oscarlab/graphene), yet to be production ready
- [Google Asylo](https://asylo.dev/)
- [Microsoft Open Enclave SDK](https://openenclave.io/sdk/)

# Public cloud service comparison

Many organizations consider security and confidentiality a barrier to cloud adoption, and worry about cloud providers having access to or exposing
their confidential data and intellectual property.

**The rise of confidential computing in the cloud will enable many corporations that are currently missing out on the myriad of benefits that the cloud provides, without risk.**

## IBM Cloud - Data Shield
I'm particularly curious about IBM Cloud's Data Shield, considering its native integration with Kubernetes. The fact it is delivered as a Helm Chart and is easily integrated with existing DevOps toolchains is incredibly appealing to me.

Data Shield uses Intel SGX, and extends the platform's support to Python and Java.
 
I will be doing one of my technical experiments on IBM Cloud's Data Shield soon - join my mailing list and stay tuned, look for the signup form at the end of this article.

## Google Cloud Platform - Confidential VMs
Currently in Beta, Confidential VMs provide a seamless path to running isolated workloads, protected by AMD EPYC CPUs using its Secure Encrypted Virtualization Extension), you can run anything you are currently on your existing VM in a confidential VM without any code level changes.

This is a great option for anybody running workloads in virtual machines looking for the benefits of confidential computing, however, I'm curious as to the benefits of the service in the world of containerized workloads.

I will be doing one of my technical experiments on GCP's Confidential VMs soon - join my mailing list and stay tuned, look for the signup form at the end of this article.

## Azure Confidential Computing
Now generally available, Azure also offers specific types of VMs enabled for confidential compute leveraging Intel's SGX. This seems to me like a similar approach to that of Google Cloud, except that has reached general availability maturity.

## What about AWS?
I could not find confidential compute service offering from Amazon Web Services which was quite surprising to me, so much so that I wonder if I'm missing out on something.

If you are aware of AWS support for confidential compute, please let me know in the comments.

---

I hope you gained some valuable insights on Confidential Computing by reading this post. As I mentioned above I will be doing some hands on experimentation with the various cloud based confidential computing services, so stay tuned.

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