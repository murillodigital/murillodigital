---
title: "Versioning data with DVC (and a quick DataOps intro)"
description: "Predictable and collaborative handling of data for reliable ML - Part 1"
featured_image: "/images/dvc-hero.jpg"
date: "2021-01-04"
author: "Leonardo Murillo"
images:
- "/images/dvc-thumbnail.png"
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

# Let's dig into DVC

> This is the first of a series where we will dig deeper into DVC, DataOps and ML pipelines. In this instance we'll look at basic version controlling of data.

DVC stands for Data Version Control, but let's get a couple of things straight first:

- DVC does not track versions of data in historically, sequential fashion; in other words, DVC does keep multiple copies of data stored in a backend, but the tracking of the order of changes to that data is managed directly by git.
  
- Data can mean anything, not just data sets - you can effectively use DVC to keep multiple versions of basically anything large enough that you would not squeeze in your Git repo.

Which means, DVC is completely dependent on git, and git alone, it is not compatible with any other version control system.

Now, that's not necessarily a bad thing, particularly if you're already used to Git flow and maybe are even doing other cool things such as GitOps, alas, if your use case requires a difference VCS, you're going to have to look elsewhere.

# Getting our repo and DVC setup

DVC is written in Python, and from my perspective, although [other install methods](https://dvc.org/doc/install) are available, I definitely advise using pip and installing it in a `virtualenv`. This is particularly relevant to start on the right foot in the path of everything reproducible and everything as code.

Let's create a new `virtualenv`, initialize it as a git repo, install DVC in it and initialize it as well. We will go over DVC storage backends a bit later, but here you'll see that we're installing two pip packages, including one for using Google Cloud Storage for backend.

```bash
$ python3 -m venv murillodigital-dvc
$ cd murillodigital-dvc
$ mkdir data
$ git init
$ . bin/activate
$ pip install dvc dvc[gs]
$ pip freeze > requirements.txt
$ dvc init
```

Once you're done with this, in addition to the usual stuff you get with a git repo and a virtualenv, you will find a directory named `.dvc/` and a `.dvcignore` file.

```bash
new file:   .dvc/.gitignore
new file:   .dvc/config
new file:   .dvc/plots/confusion.json
new file:   .dvc/plots/confusion_normalized.json
new file:   .dvc/plots/default.json
new file:   .dvc/plots/linear.json
new file:   .dvc/plots/scatter.json
new file:   .dvc/plots/smooth.json
new file:   .dvcignore
```

With this, you are almost ready to start working with Git and DVC, all you need to add now is a storage backend where DVC will store your actual data. DVC uses a concept very similar to git in the sense of `remotes`. You can have many named storage backends using multiple named remotes.

For this post we are going to crate a single one and set it as default. We installed the Google Cloud Storage backend adapted (the `dvc[gs]` package you can see above), so we will need to create a Cloud Storage Bucket and will use it as remote with the name `google-storage`.

> It's beyond the scope of this post instructions on how to create it, but [here's some documentation to get you going](https://cloud.google.com/storage/docs/creating-buckets) - in the following examples our bucket is called `murillodigital-dvc`.

```bash
$ dvc remote add -d google-storage gs://murillodigital-dvc/data
Setting 'google-storage' as a default remote.
```

Once you've done this, you are all good to go. You will find the remote configuration in `.dvc/config`, very much like remotes in git.

```bash
$ cat .dvc/config
[core]
    remote = google-storage
['remote "google-storage"']
    url = gs://murillodigital-dvc/data
```

# Let's start working with data

We're going to use a public data set for our exercise, and we're going to have two versions of it, a small data set for development use, and the full data set for production use.

I chose the [COVID-19 Open Data set by Google](https://github.com/GoogleCloudPlatform/covid-19-open-data), and we'll use `main.csv` for full production data set (2.5GB in size), and a subset of 4MB for development work.

We'll store the `main.csv` file in the `data/` directory of our repo. We definitely do not want a 2.5GB file in our repository, so lets use DVC to track it and push it to our remote. But before we do that, lets create a smaller version of the file with just the first 15000 lines.

```bash
$ head -n 15000 data/main.csv > data/main-dev.csv
```

Now that we have both our data sets in our `data/` directory, lets get dvc to track them:

```bash
$ cd data
$ dvc add main-dev.csv main.csv
Computing md5 for a large file 'main.csv'. This is only done once.
100% Add|██████████████████████████████████████████████████████████████████████████████████|2/2 [00:23, 11.59s/file]

To track the changes with git, run:

        git add main.csv.dvc .gitignore main-dev.csv.dvc
```

As you can see, DVC has created three files:

- A `.gitignore` to make sure you don't inadvertently commit your data files to the repo
- A `.dvc` file, with the same base name as the data file, which includes the md5 hash of the file and the name that will be used as local reference to that data.

Make sure you run the shown commands to avoid any problems:

```bash
$ git add main.csv.dvc .gitignore main-dev.csv.dvc
```

Let's take a quick look at `main.csv.dvc` to get a better sense of what DVC is tracking:

```bash
$ cat main.csv.dvc
outs:
- md5: 4bce2fda597e2d7de973718953b86d10
  size: 2668297750
  path: main.csv
```

Now, lets push our data to our configured remote. Since we are using Google Cloud Storage as backend, we must have logged into our GCP account with our `gcloud` utility, this process will vary depending on your backend storage provider. For GCP, make sure you run `gcloud auth application-default login` and follow the instructions, after you've done that, its a matter of gcp pus

```bash
$ dvc push
Estimating size of cache in 'gs://murillodigital-dvc/data'                                |0.00 [00:00,     ?file/s]
2 files pushed
```
That's it, we now have our two files stored in our remote storage backend and tracked by Git.

# Team collaboration and data versioning

Now, let's imagine somebody else needs to be involved, somebody who does not have the data and is just getting started with the repo, how does it look for a team member collaborating?

Let's look at the usual collaboration and data versioning workflow. First, we clone the repo:

```bash
$ git clone git@github.com:murillodigital/murillodigital-dvc.git
Cloning into 'murillodigital-dvc'...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 18 (delta 4), reused 18 (delta 4), pack-reused 0
Receiving objects: 100% (18/18), done.
Resolving deltas: 100% (4/4), done.
```

As you can see, the repo is very small, lets initialize it as a virtualenv and get the requirements installed:

```bash
$ python3 -m venv murillodigital-dvc
$ cd murillodigital-dvc
$ . bin/activate
$ pip install -r requirements.txt
```

At this point we have everything we need to work with our data. Now we just need to ask DVC to pull our data from our backend. The configuration of the storage backend as well as the references to the files themselves are already in the repo, so, other than making sure that you do have access to the bucket in which the data lives (by issuing the applicable `gcloud` auth command) you should be good to go:

```bash
$ dvc pull
Estimating size of cache in 'gs://murillodigital-dvc/data'                                |0.00 [00:00,     ?file/s]

 50% Downloading|██████████████████████████████████                                  |1/2 [00:02<00:02,  2.89s/file]
 92%|█████████▏|main.csv                                                      2.28G/2.49G [03:47<00:18,    11.6MB/s]

A       main-dev.csv
A       main.csv
2 files added and 2 files fetched
```

Done, we now have our files fetched from the storage container into our current working copy of our git repository:

```bash
$ ls -lah
total 2.5G
drwxrwxr-x 2 lmurillo lmurillo 4.0K Jan 12 17:52 .
drwxrwxr-x 9 lmurillo lmurillo 4.0K Jan 12 17:39 ..
-rw-rw-r-- 1 lmurillo lmurillo   24 Jan 12 17:34 .gitignore
-rw-r--r-- 1 lmurillo lmurillo 4.0M Jan 12 17:52 main-dev.csv
-rw-rw-r-- 1 lmurillo lmurillo   83 Jan 12 17:34 main-dev.csv.dvc
-rw-r--r-- 1 lmurillo lmurillo 2.5G Jan 12 17:52 main.csv
-rw-rw-r-- 1 lmurillo lmurillo   82 Jan 12 17:34 main.csv.dvc
```

Now, let's consider that the current engineer thinks the development data source is too small, she wants to double it in size and get 30000 records, not just 15000. So she goes and updates the main-dev.csv file to have twice as many rows:

```bash
$ head -n 30000 main.csv > main-dev.csv
$ ls -lah
total 2.5G
drwxrwxr-x 2 lmurillo lmurillo 4.0K Jan 12 17:52 .
drwxrwxr-x 9 lmurillo lmurillo 4.0K Jan 12 17:39 ..
-rw-rw-r-- 1 lmurillo lmurillo   24 Jan 12 17:34 .gitignore
-rw-r--r-- 1 lmurillo lmurillo 8.9M Jan 12 17:56 main-dev.csv
...
```

Notice how the `main-dev.csv` is now 8.9MB as opposed to the previous 4MB version. So, how do we add this new version and get it tracked in the repo and stored:

```bash
$ dvc add main-dev.csv
100% Add|██████████████████████████████████████████████████████████████████████████████████|1/1 [00:00,  1.10file/s]

To track the changes with git, run:
git add main-dev.csv.dvc

$ git add main-dev.csv.dvc
$ dvc push
Estimating size of cache in 'gs://murillodigital-dvc/data'                                |0.00 [00:00,     ?file/s]P
1 file pushed
```

Notice how only one file was pushed, because only one file was changed. Now that we have an updated `main-dev.csv.dvc` file and we have pushed our new data file to our backend storage, we can commit and push this change to git:

```bash
$ git commit -m "Increasing size of dev dataset"
[main a2aac0d] Increasing size of dev dataset
 1 file changed, 2 insertions(+), 2 deletions(-)
 
$ git push
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 554 bytes | 554.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To github.com:murillodigital/murillodigital-dvc.git
   6ee890f..a2aac0d  main -> main
```

Pretty cool right, but we can see the real magic going back and forth in versions, so have a look at what happens when we switch back to the previous version:

```bash
$ ls -lah
total 2.5G
drwxrwxr-x 2 lmurillo lmurillo 4.0K Jan 12 18:18 .
drwxrwxr-x 9 lmurillo lmurillo 4.0K Jan 12 17:39 ..
-rw-rw-r-- 1 lmurillo lmurillo   24 Jan 12 17:34 .gitignore
-rw-r--r-- 1 lmurillo lmurillo 8.9M Jan 12 18:18 main-dev.csv

# Notice how the current main-dev.csv file is 8.9MB, now, lets go back to the previous commit

$ git checkout HEAD@{1}
HEAD is now at 6ee890f Initialize data

$ dvc checkout
M       main-dev.csv

$ ls -lah
total 2.5G
drwxrwxr-x 2 lmurillo lmurillo 4.0K Jan 12 18:27 .
drwxrwxr-x 9 lmurillo lmurillo 4.0K Jan 12 17:39 ..
-rw-rw-r-- 1 lmurillo lmurillo   24 Jan 12 17:34 .gitignore
-rw-r--r-- 1 lmurillo lmurillo 4.0M Jan 12 18:27 main-dev.csv

# And we're back to our previous data set!
```

# Wrap up and next steps

We have seen how to set up a basic DVC development environment, how add and store data sets, and how to create new versions of those data sets!

In our next post in the series we are going to look at integrating DVC into a DataOps pipeline, and working with data in multiple stages. Stay tuned!

To be notified when new posts are published, please join my mailing list.


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
