# Ultimate Guide to BOSH

[BOSH](https://bosh.io) is an open source tool for release engineering, deployment, lifecycle management and monitoring of distributed systems.

It's incredible. Huge companies are using it. Tiny companies are using it. You too could be using it.

This is the Ultimate Guide to BOSH.

It will place you in the middle of daily life with BOSH and gradually guide you toward understanding, deploying your own systems, and then through to deep understanding. You'll become a raving fan.

# TOC

   * [Ultimate Guide to BOSH](#ultimate-guide-to-bosh)
   * [TOC](#toc)
   * [Introduction](#introduction)
      * [WIP](#wip)
      * [Guide to the guide](#guide-to-the-guide)
      * [Joyful operations](#joyful-operations)
      * [Brief history of BOSH](#brief-history-of-bosh)
      * [BOSH in production](#bosh-in-production)
      * [Why write the Ultimate Guide to BOSH?](#why-write-the-ultimate-guide-to-bosh)
      * [Additional sources of information](#additional-sources-of-information)
   * [Why BOSH?](#why-bosh)
      * [What is a running software system?](#what-is-a-running-software-system)
      * [Choose your own deployment level](#choose-your-own-deployment-level)
      * [Assumptions](#assumptions)
      * [Continuous Integration and Continuous Delivery](#continuous-integration-and-continuous-delivery)
   * [Deployments](#deployments)
      * [New deployments](#new-deployments)
      * [New deployments of Zookeeper](#new-deployments-of-zookeeper)
      * [BOSH Architecture, Part 1](#bosh-architecture-part-1)
      * [CPI - the ultimate Cloud Provider Interface abstraction](#cpi---the-ultimate-cloud-provider-interface-abstraction)
   * [Instances](#instances)
      * [SSH](#ssh)
      * [Shell user prompts in examples](#shell-user-prompts-in-examples)
      * [Monit process monitoring](#monit-process-monitoring)
      * [Job templates describe processes](#job-templates-describe-processes)
      * [Job templates](#job-templates)
      * [Running processes, summary-in-progress](#running-processes-summary-in-progress)
      * [Logs](#logs)
      * [Linux pipe operators](#linux-pipe-operators)
      * [Short-lived infrastructure](#short-lived-infrastructure)
      * [Persistent volumes](#persistent-volumes)
      * [Filesystem layout](#filesystem-layout)
      * [What is vcap?](#what-is-vcap)
      * [Packages](#packages)
      * [Releases, Part 1](#releases-part-1)
   * [Deployment manifests, Part 1](#deployment-manifests-part-1)
      * [Sizing a deployment, Part 1](#sizing-a-deployment-part-1)
      * [Explicit declaration in manifests](#explicit-declaration-in-manifests)
      * [Immutable manifest attributes](#immutable-manifest-attributes)
      * [Cloud Config, Part 1](#cloud-config-part-1)

NOTE: update TOC using `bin/replace-toc`

# Introduction

## WIP

I recently started writing this. If you're actually reading this guide now, please let me know (I'll actively ask you to review bits as I write them) and please "Watch" this repo. Perhaps I can update it via Github Releases so you can get notifications of new sections or updates. Or better jokes.

## Guide to the Guide

This Ultimate Guide to BOSH is to be read linearly. Each section will build upon the preceding sections.

This guide will be a single [README.md](README.md) until that just doesn't make sense anymore.

The guide uses sample commands and videos to show you real systems instead of expecting that you can deploy systems yourself on day one.

Later in the guide you will deploy your own BOSH and use it to deploy systems. At that point you will install the `bosh` command-line tool, and you will need to decide which target cloud infrastructure you will use.


## Joyful operations

You're a professional. You're resourceful. You're a king maker. You keep your organization in the business of winning.

In past lives, you might have been called: developer, sysadmin or devops.

You're always on the look out for better tools, better mental models and better systems.

I'm going to show you how I do some day-to-day activities using BOSH. You get to decide if you'd like to level up your superhero status and learn how to do this too. Learning is involved. Effort. New tools. New ecosystem. I definitely think it's worth it. Let me know what you decide!

Deploy a 5-node cluster of Zookeeper to Amazon AWS:

```
git clone https://github.com/cppforlife/zookeeper-release
cd zookeeper-release
export BOSH_ENVIRONMENT=aws
export BOSH_DEPLOYMENT=zookeeper
bosh deploy manifests/zookeeper.yml
```


Sanity check that the zookeeper cluster is working:

```
bosh run-errand smoke-tests
```

Upgrade to the new version of Zookeeper:

```
git pull
bosh deploy manifests/zookeeper.yml
```

Upgrade the base operating system to push out critical security patches:

```
bosh upload-stemcell https://bosh.io/d/stemcells/bosh-aws-xen-hvm-ubuntu-trusty-go_agent
bosh deploy manifests/zookeeper.yml
```

If Amazon AWS deletes one of your VMs, heal your cluster by receating a new VM, reattach the persistent disk, remount it, and restart all the processes to join the Zookeeper node into the cluster:

```
# do nothing, this resurrection will happen automatically
```

List the health of each Zookeeper server, including disks:

```
bosh instances --vitals
```

SSH into one of the Zookeeper servers to check on something:

```
bosh ssh zookeeper/0
```

Run a command on each Zookeeper server and display results:

```
bosh ssh -c '/var/vcap/jobs/zookeeper/bin/ctl status' -r
```

Tear down your Zookeeper cluster:

```
bosh delete-deployment
```

## Brief History of BOSH

I was fortunate to be invited to the VMWare campus in Palo Alto, CA, on April 11, 2012 for the unveiling of "the Outer Shell" that deploys Cloud Foundry. The Outer Shell was called BOSH. This is an acronym for "BOSH Outer Shell". Engineers know one thing: recursion is funny.

I invited myself to the VMware campus for two days to meet the developers of BOSH and Cloud Foundry, and came away fascinated by the vast scope of problems that the BOSH team was trying to solve. The BOSH team was kind enough to either answer a question or fix BOSH so the question was void.

In 2012, I was very publicly excited about BOSH on Twitter ([@drnic](https://twitter.com/drnic)) and I gave presentations at meetups and conferences about BOSH. I also began creating new open source projects to help myself and others to use BOSH and to create new BOSH releases. I also created the first [BOSH Getting Started](https://github.com/cloudfoundry-community-attic/LEGACY-bosh-getting-started) guide.

Many of those early presentations and guides were helpful to the many people who've discovered BOSH and are using it to run production systems at huge scales.

In 2013, the BOSH project was taken over by Pivotal engineering and has been gifted to the Cloud Foundry Foundation to secure its long term success as an open source, open community project. Thanks to Pivotal, IBM and other members of the Cloud Foundry Foundation, the BOSH project has received huge consistent investment to this day.

There are many people in the history of BOSH who have directly made BOSH what is, actively sponsored its investment, or evangelised it.

* James Watters, SVP Product at Pivotal, has been the loudest cheerleader of BOSH on the planet.
* Ferran Rodenas, Consultant at Stark & Wayne, has been my BOSH friend since 2012.
* Dmitriy Kalinin, Product Manager for BOSH, has been driving his incredible vision for BOSH.
* The original BOSH team at VMware - Mark Lucovsky, Vadim Spivak, Oleg Shaldybin and Martin Englund - who have had the original vision and execution to create the ultimate tool for release engineering, deployment, lifecycle management and monitoring of distributed systems.

## BOSH in Production

BOSH is the core technology to Pivotal Ops Manager and its Pivotal Network delivery system for complex on-premise software systems. BOSH is the deployment technology used behind the scenes for [Pivotal Web Services](https://run.pivotal.io) which runs upon Amazon AWS.

BOSH is the deployment technology used behind the scenes by [IBM BlueMix](https://www.ibm.com/cloud-computing/bluemix/) on its Soft Layer infrastructure.

BOSH is the deployment technology used by GE [Predix](https://www.predix.io/), which runs on various clouds and data centres.

BOSH is the deployment technology used by Swisscom [appCloud](https://developer.swisscom.com/), which runs inside Swisscom data centres.

These are huge companies who have small teams running huge production systems using BOSH.

On the smaller end - our consultancy [Stark & Wayne](https://www.starkandwayne.com)  - uses BOSH to run a variety of our internal systems across vSphere, Amazon AWS and Google Compute. (The rest of our systems run upon Cloud Foundry itself, such as https://www.starkandwayne.com and https://www.starkandwayne.com/blog).

## Why Write the Ultimate Guide to BOSH?

BOSH has been my not-so-secret weapon since 2012. Yet you might not yet be using BOSH.

I don't want to write a book.

I wrote a PhD thesis in 2001 and 6 people read it (two supervisors, three judges and myself). Fortunately, this is enough people to be awarded a doctorate. Unfortunately, it doesn't really qualify as "sharing knowledge".

I wrote my first blog post in 2006 and when I quickly reached 7 subscribers. I knew I'd found my preferred medium for sharing.

I enjoy the continuous publication and feedback loop of blogging, and of sharing open source projects. I enjoy a relaxed writing style.

I don't want to write a "book" book.

I want to share all the wonders of BOSH with you. I want you to use BOSH. I want you to feel great using BOSH. I want you to feel like a superhero. I want you to convince your friends and colleagues to use BOSH. I want you to help me evangelise BOSH.

I also want you to switch to Queen's English, learn more about Australia, and to use the Oxford comma.

## Additional Sources of Information

In addition to this "Ultimate Guide to BOSH", there are some other sources of factual knowledge and tutorials.

[BOSH documentation website](https://bosh.io/docs/) is very thorough and new features of BOSH now regularly appear simultaneously in this documentation site.

Duncan Winn's [Cloud Foundry: The Definitive Guide](http://shop.oreilly.com/product/0636920042501.do): he includes an entire chapter, "BOSH All the Things," toward helping you deploy and operate Cloud Foundry.

Stark & Wayne's own blog (["bosh" tag](https://www.starkandwayne.com/blog/tag/bosh/)) has over one hundred articles, tutorials and tiny tips on BOSH.

BOSH is an open source project. You can read the source code and learn how it works. A selection of repositories include:

* https://github.com/cloudfoundry/bosh-cli - the `bosh` CLI
* https://github.com/cloudfoundry/bosh - the BOSH director
* https://github.com/cloudfoundry/bosh-agent - the BOSH agent
* https://github.com/cloudfoundry/bosh-deployment - manifests for various permutations of deploying your own BOSH director

# Why BOSH?

First, let's answer the question:

## What is a running software system?

![app-stack](images/handdrawn/app-stack.jpg)

Your bespoke or user-facing application is either a compiled application (Golang), a source code that runs within an interpreter (Ruby or Python), or is compiled and requires an interpreter (JVM languages).

Your bespoke application will be composed of bespoke code plus third party software libraries (RubyGems for Ruby, NPM for Node, Wheels for Python, etc).

Your application will need to be configured (combination of local configuration files, environment variables, service discovery systems) to run and connect to any dependent systems.

Your application will require local dependencies to be already installed - its interpreter, linked libraries, executable applications, etc.

Your application and its dependencies all require an operating system, formatted disks and the networking to be configured.

All this runs in a virtual server/virtual machines (VMs) either in someone else's data centre called "the cloud" (Amazon AWS, Google Compute, Microsoft Azure) or someone else's data centre called "on premise" (but it's normally not in your building, is it?) running virtualisation software (vSphere, OpenStack).

Your applications and databases running on virtual machines will require disks: local or ephemeral disks that might not survive VM downtime or replacement; and persistent networked disks that are independent of each VM and will be available again if you need (or are forced) to replace your VMs.

All of this runs upon physical machines connected to actual storage systems and interconnected by real-world networking.

Servers need to be powered, so you'll need stable, affordable electricity. Servers get hot, so you'll need cooling. Servers can be stolen or physically hacked, so you'll need security guards with appropriate lapel badges.

It's incredible that it all works. Click on https://google.com to check that it all works.

Note: "The Ultimate Guide to BOSH" will include unsolicited sarcasm and humour. With luck, you'll enjoy both the "Ultimate Guide BOSH" and the humour.

## Choose Your Own Deployment Level

You might define "deploying my system" at a different level to other people:

* using an application platform, such as Cloud Foundry or Heroku
* using a container orchestration system provided by someone else, such as Kubernetes, Docker, Amazon ECS
* using virtual machines provided by someone else, such as Amazon AWS, Google Compute, vSphere
* using bare metal machines provided by someone else
* racking bare metal servers or putting Raspberry Pis into the field

From the perspective of your organization and their goals of efficiently using your time and energy,
hopefully you can start as high up in this stack as possible. For example, there is simply nothing faster, more time efficient, and UI consistent as `cf push`-ing an application to any Cloud Foundry. Every system you deploy should have to first justify why it cannot be deployed to Cloud Foundry, Heroku or Google App Engine.

If you do need to "go down the stack" and take responsibility for more then you will need more help; either your organization will need to expect less from you and your team, or you'll need more tooling, automation and education.

## Assumptions

"The Ultimate Guide to BOSH" assumes you need the latter: you need tooling, automation and education.

It also assumes that you have direct access to your virtualisation/cloud infrastructure - you have suitable AWS credentials, a Google Compute account or vSphere admin access.

"The Ultimate Guide to BOSH" assumes you are prepared to learn a new tool, its features, and its quirks.

## Continuous Integration and Continuous Delivery

BOSH slots in very nicely into any continuous deployment systems you might already be using. The `bosh` command-line tool is a perfect abstraction for "please make this happen" that will make it pleasurable to move BOSH deployments into your CI/CD systems.

# Deployments

Let's begin!

The highest level concept of BOSH is the "deployment" of a system. The purpose of BOSH is to continuous run one or more deployments. For example, a cluster of servers that form a Zookeeper cluster is a deployment of the Zookeeper system.

In [Joyful operations](#joyful-operations) we began by creating a deployment:

```
> export BOSH_DEPLOYMENT=zookeeper
> bosh deploy manifests/zookeeper.yml
```

And we finished the lifecycle of that system by deleting the BOSH deployment:

```
> bosh delete-deployment
```

## New deployments

When we ask BOSH to provision a new system, BOSH takes upon the entire responsibility for making this happen:

* BOSH will communicate with your cloud infrastructure API to request new servers/virtual machines (called "instances")
* BOSH manages the base machine image used for each virtual machine (called "stemcells")
* BOSH will allocate an available IP address for each instance
* BOSH will communicate with your cloud infrastructure API to request persistent disk volumes (we will see soon that the `zookeeper.yml` manifest requires a persistent disk volume for each instance in the deployment)
* BOSH will request that the disk volumes are attached to the instances

Once the instances are provisioned and the disks are attached, BOSH then starts communicating with each instance:

* BOSH will format disk volumes if necessary
* BOSH will download the software required (called "packages")
* BOSH will construct configuration files for the packages and commence running the software (called "job templates")
* BOSH provides service discovery information to each job template about the location and credentials of other instances (called "links")

At this point, it becomes the installed software's responsibility to do things that it needs to. It now has been given a brand new instance running on a hardened base operating system, with a mounted persistent disk for it to store data, and has been configured with the information for forming a cluster with its peers, and connecting as a client to any other systems.

## New Deployments of Zookeeper

Let's revisit each of these actions for the specific case of our 5-instance deployment of Zookeeper running on Amazon AWS.

```
> export BOSH_DEPLOYMENT=zookeeper
> bosh deploy manifests/zookeeper.yml
```

Inside `zookeeper.yml` is the description of an group of five instances, each with a 10GB persistent disk volume (we will review the contents of this file soon).

* BOSH sends requests to Amazon AWS API for five EC2 VMs, using a specified Amazon Machine Image (AMI) as the base file system/operating system
* BOSH will manage the allocation of IPs within the VPC subnet rather than using DHCP (more on networking later)
* BOSH sends requests to AWS for five EBS volumes and then attaches each one to a different EC2 VM

Each of the AWS EC2 VMs will eventually "call home" to BOSH saying that they are awake and ready.

BOSH then begins preparing them for their role of "Zookeeper" instance.

* BOSH downloads special BOSH packages of Apache Zookeeper, plus the Java JDK which is a dependency for running Zookeeper.
* BOSH downloads special BOSH job templates that describe how to configure and run a single node of Zookeeper on each instance
* BOSH provides each Zookeeper job template with the IP address, client port, quorum port and leader election port for every other member of the deployment (these are Zookeeper specific requirements to for a cluster of Zookeeper instances).

## BOSH Architecture, Part 1

In the previous sections, I've made reference to a `bosh` CLI but have otherwise danced around the topic of "What is BOSH really?"

From now on, I will stop simplistically saying "BOSH does a thing" and start to be consistently discerning about which aspect of BOSH is doing something.

Right now, think of BOSH as three things:

* BOSH CLI - the `bosh` command being referenced in the earlier examples. The CLI is a client to the:
* BOSH director - an HTTP API that receives requests from the CLI and either communicates directly with instances or with your cloud infrastructure. Communication with your cloud infrastructure is via a:
* Cloud Provider Interface (CPI) - the specific implementation of how a BOSH director communicates with Amazon AWS, Google Compute, vSphere, OpenStack or any other target.

## CPI - The Ultimate Cloud Provider Interface Abstraction

The CLI, the director and a CPI are the basic components that bring a deployment to life on your target cloud infrastructure.

For our Zookeeper example, we begin with:

```
> bosh deploy manifests/zookeeper.yml
```

The BOSH CLI loads the `zookeeper.yml` file from your local machine (which originally came from a [Github repository](https://github.com/cppforlife/zookeeper-release/blob/master/manifests/zookeeper.yml) in the [Joyful operations](#joyful-operations) section above).

The BOSH CLI forwards this file on to the BOSH director.

The BOSH director decides that it is a new deployment (it has a name that the BOSH director does not know yet). The BOSH director decides it needs to provision five new virtual machines and five persistent disks (we will investigate the contents of `zookeeper.yml` soon). The BOSH director delegates this activity to the BOSH CPI for Amazon AWS (where we are attempting to deploy Zookeeper in our example).

The BOSH CPI is a local command line application hosted inside the BOSH director. You will never need to touch it, find it or run it manually. But it can be helpful to understand its nature. A CPI - the abstraction for how a BOSH director can interact with any cloud infrastructure - is just a CLI. The BOSH director - a long-running HTTP API process - calls out to the CPI executable and invokes commands using a JSON payload. When the CPI completes its task - creating a VM, creating a disk, etc - it will return JSON with success/failure information.

For Zookeeper running on Amazon AWS, our BOSH director will be running with the AWS CPI CLI (TLA BINGO - three, three letter acronyms in a row) installed on the same server. The combination of the BOSH director and a collocated CPI CLI is the magic of how a BOSH director can be configured to communicate with any cloud infrastructure. The CPI CLIs can be written in different programming languages than BOSH director, and be maintained by different engineering teams at different companies. It is a wonderfully powerful design pattern.

This will be the last we will reference the CPIs for a long time. They exist. They allow a BOSH director to interact with any cloud infrastructure. There are many of them already implemented (Amazon AWS, Google Compute Platform, Microsoft Azure, VMWare vSphere, OpenStack, IBM SoftLayer, VirtualBox, Warden/Garden, Docker).

And you will most likely never need to know about them.

Here is the command for deploying five Amazon EC2 servers running Zookeeper, backed by Amazon EBS volumes, running inside Amazon VPC networking:

```
> bosh deploy manifests/zookeeper.yml
```

In the Amazon AWS console, your list of EC2 servers (including the BOSH director VM) might look like:

![zookeeper-deployment-aws](images/zookeeper-deployment-aws.png)

Here is the command for deploying five Google Compute VM Instances, backed by Google Compute Disks, running inside GCP networking, installed and configured to be a Zookeeper cluster:

```
> bosh deploy manifests/zookeeper.yml
```

In the Google Compute Platform console, your list of VM instances (including a NAT VM, bastion VM, and BOSH director VM) might look like:

![zookeeper-deployment-google](images/zookeeper-deployment-google.png)

Never used VMWare vSphere before? Here is the command for deploying a five ESXi virtual machines using a concept of persistent disks on any cluster of physical servers in the world. And they will be Zookeeper:

```
> bosh deploy manifests/zookeeper.yml
```

In VMWare vCenter, your deployment will not specifically look like anything. vSphere is a crazy mess to me.

There are distinctions in deploying any system to any infrastructure that need to be made, but the command above is valid and will work once we have a running BOSH director configured with a CPI. That's fantastic.

# Instances

A deployment is made up of instances. Normally instances represent long-running servers on your cloud infrastructure. They can also represent "errands" - one-off tasks that are run inside of temporary servers.

The BOSH CLI makes it easy to see the list of instances for a deployment and their basic health status with the `bosh instances` command:

```
> bosh instances
Using environment '10.0.0.4' as client 'admin'

Task 1808. Done

Deployment 'zookeeper'

Instance                                          Process State  IPs
smoke-tests/dd931466-4329-46cc-971e-34aeee3f8baf  -              -
zookeeper/146229c3-648a-4280-b725-692b0092eae9    running        10.0.0.9
zookeeper/5b879e13-48b1-40f3-b6a7-ae18ef375bcb    running        10.0.0.7
zookeeper/680367f6-2303-4416-877f-8f1d68d4d978    running        10.0.0.6
zookeeper/aa88d2b6-4a94-4801-b0f4-b82d7b6c05b8    running        10.0.0.5
zookeeper/bc988e19-a8e6-41c4-bc2d-3cad00306aef    running        10.0.0.8

6 instances

Succeeded
```

This is my 5-node cluster of Zookeper, running on one cloud infrastructure or another.

We can see that all the `zookeeper` instances are `running`. There is a 6th instance `smoke-tests` which does not have a Process State. It is an errand instance which has no VM running for it at the time the instances were listed.

For now, we will focus on the long-running instances and return to errands later.

Each instance has at least one assigned IP address. The `zookeeper.yml` manifest did not need to allocate these IP addresses, rather it left this assignment to the BOSH director, the CPI and the cloud infrastructure. It is possible to statically assign IP addresses in a deployment manifest, but ideally there are few reasons to do so. It is not a fun part of a human's day to keep track of IP allocations.

With the BOSH CLI we can also start to introspect what is running on each instance with `bosh instances --ps`:

```
Instance                                          Process    Process State  IPs
smoke-tests/dd931466-4329-46cc-971e-34aeee3f8baf  -          -              -
zookeeper/146229c3-648a-4280-b725-692b0092eae9    -          running        10.0.0.9
~                                                 zookeeper  running        -
zookeeper/5b879e13-48b1-40f3-b6a7-ae18ef375bcb    -          running        10.0.0.7
~                                                 zookeeper  running        -
zookeeper/680367f6-2303-4416-877f-8f1d68d4d978    -          running        10.0.0.6
~                                                 zookeeper  running        -
zookeeper/aa88d2b6-4a94-4801-b0f4-b82d7b6c05b8    -          running        10.0.0.5
~                                                 zookeeper  running        -
zookeeper/bc988e19-a8e6-41c4-bc2d-3cad00306aef    -          running        10.0.0.8
~                                                 zookeeper  running        -
```

Each `zookeeper` instance is running a process called `zookeeper`. Very educational information.

Let's look at another BOSH deployment that collocates more processes and is more interesting. The following deployment is for the Stark & Wayne CI system https://ci.starkandwayne.com.

```
Deployment 'concourse'

Instance                                        Process       Process State   IPs
db/af2a4827-82fd-45f1-b37b-cb7d3c6f4ab9         -             running         10.57.111.7
~                                               postgresql    running         -
haproxy/93ee7e45-8190-4f53-86b2-b1ea211f5cf9    -             running         10.57.111.6
                                                                              184.98.185.163
~                                               haproxy       running         -
web/66bac8cf-3af8-4f7e-b587-b82856f47cdc        -             running         10.57.111.8
~                                               atc           running         -
~                                               tsa           running         -
web/bed6bd94-eecc-4147-9653-5d4bdf40dcf9        -             running         10.57.111.9
~                                               atc           running         -
~                                               tsa           running         -
worker/194ac3c7-0a07-4681-ade9-afbf0e47a1a9     -             running         10.57.111.15
~                                               baggageclaim  running         -
~                                               beacon        running         -
~                                               garden        running         -
```

In this deployment we have 4 different instance groups: `db`, `haproxy`, `web` (there are two instances), and `worker` (I've shown one of them above, but our deployment of [Concourse](https://concourse.ci/) has many `worker` instances).

Each `worker` instance is running three processes: `baggageclaim`, `beacon` and `garden`.

The `haproxy` instance has two IP addresses. The latter `184.98.185.163`, is a public IP on the Internet. All the other IPs are private to the vSphere data centre. This `haproxy` instance is an inbound HTTP load balancer and has a statically assigned public IP for the benefit of configuring the external CloudFlare service which sits infront of receiving https://ci.starkandwayne.com traffic.

The labels for the processes above come directly from inside the running instances. We will now look inside the `worker` instance and match up where this information comes from.

## SSH

To access a shell session on any instance we can use `bosh ssh`:

```
> bosh ssh worker/194ac3c7-0a07-4681-ade9-afbf0e47a1a9
```

If you don't know the long UUID for an instance and you just want to SSH into any instance in the instance group, then use a numbered index such as `/0` or `/1`.

```
> bosh ssh worker/0
```

If your deployment only has a single instance, then you can omit the label altogether:

```
> bosh ssh
```

If you attempt this latter command but your deployment has more than one instance, you will get a red error message similar to:

```
Running SSH:
  Interactive SSH only works for a single host at a time
```

After successfully running a `bosh ssh` command, you will be presented with a shell prompt like:

```
worker/194ac3c7-0a07-4681-ade9-afbf0e47a1a9:~$
```

The start of the prompt indicates that you are inside `worker/194ac3c7...`. The `~` segment means you are in the home folder (as an aside, `cd ~` will take you to the home folder on linux machines).

Each time you open an SSH shell using `bosh ssh` you will be allocated a new user account:

```
$ whoami
bosh_5510a7b92da9475
```

To investigate the processes running, we will need to change to the root user:

```
worker/194ac3c7-0a07-4681-ade9-afbf0e47a1a9:~$ sudo su -
worker/194ac3c7-0a07-4681-ade9-afbf0e47a1a9:~#
```

The suffix of the prompt subtly changed from `$` to `#` to represent we are now a root user.

## Shell User Prompts in Examples

We now have three different shell users in examples: your local machine, a dynamically created non-root user on an instance, and the root user on an instance.

I would like to help make it obvious at all times from which machine/user a command is being run in the examples within the Ultimate Guide to BOSH.

When an example is being run from a local developer machine (that is not inside a BOSH instance) I will prefix the commands with the greater-than symbol `>`. For example, `bosh` commands are run from outside of BOSH instances.

```
> bosh deployments
```

From inside a BOSH instance, I will abbreviate the prompt to `$` in future for non-root user, and `#` for the root user. Like any good person, I will try hard to stick to being a non-root user.

```
> bosh ssh
$ whoami
bosh_5510a7b92da9475
$ sudo su -
# whoami
root
```

If an example does not include a shell prompt (`>`, `$`, or `#`), then it is the contents of a shell script or the output from a command.


**If you `bosh ssh` into a production system and have changed to `root` user, then please place a large cowboy hat on your head. A big one. Everyone needs to know you're a cowboy.**

Instead, try changing to the user for the processes you are working on. On most BOSH deployments the convention is to use a user called `vcap`. To change from your random `bosh_xxx` user to `vcap`:

```
$ sudo su - vcap
$ whoami
vcap
```

Right now we need to be the `root` user to inspect Monit. Generally, you should not need to be `root` user.

## Monit Process Monitoring

```
# monit summary
The Monit daemon 5.2.5 uptime: 8d 3h 26m

Process 'beacon'                    running
Process 'baggageclaim'              running
Process 'garden'                    running
System 'system_localhost'           running
```

The three `Process` entries above match directly to the three `Processes` items in the `bosh instances --ps` output in the preceding section. This is where this information comes from.

Inside each VM we delegate to [Monit](https://mmonit.com/monit/) to start, stop and monitor the health of processes.

Monit plays a small but vital role within every BOSH deployment running on Linux servers. Monit has been the process monitoring heart of BOSH instances since before BOSH was publicly open sourced in 2012. And ever since 2012, every single Product Manager of BOSH has said, "We will replace Monit with something else."

If you learn about using BOSH on Windows, you will discover that instead of Monit, we use native Windows Services to start, stop and repair processes. But for Linux, it's Monit.

The good news is that Monit has an [extensive set of configuration](https://mmonit.com/monit/documentation/monit.html#THE-MONIT-CONTROL-FILE) that you might wish to use in future to describe good process behavior.

Let's regroup and reestablish what we know:

`bosh instances --ps` displays a list of processes and their `running` or otherwise state. This information comes directly from Monit running on each instance.

In daily life you will run `monit summary` or `bosh instances --ps` as part of debugging.

## Job Templates Describe Processes

Earlier in [New deployments](#new-deployments), I introduced the terminology of a "job template":

> BOSH will construct configuration files for the packages and commence running the software (called "job templates")

Job templates are where we configure Monit processes, which in turn runs processes, which in turn results in a running system (of Zookeeper or Concourse or whatever our deployment is designed for). Job templates also configure the processes, describe how to start a process, and how to stop a process.

First, let's finish typing up the story of Monit processes.

Each BOSH instance has a folder `/var/vcap/jobs` containing one or more job templates.

```
# ls /var/vcap/jobs/
baggageclaim  garden  groundcrew
```

This Concourse `worker` instance is configured to run three job templates called `baggageclaim`, `garden` and `groundcrew`.

These three job template named `baggageclaim garden groundcrew` are ALMOST the same as the Monit process names from above, `baggageclaim garden beacon`. If they ever match, it is for convenience.

Each job template folder contains a single `monit` file, which in turn contains zero or more `check process <process name>` instructions.

```
# tail /var/vcap/jobs/*/monit
==> /var/vcap/jobs/baggageclaim/monit <==
check process baggageclaim
  with pidfile /var/vcap/sys/run/baggageclaim/baggageclaim.pid
  start program "/var/vcap/jobs/baggageclaim/bin/baggageclaim_ctl start"
  stop program "/var/vcap/jobs/baggageclaim/bin/baggageclaim_ctl stop"
  group vcap

==> /var/vcap/jobs/garden/monit <==
check process garden
  with pidfile /var/vcap/sys/run/garden/garden.pid
  start program "/bin/sh -c '/var/vcap/jobs/garden/bin/garden_ctl start'"
  stop program "/var/vcap/jobs/garden/bin/garden_ctl stop"

  if failed host 127.0.0.1 port 7777
    with timeout 5 seconds for 12 cycles
    then restart

  group vcap

==> /var/vcap/jobs/groundcrew/monit <==
check process beacon
  with pidfile /var/vcap/sys/run/groundcrew/beacon.pid
  start program "/var/vcap/jobs/groundcrew/bin/beacon_ctl start"
  stop program "/var/vcap/jobs/groundcrew/bin/beacon_ctl stop"
  group vcap
```

We see above that the `groundcrew` job template's `monit` file describes `check process beacon`. This is where the `beacon` name comes from. If we ever observe a problem with the `beacon` process, we now know it is configured inside the `groundcrew` job template.

As an aside, `/var/vcap/jobs/garden/monit` above shows an advanced example of a Monit control file. It includes additional rules for when to automatically trigger a restart of the `garden` process. Monit will monitor port `7777` and if it cannot connect to the application locally then it will restart the process.

Looking again at `groundcrew` job template's `monit` file:

```
check process beacon
  with pidfile /var/vcap/sys/run/groundcrew/beacon.pid
  start program "/var/vcap/jobs/groundcrew/bin/beacon_ctl start"
  stop program "/var/vcap/jobs/groundcrew/bin/beacon_ctl stop"
  group vcap
```

When Monit needs to start `beacon` it will invoke `/var/vcap/jobs/groundcrew/bin/beacon_ctl start`. That is, it invokes `/var/vcap/jobs/groundcrew/bin/beacon_ctl` and passes `start` as the first and only argument.

At this point, the Monit `check process beacon` expects that the Linux Process ID (PID) of a running Linux process will be placed at `/var/vcap/sys/run/groundcrew/beacon.pid`. Monit will continuously watch that this Linux process is running. If it isn't - that is, if the process dies unexpectedly - then Monit will restart to process. That is, it will invoke `/var/vcap/jobs/groundcrew/bin/beacon_ctl start` again.

This is Monit's primary role - to monitor processes (by their PID) and ensure they are restarted if faulty or missing.

When Monit needs to stop `beacon` it will invoke `/var/vcap/jobs/groundcrew/bin/beacon_ctl stop`.

## Job Templates

The `groundcrew` job template will always be located at `/var/vcap/jobs/groundcrew`. The `garden` job template will always be located at `/var/vcap/jobs/garden`.

As it happens, within our `zookeeper` example deployment, each `zookeeper` instance includes a job template called `zookeeper`. This means there will be a `/vcap/vcap/jobs/zookeeper` job template folder. This folder contains a `monit` control script with `create program zookeeper`. Consistency of names - using `zookeeper` as the name of the instance group, job template and monit process - is convenient once you understand what is going on and is a common pattern that you will see.

In summary, all job templates - the configuration of how software is configured and executed - are located on every BOSH instance around the world in the same location: `/var/vcap/jobs/`

**Conventions like this radically lower the mental challenges of support/debugging on running production systems.** You will discover BOSH has many pleasant conventions across all deployments.

Job templates must contain a `monit` file, but that `monit` file can be empty if the job template does not require any processes to be run.

Job templates will also be able to provide any configuration files used by the running processes. Some software requires configuration files. Or software might be configured with environment variables. Job templates will be written to suite the software it is configuring to run.

Let's look at the files within the `zookeeper` job template on a `zookeeper` deployment instance:

```
> bosh ssh zookeeper/0
$ cd /var/vcap/jobs/zookeeper
$ tree
.
├── bin
│   ├── ctl
│   └── pre-start
├── config
│   ├── configuration.xsl
│   ├── log4j.properties
│   ├── myid
│   └── zoo.cfg
├── monit
└── packages
    ├── java
    └── zookeeper
```

As discussed above, the `monit` file is the entry point for a job template being used to run Linux processes. The contents of this file tell Monit what command to run, to start or stop any Linux processes that are required:

```
check process zookeeper
  with pidfile /var/vcap/sys/run/zookeeper/pid
  start program "/var/vcap/jobs/zookeeper/bin/ctl start"
  stop program "/var/vcap/jobs/zookeeper/bin/ctl stop"
  group vcap
```

Within our `/var/vcap/jobs/zookeeper` job template directory, the `bin/ctl` script has a `start` command, and Monit expects a PID file to be placed at `/var/vcap/sys/run/zookeeper/pid` (a file location outside of the job template directory that we will revisit soon).

The abbreviated `bin/ctl` shell script showing the `start` and `stop` subcommands is below (the full original source code is [online](https://github.com/cppforlife/zookeeper-release/blob/master/jobs/zookeeper/templates/ctl.erb))

```bash
case $1 in
  start)
    # Hidden: setup of other env vars
    export ZOOCFGDIR=/var/vcap/jobs/zookeeper/config

    # Hidden: create log/pid folders
    echo $$ > /var/vcap/sys/run/zookeeper/pid

    exec chpst -u vcap:vcap \
      /var/vcap/packages/zookeeper/bin/zkServer.sh start-foreground \
      >>/var/vcap/sys/log/zookeeper/stdout.log \
      2>>/var/vcap/sys/log/zookeeper/stderr.log
    ;;

  stop)
    if [ -f $PIDFILE ]; then
      kill -9 `cat $PIDFILE` || true
      rm -f $PIDFILE
    fi
    ;;
esac
exit 0
```

This `bin/ctl` is a very common implementation of a Monit start/stop wrapper script that you will see in most BOSH job templates. A `case` statement that takes `start` and `stop` from the first argument passed to `bin/ctl` and runs one of two different code paths:

* The `start` subcommand will run a single Linux process and ensure it drops the PID of that process.

* The `stop` subcommand will terminate the Linux process if it is running.

Some software is capable running as a background/daemon process and managing its own PID file. Others do not manage their own PID. Some software can do either and the BOSH job template author will have decided in which mode to run the software.

In the `zookeeper` example above, the `bin/ctl start` command it creating the PID file:

```
echo $$ > /var/vcap/sys/run/zookeeper/pid
```

The expression `$$` is PID of the current shell (the running `bin/ctl` script). So the command above is inserting the current running script's PID into a file. Importantly, this file is the same as the `check process zookeeper with pidfile /var/vcap/sys/run/zookeeper/pid` from the `monit` file above.

The `bin/ctl` running script is then uses `exec` to replace the current running shell (the wrapper `bin/ctl` script) with a new command `/var/vcap/packages/zookeeper/bin/zkServer.sh`

If the script only used `echo $$ > pid` and `exec run-software` then the software would be ran with escalated root user privileges. Instead, the Zookeeper application will be ran as a restricted `vcap` user and `vcap` group using the `chpst` command.

Combined together we have a common pattern in many BOSH job templates:

```
echo $$ > path/to/pidfile

exec chpst -u user:group run-something-in-foreground
```

For the `zookeeper` example, the Zookeeper software was ran using a shell script provided by the Apache Zookeeper package (`zkServer.sh`):

```
echo $$ > /var/vcap/sys/run/zookeeper/pid

exec chpst -u vcap:vcap \
  /var/vcap/packages/zookeeper/bin/zkServer.sh start-foreground
```

An alternate pattern to using `exec chpst -u vcap:vcap run-something` that you might find is the use of `su - vcap -c "run-something"`.

## Running Processes, Summary-in-Progress

We have now covered the most important primary concepts of BOSH: it will provision virtual machines on your target cloud infrastructure, it will install job templates that describe how software is to be started and stopped, and it uses Monit to continuously monitor the health of these processes.

We can see the overall health of a BOSH deployment using:

```
> bosh instances --ps
```

From inside a BOSH instance you can see similar information:

```
# monit summary
```

If you want to see all the Linux processes currently running on a BOSH instance, try the `ps` command:

```
$ ps axwwf
```

This command will show all running processes in wide screen mode. You will see the full explicit commands that were ran to start Linux processes. From a `zookeeper` instance this will include this Java monster command:

```
 7913 ?        S<l    0:08 /var/vcap/packages/java/jdk/bin/java -Dzookeeper.log.dir=/var/vcap/sys/log/zookeeper -Dzookeeper.root.logger=INFO,CONSOLE,ROLLINGFILE -cp /var/vcap/packages/zookeeper/bin/../build/classes:/var/vcap/packages/zookeeper/bin/../build/lib/*.jar:/var/vcap/packages/zookeeper/bin/../lib/slf4j-log4j12-1.6.1.jar:/var/vcap/packages/zookeeper/bin/../lib/slf4j-api-1.6.1.jar:/var/vcap/packages/zookeeper/bin/../lib/netty-3.10.5.Final.jar:/var/vcap/packages/zookeeper/bin/../lib/log4j-1.2.16.jar:/var/vcap/packages/zookeeper/bin/../lib/jline-0.9.94.jar:/var/vcap/packages/zookeeper/bin/../zookeeper-3.4.10.jar:/var/vcap/packages/zookeeper/bin/../src/java/lib/*.jar:/var/vcap/jobs/zookeeper/config:/var/vcap/packages/zookeeper/lib/slf4j-log4j12-1.6.1.jar:/var/vcap/packages/zookeeper/lib/slf4j-api-1.6.1.jar:/var/vcap/packages/zookeeper/lib/netty-3.10.5.Final.jar:/var/vcap/packages/zookeeper/lib/log4j-1.2.16.jar:/var/vcap/packages/zookeeper/lib/jline-0.9.94.jar:/var/vcap/packages/zookeeper/dist-maven/zookeeper-3.4.10.jar:/var/vcap/packages/zookeeper/dist-maven/zookeeper-3.4.10-tests.jar:/var/vcap/packages/zookeeper/dist-maven/zookeeper-3.4.10-sources.jar:/var/vcap/packages/zookeeper/dist-maven/zookeeper-3.4.10-javadoc.jar: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /var/vcap/jobs/zookeeper/config/zoo.cfg
 ```

 Perhaps seeing all that will help you debug Zookeeper. Perhaps it will prompt you to change professions.

## Logs

The previous sections have shown you what instances and processes are running in a system. In order to learn more about what all the software is doing as it is running, you will need to be able to view the logs of all the running processes.

The BOSH CLI is very convenient for getting started with logs. You can stream all the logs from all the job templates from all the instances in a deployment with one beautiful command:

```
bosh logs --follow
```

The `bosh logs --follow` flag also has the short alias `bosh logs -f`.

FIXME: `bosh logs --follow` did not work as expected on Zookeeper https://github.com/cloudfoundry/bosh-cli/issues/315

Some systems only emit logs if interesting things are happening. These can be pleasant logs to view.

Unfortunately other systems can emit logs with a frequency which might infer they have nothing better to do. Your screen might be continuously populated with new logs such that it is impossible to understand anything.

You have some options to survive log overload.

* Pipe the logs to `grep` to restrict what you see

    For example, if you're looking for log lines that contain `stderr` string:

    ```
    > bosh logs -f | grep stderr
    ```

* Use `bosh logs` (without the `--follow` or `-f` following flag) and all the current logs will be downloaded to your local machine.

* Use `bosh logs --job name` flag to try scoping down the subset of instances for which you want to see logs.

* Use an external log collector where you may have additional features for searching/filtering logs.

* SSH into each instance and inspect the logs directly

The reason that `bosh logs` can find all the process logs is a convention used amongst all BOSH deployments to place their process logs in the same subfolder `/var/vcap/sys/log`.

Note, this log location is different from many other Linux conventions, such as `/var/log`. We will discuss the alternate filesystem layout of BOSH instances soon.

Within a BOSH instance, you can try watching all the logs using `tail`:

```
$ tail -f /var/vcap/sys/log/{*.log,*/*.log}
```

Different job templates in different deployments will produce different sets of logs, but there is a convention that they will be placed under `/var/vcap/sys/log`.

If we revisit the `bin/ctl start` for `zookeeper` above, we can see how it is storing some logs:

```
exec chpst -u vcap:vcap \
  /var/vcap/packages/zookeeper/bin/zkServer.sh start-foreground \
  >>/var/vcap/sys/log/zookeeper/stdout.log \
  2>>/var/vcap/sys/log/zookeeper/stderr.log
```

Any regular output will be appended to a file `/var/vcap/sys/log/zookeeper/stdout.log`, and any error or warning messages will be appended to a file `/var/vcap/sys/log/zookeeper/stderr.log`.

## Linux Pipe Operators

The `>>` and `2>>` symbols in the example above are Linux pipe operators. They are similar to the pipe operators `>` and `2>`.

Every Linux process can emit two pipes called "standard out" (stdout) and "standard error" (stderr). Typically, applications will print any regular output for the application to stdout and emit errors and warnings to stderr. This means that any text output, to either stdout or stderr, has some additional metadata - was it a regular output or was it an error/warning message.

When we run applications in a terminal or SSH session, anything appearing in these two pipes will be displayed on the screen.

When we use Monit to run applications there is no terminal screen for a human to see. The contents of these two pipes might be ignored. That is, we would not know if applications were emitting good output or error messages.

One option is to explicitly redirect these two pipes to local files. That is what is happening in the `bin/ctl start` example above.

* `>>` will append any stdout pipe contents to the file `/var/vcap/sys/log/zookeeper/stdout.log`
* `2>>` will append any stderr pipe contents to the file `/var/vcap/sys/log/zookeeper/stderr.log`

You will now be able to look inside `/var/vcap/sys/log/zookeeper/stderr.log` and see only the error/warning messages.

If the pipe operators `>` and `2>` are used, then new log files will be created when the application is executed. This will effectively erase any previous logs that might have explained the history of the process before it was restarted. Therefore you will typically see the append operators `>>` and `2>>`.

## Short-Lived Infrastructure

We cannot assume that files or data written to the filesystem will be available forever. Cloud infrastructure providers will not make any commitments to the longevity of their virtual machines and the associated root file systems.

The ephemeral lifespan of virtual machines needs to be considered, and fortunately BOSH is here to help.

If the BOSH director ever discovers that an instance has disappeared or has become unresponsive, it will fix the problem. The BOSH director will resurrect any missing instances.

**Resurrection of infrastructure is an incredibly powerful feature of BOSH that makes it essential to your organization.** And your personal life.

If the original server is missing (perhaps the cloud provider lost the host machine or perhaps someone with admin permissions accidentally deleted it), the BOSH director creates a new server.

Alternately, if the cloud provider API believes the original server still exists but the BOSH director cannot access it, then the BOSH director will request that the server be destroyed and will then create a replacement.

Cloud servers will also be destroyed and recreated during normal BOSH operations, in addition to the abnormal situations above.

Routinely, you will want to upgrade all the base operating systems for all the instances of all your deployments to push out security patches. The base operating system is called a BOSH "stemcell". These are maintained by the BOSH Core team and are regularly released with new security fixes (thanks also go to Canonical who maintain the Ubuntu distribution). You might find that one or two new stemcells are released each month containing security fixes in the base operating system alone.

Fortunately, upgrading all your deployments to new BOSH stemcells is a very easy operation and we will definitely be returning to this topic soon. For now, you need to know that this process will result in your application processes being stopped (via Monit), the original servers being deleted (via the CPI) and new servers being created to replace them (via the CPI). Any files written arbitrarily to the filesystem will be lost.

Another routing operation you will perform that causes instances to be stopped, and the servers deleted and replaced is resizing or scaling up your instances. BOSH CPIs assume that cloud providers do not know how to resize a running server, so they will emulate "resizing" by deleting the original small server and replacing it with a new larger server. Any files written arbitrarily to the filesystem will be lost.

## Persistent Volumes

Fortunately, there is a solution to storing data that survives longer than any ephemeral cloud server: persistent volumes.

Each cloud infrastructure provider has its own implementation of long-lived storage volumes. Amazon AWS has [Elastic Block Storage](https://aws.amazon.com/ebs/) (EBS). Google Compute has [Persistent Disks](https://cloud.google.com/compute/docs/disks/#pdspecs).

BOSH CPIs will map each cloud implementation to a homogenous experience for BOSH instances. Any BOSH instance that has persistent storage enabled will have a folder `/var/vcap/store`. Any files or data written within this folder will survive the turbulent life of ephemeral cloud servers.

Within a BOSH instance we can see that `/var/vcap/store` is implemented as a separate Linux volume. The `df` Linux command will display all the mounted volumes:

```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.7G  4.0K  3.7G   1% /dev
tmpfs           748M  272K  748M   1% /run
/dev/sda1       2.8G  1.2G  1.5G  46% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
none            5.0M     0  5.0M   0% /run/lock
none            3.7G     0  3.7G   0% /run/shm
none            100M     0  100M   0% /run/user
/dev/sda3       9.6G  366M  8.7G   4% /var/vcap/data
tmpfs           1.0M  4.0K 1020K   1% /var/vcap/data/sys/run
/dev/sdb1       9.8G   23M  9.2G   1% /var/vcap/store
```

At the bottom we can see a volume mounted at `/var/vcap/store`. It is almost 10GB in size, has 23MB already used and 9.2GB remaining. These numbers don't mathematically add up at all. In summary, there is a persistent disk and not much of it has been used yet.

The existence and size of the persistent volume is configured in the deployment manifest. For example, the `manifests/zookeeper.yml` file from our ongoing Zookeeper example. Soon we will begin looking inside this YAML file.

For now, know that it can be very simple to declare a persistent disk. A 10GB persistent volume will be provisioned, attached, and mounted at `/var/vcap/store` with the simple manifest entry:

```
persistent_disk: 10240
```

You will also learn that you can select different volume types and provide other cloud configuration using Persistent Disk Pools. For example, it is possible to configure all the different [Amazon EBS Volume Types](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html) and specific IOPS requirements.

## Filesystem layout

The `df` output of filesystem volumes above begins to indicate why BOSH instances have a non-standard filesystem layout (for example, log files do not go into `/var/log` rather they go into `/var/vcap/sys/run`).

The root volume (`/`) of the instance above is very small:

```
/dev/sda1       2.8G  1.2G  1.5G  46% /
```

It only has 1GB of available capacity. We do not want to install software packages or place fast-growing log files on this disk volume. Computer system behavior is unpredictable when disks fill up. The bad sort of unpredictable.

In addition to the small root volume, each BOSH instance is allocated an additional larger volume `/var/vcap/data`:

```
/dev/sda3       9.6G  366M  8.7G   4% /var/vcap/data
```

The size of `/var/vcap/data` is configurable and large, where as the `/` root volume is fixed and small.

**We want to use `/var/vcap/data` for storing everything that is ephemeral, `/var/vcap/store` for everything that is permanent, and avoid the root volume.**

For example, we want to use `/var/vcap/data` to install software packages, job templates and log files. Except I've previously indicated that the filesystem location for these three was `/var/vcap/packages`, `/var/vcap/jobs` and `/var/vcap/sys/log`.

BOSH instances have convenience symlinks so that files are actually stored within the large ephemeral `/var/vcap/data` volume. Some examples:

```
$ ls -al /var/vcap/jobs/zookeeper
lrwxrwxrwx ... /var/vcap/jobs/zookeeper -> /var/vcap/data/jobs/zookeeper/a14a07c550a0...

$ ls -al /var/vcap/packages/*
lrwxrwxrwx ... /var/vcap/packages/java -> /var/vcap/data/packages/java/050ac5643452f2...
lrwxrwxrwx ... /var/vcap/packages/zookeeper -> /var/vcap/data/packages/zookeeper/be0f...

$ ls -al /var/vcap/sys
lrwxrwxrwx ... /var/vcap/sys -> /var/vcap/data/sys
```

## What is VCAP?

In the time before VMWare first publicly announced Cloud Foundry in April 2011, and before they announced BOSH in April 2012, these collections of projects were known as "VMWare Cloud Application Platform". Or VCAP for short. Which naturally became `vcap` anywhere the engineers aesthetically preferred all-lowercase labels. Thus, the common root folder for everything installed on a machine became `/var/vcap`.

There have been occasional half-hearted ideas about renaming `/var/vcap` to something else. Anything else. But the convention that every file in a BOSH instance is nested underneath `/var/vcap` is incredibly pervasive today.

Naming is hard. Renaming is harder.

If you ever want to make a permanent mark in the short-lived, short-attention span world of software and operations, just put your name at the root of a folder system.

## Packages

This is the final stop on our tour of BOSH instances. We arrive at the depot of raw materials of what allows a `zookeeper` BOSH deployment with five BOSH instances to actually behave like a cluster of Apache Zookeeper: the Apache Zookeeper software itself.

If you have had a diverse expose to different operating systems (Windows, OS X, different Linux distributions) then you will have also been exposed to different packaging systems for the distribution and installation of software. This broad exposure will prepare you for BOSH packaging in one special way: you will be lacking the energy and the will to fight against "OMG, why is there another packaging system?!"

BOSH packaging is similar to [Homebrew](https://brew.sh/) packaging for MacOS/OS X in that an installed package is a singular folder. An installed Homebrew package will be located at `/usr/local/Cellar/pkgname`. An installed BOSH package will be locatable at `/var/vcap/packages/pkgname` (and actually stored within the `/var/vcap/data` ephemeral volume as discussed in a previous section).

This model of "a package is a folder" is not compliant with the [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard). But then again, you didn't know that the FHS existed until I just mentioned it.

There is a comfortable convenience to standard Linux filesystems. Your `PATH` environment variable is already setup for the common locations of executables. For example, on a BOSH instance it is:

```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

That's right, there are two folders included in `PATH` for games. I had to look it up - these are not mentioned in the [FHS wikipedia page](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard).

But I personally have a dissatisfaction with the standard Linux filesystem hierarchy: the knowledge of which files belong to each package is obscure. At the time that you need to know "how does this process get started and stopped" or "where are the log files for this process", it's normally urgent and you're in a bad mood.

This is a big positive for the BOSH filesystem hierarchy: you can find everything in a hurray.

You look inside `/var/vcap/packages/pkgname/` and find every executable, shared library, and includable header file associated with that package.

You look inside `/var/vcap/jobs/jobname` and find the Monit control script, wrapper start/stop script, and every other associated configuration file to run that job template.

You look inside `/var/vcap/sys/log` and find all the logs for the processes that are running now or have run in the past.

One difference between BOSH packaging, and every other packaging system (including Homebrew), is that packages are neither first class citizens nor is there any global repositories of shared packages.

Packages do not exist in isolation. Neither do job templates. Instead, they are rolled up into a singular first class entity called the BOSH release.

## Releases, Part 1

A BOSH deployment is a collection of instances (long-running servers and short-lived errands) upon which have been installed job templates and their dependent packages.

In the `zookeeper` deployment, the `zookeeper` job template needs the `zookeeper` and `java` packages installed.

Which `zookeeper` package? You might assume that it is Apache Zookeeper, but which version? Is it the unmodified source code, or a pre-built package? Do we need some additional security or features patched into it?

Which `java` package? Oracle Java or OpenJDK? The JDK or the JRE? Which version?

There is an important assumption built into the very fiber of BOSH that each of these questions is important and the answer potentially critical. Even if the initial person who deployed Zookeeper initially didn't specifically care which version of Apache Zookeeper or which JDK they used; everyone from that day onwards will need to care.

Each version of a BOSH release combines a specific combination of packages and job templates that are written and packaged to work together. Instead of worrying about a matrix of Zookeeper and Java editions and versions that might work together, a BOSH release will explicitly package one of each together as a known good pair.

If a security patch is released for Apache Zookeeper, then a new version of the entire BOSH release needs to be created, tested and deployed.

If you need to switch from Oracle Java to OpenJDK, then a new version of the entire BOSH release needs to be created, tested and deployed.

For this reason, I say that packages and job templates are not first class citizens in BOSH. BOSH releases are the primary unit of deployment into BOSH instances.

The method for selecting specific versions of specific BOSH releases to be combined into a BOSH deployment is using the deployment manifest.

## Deployment Manifests, Part 1

To provision a new deployment we provide a deployment manifest to the BOSH director. To make modifications to an existing deployment, we provide the BOSH director with a modified deployment manifest.

A deployment manifest is the explicit declaration of what software needs to run, with specific configuration properties, on each different instance.

**The same deployment manifest deployed today should produce the same running system if you deployed it again in 5 years time.**

Our first example deployment manifest will be `zookeeper.yml`, which I've been referring to throughout "The Ultimate Guide to BOSH" so far (the [original file](https://github.com/cppforlife/zookeeper-release/blob/207c9d79eb12399dffe6df7f89abd854d4888f3e/manifests/zookeeper.yml) at time of writing). Below is a subset of the manifest that references the concepts covered so far:

```yaml
---
name: zookeeper

releases:
- name: zookeeper
  version: 0.0.6
  url: https://bosh.io/d/github.com/cppforlife/zookeeper-release?v=0.0.6
  sha1: eea677d086161ada53dc7f6a056e94023384bba0

instance_groups:
- name: zookeeper
  instances: 5
  jobs:
  - name: zookeeper
    release: zookeeper
  persistent_disk: 10240

- name: smoke-tests
  lifecycle: errand
  jobs:
  - name: smoke-tests
    release: zookeeper
```

BOSH deployment manifests use [YAML](http://yaml.org/) markup language. YAML is a relatively human-readable format though your friendship with YAML will be stretched when deployment manifests grow beyond a 100 lines or more.

The goal of a deployment manifest is to describe a deployment that will be reproducible again and again in the future. Three years from now we do not want to accidentally see different packages or configuration files being installed unless we explicit upgraded to them.

In the example manifest above, the top level sections of this YAML file are:

* `name` is the unique name of this deployment within a BOSH director. When a BOSH director receives subsequent deployment manifests with the same `name` it will assume it is an upgrade of the existing deployment.
* `releases` lists the specific BOSH release versions that are to be used, which almost means the specific sets of job templates and packages.
* `instance_groups` lists the sets of instances that will run the same job templates/packages as each other. Instance groups will be deployed as long running instances by default. The configuration `lifecycle: errand` means they will instead be errands (to be discussed later).

## Sizing a Deployment, Part 1

In our example subset manifest above, there are two attributes we can change to resize the infrastructure used for our deployment:

* `instances: 5` will set or change the number of instances used
* `persistent_disk: 10240` will set or change the size of the persistent volume attached to each instance

For a brand new deployment, setting these attributes and invoking `bosh deploy` will establish the initial CPI requests to the cloud infrastructure.

For an existing deployment, changing these attributes and invoking `bosh deploy` will perform a transformation of your system from its current cloud infrastructure servers and disks to the new requirements - more or fewer servers, and/or smaller or large persistent disks.

We will review how the BOSH director performs these transformations of existing deployments later. It's pretty fabulous.

## Explicit Declaration in Manifests

If we keep this manifest the same we will always get the same deployment of instances, job templates and packages year after year. This is achieved by our explicit declaration of `releases`.

In the example above, we explicit require `zookeeper/0.0.6` BOSH release. The `0.0.6` version number is only relative to preceding versions of the same BOSH release, not to any upstream packages. At the time of writing, the `zookeeper` BOSH release being used was packaging Apache Zookeeper v3.4.10 ([list of source blobs](https://github.com/cppforlife/zookeeper-release/blob/207c9d79eb12399dffe6df7f89abd854d4888f3e/config/blobs.yml)).

If you deployed `zookeeper/0.0.6` every day for a year, you would always be deploying Apache Zookeeper v3.4.10 as it is packaged inside the BOSH release. It would always use the same Monit control script, the same Monit start/stop wrapper script, and the same configuration templates.

In the deployment manifest we decide which instances install which job templates/packages within the `instance_groups` section.

The `zookeeper.yml` example above specifies as single long-running group of instances:

```yaml
instance_groups:
- name: zookeeper
  instances: 5
  jobs:
  - name: zookeeper
    release: zookeeper
  persistent_disk: 10240
```


This group of instances will be known within the deployment by its name `zookeeper`. This instance group name is coincidentally the same name as the entire deployment. Whilst deployment names are unique across deployments within the same BOSH director, instance group names only need to be unique within a deployment. You can have many different deployments in one BOSH director with an instance group called `zookeeper`, but they would all need different deployment names.

Each instance in the group will have a 10GB persistent disk (the plain number `10240` is in MB).

The software, configuration and start/stop scripts running on each `zookeeper` instance is described by the `jobs:` section of an instance group.

```yaml
jobs:
- name: zookeeper
  release: zookeeper
  properties: {}
```

This `jobs` section declares that it will install a job template called `zookeeper` from the release named `zookeeper`. The consistency of naming the deployment, instance group, release, and job templates all `zookeeper` is mentally efficient eventually.

But right now, you might find it confusing.

## Immutable Manifest Attributes

Let's rename as many attributes in this manifest as we can and discuss which attributes we cannot modify.

```yaml
---
name: zookeeper-deployment

releases:
- name: zookeeper
  version: 0.0.6
  url: https://bosh.io/d/github.com/cppforlife/zookeeper-release?v=0.0.6
  sha1: eea677d086161ada53dc7f6a056e94023384bba0

instance_groups:
- name: zookeeper-instances
  instances: 3
  jobs:
  - name: zookeeper
    release: zookeeper
  persistent_disk: 20480

- name: smoke-tests-errand
  lifecycle: errand
  jobs:
  - name: smoke-tests
    release: zookeeper
```

Compare this manifest to the earlier version and see that we have modified the following attributes:

* Deployment name changed to `zookeeper-deployment`
* Instance group `zookeeper` renamed to `zookeeper-instances`
* We've reduced the cluster size from five instances to three
* We've resized the persistent disk from 10GB to 20GB
* Errand `smoke-tests` renamed to `smoke-tests-errand`

These were the only attributes in our manifest subset that were easily modifiable or renamable.

Conversely, the following attributes of the manifest were not easily modifiable:

* The `name` of each item in the `releases` section comes from the BOSH release we use; we cannot rename or alias them within the manifest
* The `verion`, `url`, and `sha1` are related to each other to describe which BOSH release to use; if we want to upgrade to a newer release we would change these
* Within `jobs` sections of instance groups the `release` must match one of the names in the top-level `releases` section. At the top-level these are immutable, so they are correspondingly immutable within the `jobs` sections of instance groups.
* Within `jobs` sections of instance groups, the `name` of a job template is derived from the BOSH release itself (more on this soon), and cannot be renamed or aliased within a deployment manifest.


## Cloud Config, Part 1

The number of instances and the size of persistent disks are attributes that are not specific to which cloud infrastructure you are using. Conversely, the specific details about the size of the servers is is specific to each cloud infrastructure. On vSphere you will want to specify the explicit allocation of CPUs, RAM and ephemeral disk. Whereas, when using Amazon EC2, you might choose between a list of [Instance Types](https://aws.amazon.com/ec2/instance-types/), such as `m4.large`, or `t2.medium`. Similarly, Google Compute [Machine Types](https://cloud.google.com/compute/docs/machine-types) have a list of predefined sizes you can provision, albeit with a different set of names such as `n1-standard-1`, and different attributes from Amazon EC2.

This crossover from the deployment manifest to specific cloud infrastructure requirements is not placed in the deployment manifest. Instead, we will have provided our BOSH director with "Cloud Config" to educate it about how we wish our deployment manifests to be applied to our cloud infrastructure.

Each BOSH director has a Cloud Config specification. We can download and view a BOSH director's Cloud Config:

```
bosh cloud-config
```

There are several areas of configuration, but for now let's look at one - `vm_type` - the specification of the size of servers. An example of a `bosh cloud-config` for Google Compute might look like:

```yaml
vm_types:
- name: default
  cloud_properties:
    machine_type: n1-standard-2
    root_disk_size_gb: 20
    root_disk_type: pd-ssd
```

For Amazon EC2, it might include:

```yaml
vm_types:
- name: default
  cloud_properties:
    instance_type: m4.large
    ephemeral_disk:
      size: 25_000
```

For vSphere, it might include:

```yaml
vm_types:
- name: default
  cloud_properties:
    cpu: 2
    ram: 1024
    disk: 3240
```

They are all different because the nature of BOSH deployments is they can target any cloud infrastructure. Whereas deployment manifests attempt to be independent of the underlying cloud infrastructure, it is necessary to eventually specify what you will provision and pay for. The `bosh cloud-config` documents this and it is a shared specification across all deployments in the same BOSH director.

As the `cloud-config` is shared across deployments your organization, and the broader BOSH community, will want to agree on a minimal set of `vm_types` that any basic deployment manifest will work with any basic `cloud-config`. In the three examples above, each `vm_types` list includes a `name: default` item. In production your `cloud-config` might include many other `vm_types` that you'd like to use within your deployments, but it is good practice to include at least one called `default`.

Each `vm_type` also has a `cloud_properties` section whose contents is specific to the target cloud infrastructure and the CPI being used within the BOSH director.
