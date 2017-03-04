---
layout: post
title: "Getting Started with Mesos on Fedora 21 and CentOS 7"
date: 2014-9-08 10:00
comments: true
sharing: true
footer: true
categories: [computing, grid, grid computing, Red Hat, Mesos, Big Data, BDAS]
---

{% img left /images/mesos_logo.png %}

##Background##
For decades now, computer scientists have debated on how
to coordinate groups of heterogeneous compute resources to solve a set of domain
specific problems.  "Scheduling" was the catch all moniker that was used to
describe this space.  This category of problems is *old*, therefore the
scheduling universe is vast, and expansive.

The most recent generation of schedulers that have emerged, strive to address the
problem of coordinating several distributed applications across a data-center.
The reason why I find this interesting, is that many distributed applications
reinvent aspects of a "scheduler", often without realizing the depth and breadth
of the domain they just stepped into.  There is a great ACM article that highlights
this point ["There's Just No Getting around It: You're Building a Distributed System".](http://queue.acm.org/detail.cfm?id=2482856)

Now-a-days, we're seeing a Cambrian explosion of software stacks, each
reinventing pieces of the scheduling wheel.  That's all well and good, but
there is a much easier way.  Enter Apache Mesos, a cluster manager
that provides efficient resource isolation and sharing across distributed
applications.

At its core, Mesos is a **focused** meta-scheduler that provides **primitives** to express a wide
variety of scheduling patterns and use cases. Solutions are **written atop** of Mesos, and are
targeted for a particular use case.  By remaining focused at its core, Mesos
is not architecturally encumbered by domain specific problems that often exist
within other monolithic schedulers.

---
##References##
*   [Introduction to Mesos](http://youtu.be/YB1VW0LKzJ4)
*   [John Wilkes #MesosCon Keynote](http://youtu.be/VQAAkO5B5Hg?list=UUeDh9omC_xMKrar2srQZiLg)
*   [Mesos Architecture](http://mesos.apache.org/documentation/latest/mesos-architecture/)
*   [Building and Running Distributed Systems using Apache Mesos](http://youtu.be/hTcZGODnyf0)
*   [Framework Development Guide](http://mesos.apache.org/documentation/latest/app-framework-development-guide/)
*   [Twitter University](https://www.youtube.com/user/TwitterUniversity)

---
##Overview##
I often find software first impressions are usually pretty important,
and if I have to spend hours setting up an application, then that usually colors
my perspective about the technology.

Therefore, in this post I will run through how simple it is for you to get started with
Mesos on Fedora 21 and CentOS 7.  **I'll leave HA deployments for another post, because I want
to outline just how simple it is to setup for "trying it out".**

---
##Prerequisites##

*   [Latest Fedora Machine](http://fedoraproject.org/en/get-fedora) (OR) [CentOS 7 box ](http://www.centos.org/download/)with[ epel installed](https://fedoraproject.org/wiki/EPEL)

###CentOS 7###
Currently dependent packages have not been fully pulled into CentOS 7, or epel
channels, but I've enabled the [mesos.spec](http://pkgs.fedoraproject.org/cgit/mesos.git/tree/mesos.spec)
to build a bundled distribution for those who want to run on CentOS 7.  For
convenience, rpms can be found [here](https://tstclair.fedorapeople.org/mesos/centos7/)

But for those who want to rebuild it for themselves, you can download the [srpm](http://koji.fedoraproject.org/koji/packageinfo?packageID=17691) and run:
    $ mock --clean --init -r epel-7-x86_64 --rebuild mesos-0.20.0-2.f421ffd.fc21.src.rpm

You will also need to update your docker installation to 1.X.

###Multi-Node Cluster###
*   If you setting up a multi-node cluster, it is recommended that you have DNS setup.
*   You may also want to alter your firewall settings.

---
##Installation##
    $ sudo yum install mesos python-mesos mesos-devel mesos-java

---
##Setup##
Make sure docker is running:
    $ systemctl start docker

###Single Node###
Out of the box, the package is configured to run mesos on a single host machine.  Which is convenient for
developers who just want to install and test their applications locally before submitting to their cluster.

###Multi-Node###
If you want to setup a multi-node cluster there is simply one parameter you need to set on your worker nodes */etc/mesos/mesos-slave-env.sh* file:
    export MESOS_master=yourmaster.yourdomain.com:5050

---
##Running##
###Master Node###
    $ systemctl start mesos-master
and open a browser to [localhost:5050](http://localhost:5050)

###Worker Node###
    $ systemctl start mesos-slave
Now check the "Slaves" tab on the browser window to verify.

###Smoke Test###
    $ mesos execute --command="/bin/sleep 10" --master="yourmaster.yourdomain.com:5050" --name="whizbang"
Verify that it ran under the "Frameworks" tab.

---
###Summary###
Distributed systems can be a complicated and thorny road, but setting up and
deploying them doesn't have to be, and **it's a breeze with Mesos**.

**HaPpY HaCkInG!**
