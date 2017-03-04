---
layout: post
title: "Bootstrapping your MapReduce 2.X programming on Fedora 20"
date: 2013-09-14 10:00
comments: true
sharing: true
footer: true
categories: [computing, grid computing, Red Hat, Hadoop, Map Reduce, Big Data]
---

<img src="{{ root_url }}/images/ElephantCowboy.jpg" alt="Picture Courtesy of Mauro Flores jr"/>

##Background##
Recently the BIG DATA SIG has added Hadoop 2.0.5 (or 2.X series) to the Fedora channels.  This 
marks the first addition into *any* OS-distribution which meets all the standards, and system integration 
requirements set forth by their steering committee(s).  Don't be fooled, bundling .jars into a package that 
looks like a .rpm or .deb != a compliant package (not even by a long shot).

So to give some props to all the effort that it took to lasso this elephant, this post will outline how to bootstrap 
the default installation for MapReduce development.

--- 

##References##

*   [Fedora 20 Hadoop Integration](https://fedoraproject.org/wiki/Changes/Hadoop)
*   [Fedora BIG DATA SIG](https://fedoraproject.org/wiki/Big_data_SIG)
*   [Upgrading with FedUp](http://fedoraproject.org/wiki/FedUp)
*   [Fedora Packaging Guidelines](https://fedoraproject.org/wiki/Packaging:Guidelines?rd=Packaging/Guidelines)

---

##Prerequisites##

*   Fedora 20 Machine

---

##Installation and Setup (as root)##

First you will need to install all the default hadoop packages and tools required. 

    yum install hadoop-common hadoop-hdfs hadoop-libhdfs hadoop-mapreduce hadoop-mapreduce-examples hadoop-yarn maven-* xmvn* 

Next you will need need to format your namenode:

    runuser hdfs -s /bin/bash /bin/bash -c "hadoop namenode -format"
    
Once your namenode has been formatted you can now start the daemons using the default service methods:

    systemctl start hadoop-namenode hadoop-datanode hadoop-nodemanager hadoop-resourcemanager

Finally you will want to create the default directories: 

    hdfs-create-dirs

---
    
##Setting up a Users Sandbox (as root)##

    runuser hdfs -s /bin/bash /bin/bash -c "hadoop fs -mkdir /user/tstclair"
    runuser hdfs -s /bin/bash /bin/bash -c "hadoop fs -chown tstclair /user/tstclair"
    
---
##Running WordCount (as user)##

For simplicity I've setup a WordCount example on github that you can copy.

    git clone https://github.com/timothysc/hadoop-tests.github

Once it has downloaded you can put the example .txt file into your user location

    cd hadoop-tests/WordCount
    hadoop fs -put constitution.txt /user/tstclair
    
Now you can build WordCount against the system installed .jars.

    mvn-rpmbuild package 
    
Finally you can run:

    hadoop jar wordcount.jar org.myorg.WordCount /user/tstclair /user/tstclair/output 
    
Feel free to cat the part-0000 file to see the results. 

---
##In Summary##
Hadoop 2.0.5 now acts like a standard package, with all the accoutrements folks have come to expect. 

*Giddyup!*
