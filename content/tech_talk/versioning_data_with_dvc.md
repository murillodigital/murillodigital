---
title: "Versioning data with DVC"
description: "Predictable and collaborative handling of data for reliable ML"
featured_image: "/images/dvc-hero.jpg"
date: "2021-01-04"
author: "Leonardo Murillo"
images:
- "/images/dvc-thumbnail.png"
draft: true
---
# The struggle with Data

# ML is as good as the data fed into it

# Following Git patterns with data

# Let's dig in

First, let's get a couple of things straight, DVC does not track versions of data in a sequential context, in other words, DVC does keep multiple copies of data stored in a backend, but the tracking of the order of changes to that data is managed directly by git.and data can mean anything, not just data sets - you can effectively use DVC to keep multiple versions DVC is like git, except for data. Matter of fact it coexists and amply leverages git to get the full version control experience.

## The architecture

DVC uses a remote storage backend to store the actual data, and uses git to track versioned metadata pointing to specific copies of the data.  

## Installation

DVC is a Python package so I would definitely suggest 

## Getting a sample data set

## Modifying the data set and creating a new version

## Caveats

dvc config going into the repo exposes credentials to a repository

