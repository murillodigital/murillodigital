---
title: "Versioning data with DVC (and a quick DataOps intro)"
description: "Predictable and collaborative handling of data for reliable ML"
featured_image: "/images/dvc-hero.jpg"
date: "2021-01-04"
author: "Leonardo Murillo"
images:
- "/images/dvc-thumbnail.png"
draft: true
---
# The struggle with Data

Don't let anybody fool you, data is tough. It's difficult to move around, and the more voluminous it gets, the harder it becomes to make sure it's clean and reliable.

Trying to solve data consistency and quality issues always feels to me like finding needles in haystacks, _except it's not haystacks, it's 50-ton balls of mud_.

It gets even more complicated when you incorporate ML into the mix. You know the saying right, [GIGO](https://en.wikipedia.org/wiki/Garbage_in,_garbage_out), well, there's no other scenario that from my perspective better embodies this concept.

Models depend on clean, reliable, solid data, and if you feed it poor data you're in trouble, because the model usually will not be explicit in that the data is poor, it will simply produce hard to identify crappy results.

# DataOps to the rescue

DevOps taught us a lot of great things to keep our systems and the lifcycle of our software reliable and predictable.

With DevOps, we learned about CI/CD pipelines and tests and all sorts of automation, we explored idempotence and reproducibility, and fell in love with the mindset of everything as code. Plus, we learned to collaborate, to participate collectively in the full lifecycle of our products from development to production.

![DevOps Everywhere](/images/devops-everywhere.webp)

### What if those very lessons could be used to improve the experience of working with data?

**Well, that is precisely what DataOps looks to accomplish!**

Now, this post is not about DataOps, but I do think is an important introduction because, for DataOps to be possible, a new generation of tooling must be considered.

When we think in terms of reproducibility, orchestration, and the concept of _everything as code_, fit for purpose tools would be very important for success. With this, we can segway into the real subject of this post, one of those very tools: [DVC](https://dvc.org/)

# Let's dig in - DVC

DVC stands for Data Version Control, but let's get a couple of things straight first:

- DVC does not track versions of data in historically, sequential fashion; in other words, DVC does keep multiple copies of data stored in a backend, but the tracking of the order of changes to that data is managed directly by git.
  
- Data can mean anything, not just data sets - you can effectively use DVC to keep multiple versions of basically anything large enough that you would not squeeze in your Git repo.

Which means, DVC is completely dependent on git, and git alone, it is not compatible with any other version control system.

Now, that's not necessarily a bad thing, particularly if you're already used to Git flow and maybe are even doing other cool things such as GitOps, alas, if your use case requires a difference VCS, you're going to have to look elsewhere.

# Getting our repo and dvc setup

DVC is written in Python, and from my perspective, although [other install methods](https://dvc.org/doc/install) are available, I definitely advise using pip and installing it in a `virtualenv`. This is particularly relevant to start on the right foot in the path of everything reproducible and everything as code.

1. Create a virtualenv and install dvc in there
2. Make the virtualenv a git repository, and add a directory to store our data, and a .keep file in there


## The architecture

DVC uses a remote storage backend to store the actual data, and uses git to track versioned metadata pointing to specific copies of the data.  

## Installation

DVC is a Python package so I would definitely suggest 

## Getting a sample data set

## Modifying the data set and creating a new version

## Caveats

dvc config going into the repo exposes credentials to a repository

