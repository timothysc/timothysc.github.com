---
layout: post
title: "Hot Rod Hadoop with Tachyon on Fedora 21 (Rawhide)"
date: 2014-2-17 10:00
comments: true
sharing: true
footer: true
categories: [computing, grid computing, Red Hat, Hadoop, Map Reduce, Big Data, BDAS]
---

{% img left /images/Tachyon.jpg  %}

##Background##
Within the last couple of years we've witnessed a natural evolution in
the "Big Data" ecosystem.  Where the common theme that you've probably heard in the
community that, "Memory is King", and it is.  Therefore, if you are looking for performance
optimization in your stack, an "in memory" layer should be part of
the equation.  Enter *Tachyon*, which provides reliable file sharing across cluster 
frameworks.

Tachyon can be used to support different framworks, as well as different filesystems.
So to bound the scope of this post, we will outline how to setup Tachyon on a local installation
to boost performance of map-reduce application whose data is stored in HDFS on Fedora 21.

---

##Special Thanks##
Tachyon is a recent addition to the Fedora channels, and it would not have been possible
without the efforts of [Haoyuan Li](https://github.com/haoyuan), Gil Cattaneo, 
and [William Benton](http://chapeau.freevariable.com/)

---

##References##

*   [BDAS Stack](https://amplab.cs.berkeley.edu/software/)
*   [YouTube - Introduction to Tachyon](http://youtu.be/4lMAsd2LNEE)
*   [Tachyon project](http://tachyon-project.org/)
*   [Fedora BIG Data SIG](https://fedoraproject.org/wiki/SIGs/bigdata)
*   [Going beyond Hadoop](http://labs.ericsson.com/blog/going-beyond-hadoop-some-insights-for-big-data-platforms)

---

##Prerequisites##

*   [Latest Fedora Machine](http://fedoraproject.org/en/get-fedora)
*   [Bootstrapping Your MapReduce 2.X Programming on Fedora 20](http://timothysc.github.io/blog/2013/09/14/hadoop-mapreduce/)

---

##Installation and Setup##
Prior to installing Tachyon please ensure that you have setup your hadoop installation 
as outlined in the pre-reqs.

First you will need to install the tachyon package:

    $ sudo yum install amplab-tachyon

Now you will need to update /etc/hadoop/core-site.xml configuration for hadoop to 
enable map-reduce to take advantage of tachyon, by appending the following snippet: 

    <property>
      <name>fs.tachyon.impl</name>
      <value>tachyon.hadoop.TFS</value>
    </property>

Now that all the plumbing is in place you can restart hadoop

    systemctl restart hadoop-namenode hadoop-datanode hadoop-nodemanager hadoop-resourcemanager

Next, make certain your local HDFS instance is up and running, then 
you will need to perform a tachyon format.

    $ sudo runuser hdfs -s /bin/bash /bin/bash -c "tachyon.sh format"
    > Formatting Tachyon @ localhost
    > Deleting /var/lib/tachyon/journal/
    > Formatting hdfs://localhost:8020/tachyon/data
    > Formatting hdfs://localhost:8020/tachyon/workers

---

###Initialization###

Prior to running the daemons you will need to mount the in-memory filesystem.

    $ sudo tachyon-mount.sh SudoMount

Now you can start the daemons.

    $ sudo systemctl start tachyon-master tachyon-slave

For completeness you can inspect the logs which are located in the standard system location

    $ ls -la /var/log/tachyon

---

###Operation###

Once you've verified tachyon is up and running, you can run a simple mapreduce application
as seen below:

    $ hadoop jar /usr/share/java/hadoop/hadoop-mapreduce-examples.jar wordcount
      tachyon://localhost:19998/user/tstclair/input/constitution.txt
      tachyon://localhost:19998/test1

You'll notice tha tachyon prefix attached to the input and output locations.  This enables 
hadoop to start the TFS shim which will load and write to tachyon.  To verify you can run the following:

    $ sudo runuser hdfs -s /bin/bash /bin/bash -c "tachyon.sh tfs ls /test1"
    > 16.65 KB  02-17-2014 15:41:11:849  In Memory      /test1/part-r-00000
    > 0.00 B    02-17-2014 15:41:12:366  In Memory      /test1/_SUCCESS

If you're interested in grok'ing further you can probably find the part file under /mnt/ramdisk.

##Summary##
Tachyon provides reliable in memory file sharing across cluster frameworks, as we have seen
in our simple example. It also enables some very interesting prospects for other back end filesystems.

In future posts we'll explore more elaborate configurations using tachyon atop different frameworks and filesystems.