---
title: "Automating K3S install on CoreOS"
description: "A lightweight Kubernetes deployment for the edge"
date: "2020-10-21"
featured_image: "/images/k3s+coreos.png"
author: "Leonardo Murillo"
images:
- "/images/k3s+coreos.png"
---
Kubernetes keeps expanding its reach, and the edge is no exception.

When thinking edge and IoT, **footprint becomes a critical issue**. You must make sure that whatever you are running to orchestrate and manage your workloads is as lightweight as possible.

Today we'll look at how to automate running a single node Kubernetes _"cluster"_ (I know, single node is not a cluster, but you get the gist) with K3S by Rancher, on top of Fedora CoreOS.

Both K3S and CoreOS are built with minimalism in mind, focused on satisfying the constrained characteristics of edge and IoT environments.

- First we'll look at our [stack of choice](#a-quick-intro-to-our-stack-of-choice)
- Next we'll see [how to automate the installation of K3S on top of Fedora CoreOS](#automating-the-installation)
- Finally, we run the steps to [produce the ISO image](#building-the-iso-image) that will automatically install K3S in our new CoreOS machine.

# A quick intro to our stack

## K3S by Rancher

[K3S](https://k3s.io/) by Rancher is a super lightweight, easy to install Kubernetes distribution meant to be used in edge and IoT devices.

Installation is super fast and everything is packaged in a single binary, including an embedded sqlite database that takes the place of the usual and heavyweight etcd store most other production grade distributions rely on. Note that other persistence backends can be configured if required.

To learn more about K3S have a look at a couple of official resources:

- [K3S Official Documentation](https://rancher.com/docs/k3s/latest/en/)
- [K3S GitHub Repo](https://github.com/rancher/k3s)

## Fedora CoreOS

Designed to run containers, with a minimal footprint and automated updates, Fedora CoreOS is a great choice for edge deployments running containerized workloads.

CoreOS is designed for immutability, meaning you create new versions of the machine to make changes as opposed to modifying the installation in-place.

Fedora CoreOS uses a utility called Ignition to manipulate disks and handle usual bootstrapping tasks when installing the OS, we will use Ignition to automate the installation of K3S.

To learn more about Fedora CoreOS here's a few links:

* [Introducing Fedora CoreOS (good reference from 2019)](https://fedoramagazine.org/introducing-fedora-coreos/)
* [Getting Started with Fedora CoreOS](https://docs.fedoraproject.org/en-US/fedora-coreos/getting-started/)
* [Ignition](https://github.com/coreos/ignition)
* [Fedora CoreOS Config Transpiler](https://github.com/coreos/fcct)

# Automating the Installation

Considering the immutable nature of CoreOS and how Ignition works, we need to do this installation in various stages, to allow the machine to reboot multiple times as we create newer versions of the configuration of the machine.

**The process in general will be as follows:**

1. Install the OS in the machine hard disk, reboot
2. Install the required RPM package for K3S to be installable, reboot
3. Install K3S

We will accomplish this using nested ignition files, and a few systemd services, all baked into a custom CoreOS ISO image that we'll create and use whenever we need to build a new edge device. You'll see what I mean with nested Ignition files shortly.

Note that the code used in this experiment can be found in my GitHub [murillodigital/experiments repo](https://github.com/murillodigital/experiments) inside the `k3s+coreos` directory.

## Our Ignition files

Ignition files are JSON documents that manifest the various configurations and tasks you want the system to perform during boot-up. Ignition has an interesting trait, where it handles these bootstrapping actions at a very early stage of the machine boot process (`initramfs`), thus being able to do all sorts of things including hard disk partitioning.

However, the CoreOS people were nice enough to allow you to write your Ignition files using a more human friendly YAML syntax called FCC (Fedora CoreOS Config) which can be transpiled to Ignition files using a handy tool called [FCCT](https://github.com/coreos/fcct).

We will need two FCC files, and we'll nest a transpiled ignition file into an FCC file, which we'll then transpile into another ignition file - this is what I refer to when I mentioned nested ignition files above.

The resulting, nested `.ign` file is what we will bake into the ISO image that we'll use to start up our CoreOS machines.

**I know this may sound tricky, but bear with me, you'll get it soon, just keep reading.**

First, lets look at the configuration of the ignition file we will be nesting. You can find this file in the experiments repo under `k3s+coreos/ignition/k3s-autoinstall.fcc`.

### Ignition for K3S auto-installation

The full file can be found in the [experiments repo](https://github.com/murillodigital/experiments/tree/master/k3s%2Bcoreos/ignition). Let's look at it part by part.

We will create two systemd services, which will take care of the two separate stages of K3S installation:

- **run-k3s-prereq-installer**: This service will run a script that installs the package necessary for K3S to be installed.
- **run-k3s-installer**: This service will run the K3S install script.

We need to make sure these services only run once, and they run in the required sequence, to accomplish the installation in the order expected.

Pay close attention to the `ConditionPathExists` and `ExecStartPost` stanzas of the services:

The prereq-installer service uses a couple of stanzas to control this behavior:

`ConditionPathExists=!/var/lib/k3s-prereq-installed
ExecStartPost=/usr/bin/touch /var/lib/k3s-prereq-installed`

What this accomplishes is of course that once it's run, the condition for its execution will no longer be true. We need to make sure the machine reboots after installing the prerequisites, and we use another `ExecStartPost` stanza to do just that:

`ExecStartPost=/usr/bin/systemctl --no-block reboot`

The K3S installer service uses a combination of the same stanzas to make sure the service will not run unless the prerequisites are installed and that it will only run the installer once:

`ConditionPathExists=/var/lib/k3s-prereq-installed
ConditionPathExists=!/var/lib/k3s-installed
ExecStartPost=/usr/bin/touch /var/lib/k3s-installed`

Let's see the full `systemd` dictionary in the FCC file:

```yaml
systemd:
  units:
  - name: getty@tty1.service
    dropins:
    - name: autologin-core.conf
      contents: |
        [Service]
        ExecStart=
        ExecStart=-/usr/sbin/agetty --autologin core --noclear %I $TERM
  - name: run-k3s-prereq-installer.service
    enabled: true
    contents: |
      [Unit]
      After=network-online.target
      Wants=network-online.target
      Before=systemd-user-sessions.service
      OnFailure=emergency.target
      OnFailureJobMode=replace-irreversibly
      ConditionPathExists=!/var/lib/k3s-prereq-installed
      [Service]
      RemainAfterExit=yes
      Type=oneshot
      ExecStart=/usr/local/bin/run-k3s-prereq-installer
      ExecStartPost=/usr/bin/touch /var/lib/k3s-prereq-installed
      ExecStartPost=/usr/bin/systemctl --no-block reboot
      StandardOutput=kmsg+console
      StandardError=kmsg+console
      [Install]
      WantedBy=multi-user.target
  - name: run-k3s-installer.service
    enabled: true
    contents: |
      [Unit]
      After=network-online.target
      Wants=network-online.target
      Before=systemd-user-sessions.service
      OnFailure=emergency.target
      OnFailureJobMode=replace-irreversibly
      ConditionPathExists=/var/lib/k3s-prereq-installed
      ConditionPathExists=!/var/lib/k3s-installed
      [Service]
      RemainAfterExit=yes
      Type=oneshot
      ExecStart=/usr/local/bin/run-k3s-installer
      ExecStartPost=/usr/bin/touch /var/lib/k3s-installed
      StandardOutput=kmsg+console
      StandardError=kmsg+console
      [Install]
      WantedBy=multi-user.target
```

As you can see, the systemd services are running scripts inside `/usr/local/bin`, these are scripts that we need to put in place as well, and we will use the `storage.files` array in our FCC document to accomplish that.

There isn't really anything extraordinary about this part of the configuration, we are merely creating a couple of shell scripts in the filesystem so that systemd will have them available to run.

The one aspect that is relevant to pay attention to is that we have split the installation of K3S in two steps because, CoreOS being an immutable operating system, installing new packages effectively creates a new version of the OS and for the package to become available you must boot into that latest version.

As you can see from the script, this is not a usual `rpm install`, rather an `rpm-ostree install` (to learn more about rpm-ostree [check this out](https://github.com/coreos/rpm-ostree)). This is also why the prereq-installer service does a reboot after it's started up.

Have a look at the full `storage` dictionary in our FCC file:

```yaml
storage:
  files:
    - path: /usr/local/bin/run-k3s-prereq-installer
      mode: 0755
      contents:
        inline: |
          #!/usr/bin/env sh
          main() {
            rpm-ostree install https://rpm.rancher.io/k3s-selinux-0.1.1-rc1.el7.noarch.rpm
            return 0
          }
          main
    - path: /usr/local/bin/run-k3s-installer
      mode: 0755
      contents:
        inline: |
          #!/usr/bin/env sh
          main() {
            export K3S_KUBECONFIG_MODE="644"
            export INSTALL_K3S_EXEC=" --no-deploy servicelb --no-deploy traefik"

            curl -sfL https://get.k3s.io | sh -
            return 0
          }
          main
```

Finally, we need to make sure we can ssh into the machine. We will use the `passwd` dictionary to give `core` (the default, `sudo`able user) a public ssh key so we can log into the box. **You will need to change the key of course to one keypair you actually control.**

```yaml
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCerzGGXd3Dg0Y4KDu8eIoGAGk+GdHJGWB6ye4p0AVBbWPbeSDS9yGh9iH3uA44YFhgqdkibHwDg3h/prXN//pcXkOlH53fotglMWY2OefF37gp/Q6FgNZ4sCeGjPU5Wtu2pzN+xbItolRAYmk+ePlezoawnRQdZeQnNjSEvdRE7dppS1WpvJXWx7Uhk5088iZsjrcfMIa2ZKz76jod5CvlDwAb1EkJGmojB5y/DWZI4s//mmgjW/XSSzncxUC7pbrrQ4MLGlz86a0VBzyz3B776p8NFf2yGr3FkVf1zpA00RJ8Oka7whJqXyKkTFmMstQg1ReeFbla68Rlp+TlIsKNx8BWXCDTqEO6jOhRhkhCHRWnGthpZa6YGX67ala4NHLyR7TZJJX95nff0EPd2uB5lr19vjB65+bd/tmkcAK7VT3SMxO9jgngYl4+8g06uUI9OZmpH49bHeoHdmybKeu+AtIaImtFZFnQgXau++Nzq730jenTinR+a5viZ+iP7M8= leonardo@devops.cr
```

### Ignition for CoreOS auto install

The full file can be found in the [experiments repo](https://github.com/murillodigital/experiments/tree/master/k3s%2Bcoreos/ignition).

The CoreOS auto installer FCC file is the one that will be embedded into our CoreOS ISO image, and this FCC file is where we will need to include the contents of the `ign` file we generated from the K3S installer FCC.

**This is the trick, we include the K3S install Ignition file inside the CoreOS Install Ignition file, and we embed the CoreOS Install Ignition file into the ISO image.**

Let's look at CoreOS install FCC file in detail.

First we create a service that runs in the live CoreOS image named `run-coreos-installer`.

It is important to understand that, when you first start up a machine with the CoreOS vanilla ISO Image, you will actually boot right into a live version of CoreOS, and at this point nothing is installed in the hard drive of the machine.
 
 For CoreOS to get installed, you need to run the coreos-installer. That is the purpose of the `run-coreos-installer` systemd service.
 
 It will run the coreos installer so that we get CoreOS actually installed in the machine's storage device and receives the K3S Ignition file as argument for the installer so that once CoreOS is in the hard drive, Ignition will take care of running the K3S install as shown in [the previous section](#ignition-for-k3s-autoinstallation).
 
 > Quick note: The ExecStartPost of the `run-coreos-installer` service you'll see below actually does a poweroff, not a reboot - the reason for that is I needed to be able to unmount the ISO image before the machine rebooted, otherwise the installer would start all over. You may be able to get away with a simple reboot if your default boot device is the hard drive and you manually select the iso image once but it defaults to hard drive on the next boot. You can change `poweroff` to `reboot` in that scenario.

```yaml
variant: fcos
version: 1.0.0
systemd:
  units:
  - name: run-coreos-installer.service
    enabled: true
    contents: |
      [Unit]
      After=network-online.target
      Wants=network-online.target
      Before=systemd-user-sessions.service
      OnFailure=emergency.target
      OnFailureJobMode=replace-irreversibly
      [Service]
      RemainAfterExit=yes
      Type=oneshot
      ExecStart=/usr/local/bin/run-coreos-installer
      ExecStartPost=/usr/bin/systemctl --no-block poweroff
      StandardOutput=kmsg+console
      StandardError=kmsg+console
      [Install]
      WantedBy=multi-user.target
```

Now lets take a look at the script that the `run-coreos-installer` systemd service is running.

In a nutshell, this script picks the device to install CoreOS into, and then runs the CoreOS installer. The installer receives the URL of the image we want it to download and install in the hard drive, as well as the path to the local ignition file we want to run as it installs.

> Note: The URL included for the image to run by the CoreOS installer (as defined in the `image_url` variable in the script below) should reflect the version you want to install - the one shown was the latest at the time of this writing but it may have changed by the time you go through this tutorial.

The ignition file we're passing for the installer to run is our [`k3s-autoinstall`](#ignition-for-k3s-auto-installation) that we looked at before, which you'll need to place in-line inside `coreos-autoinstall` ignition:

```yaml
storage:
  files:
    - path: /usr/local/bin/run-coreos-installer
      mode: 0755
      contents:
        inline: |
          #!/usr/bin/bash
          set -x
          main() {
              ignition_file='/home/core/k3s-autoinstall.ign'
              image_url='https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/32.20201004.3.0/x86_64/fedora-coreos-32.20201004.3.0-metal.x86_64.raw.xz'
              firstboot_args='console=tty0'
              if [ -b /dev/sda ]; then
                  install_device='/dev/sda'
              elif [ -b /dev/nvme0 ]; then
                  install_device='/dev/nvme0'
              else
                  echo "Can't find appropriate device to install to" 1>&2
                  poststatus 'failure'
                  return 1
              fi
              cmd="coreos-installer install --firstboot-args=${firstboot_args}"
              cmd+=" --image-url ${image_url} --ignition=${ignition_file}"
              cmd+=" ${install_device}"
              if $cmd; then
                  echo "Install Succeeded!"
                  return 0
              else
                  echo "Install Failed!"
                  return 1
              fi
          }
          main
    - path: /home/core/k3s-autoinstall.ign
      contents:
        inline: |
          .
          .
          .
```

## Building the ISO image

I hope by now you've gotten a good sense of how the two Ignition files work and the various services we'll use to run our automated installs - if you have not, reach out to me in the comments section at the bottom!

Now we'll put this all together!

You will need to have `podman` installed in your system - [click here to see how to go about getting podman installed](https://podman.io/getting-started/installation), and I recommend doing this on a linux machine, I ran this whole experiment in a Ubuntu VM on top of VirtualBox.

### Step 1 - Download the vanilla CoreOS ISO image

```bash
$ podman run --privileged --pull=always --rm -v .:/data -w /data quay.io/coreos/coreos-installer:release download -f iso
```

### Step 2 - Create an `ign` file from  `k3s-autoinstall.fcc`

Remember to modify the SSH public key in it to match one you control before doing this step!

```bash
$ podman run -i --rm quay.io/coreos/fcct:release --pretty --strict < k3s-autoinstall.fcc > k3s-autoinstall.ign
```

### Step 3 - Insert the contents of `k3s-autoinstall.ign` inside coreos-autoinstall.fcc

This is a manual step. Step 2 results in a `ign` file named `k3s-autoinstall.ign`. You need to take the contents of this file, and place them inside the `coreos-autoinstall.fcc` file inline inside the `storage.files` array, in the object with path `/home/core/k3s-autoinstall.ign`.

Be careful with indentation! It should look something like this (I've removed a lot of the lines to show the general structure, you can see a full example of this file in the [experiments repo here](https://github.com/murillodigital/experiments/blob/master/k3s%2Bcoreos/ignition/coreos-autoinstall.fcc)):

```yaml
storage:
  files:
    - path: /usr/local/bin/run-coreos-installer
      mode: 0755
      contents:
        inline: |
          .
          .
    - path: /home/core/config.ign
      contents:
        inline: |
          {
            "ignition": {
              "version": "3.0.0"
            },
            "passwd": {
              "users": [
                {
                  "name": "core",
                  "sshAuthorizedKeys": [
                    "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCerzGGXd3Dg0Y4KDu8eIoGAGk+GdHJGWB6ye4p0AVBbWPbeSDS9yGh9iH3uA44YFhgqdkibHwDg3h/prXN//pcXkOlH53fotglMWY2OefF37gp/Q6FgNZ4sCeGjPU5Wtu2pzN+xbItolRAYmk+ePlezoawnRQdZeQnNjSEvdRE7dppS1WpvJXWx7Uhk5088iZsjrcfMIa2ZKz76jod5CvlDwAb1EkJGmojB5y/DWZI4s//mmgjW/XSSzncxUC7pbrrQ4MLGlz86a0VBzyz3B776p8NFf2yGr3FkVf1zpA00RJ8Oka7whJqXyKkTFmMstQg1ReeFbla68Rlp+TlIsKNx8BWXCDTqEO6jOhRhkhCHRWnGthpZa6YGX67ala4NHLyR7TZJJX95nff0EPd2uB5lr19vjB65+bd/tmkcAK7VT3SMxO9jgngYl4+8g06uUI9OZmpH49bHeoHdmybKeu+AtIaImtFZFnQgXau++Nzq730jenTinR+a5viZ+iP7M8= leonardo@devops.cr"
                  ]
                }
          .
          .
          .
```
### Step 4 - Create the `coreos-autoinstall.ign` file from the `fcc` file created on step 3.

```bash
$ podman run -i --rm quay.io/coreos/fcct:release --pretty --strict < coreos-autoinstall.fcc > coreos-autoinstall.ign
```

### Step 5 - Embed the `coreos-autoinstall.ign` ignition file inside the ISO image

>  Note: The name of the ISO image may vary - the command on [Step 1](#step-1---download-the-vanilla-coreos-iso-image) will download the latest version, which may be different from the time of the writing of this article. Check the name of the downloaded file from the output of Step 1.

```bash
$ podman run --privileged --pull=always --rm -v .:/data -w /data quay.io/coreos/coreos-installer:release iso ignition embed -i coreos-autoinstall.ign ./fedora-coreos-32.20201004.3.0-live.x86_64.iso
```

# Voilà

And we're done, we now have a custom ISO image with automation baked in so that, booting any machine with it will automatically install CoreOS and K3S without any manual labor!

Once you boot up your first device with this ISO image, you can ssh into it using the key you used on the FCC file for [`k3s-autoinstall`](#ignition-for-k3s-auto-installation).

The `kubeconfig` file to connect to the Kubernetes control plane running in the machine can be found at `/etc/rancher/k3s/`.

Hope you enjoyed this tutorial and get to try this experiment. To keep up to date with experiments like this, subscribe to my mailing list!

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