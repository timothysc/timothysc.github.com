---
layout: post
title: "Configuring a Personal Hadoop Development Environment on Fedora 18"
date: 2013-04-22 10:00
comments: true
sharing: true
footer: true
categories: [computing, condor, HTCondor, grid computing, MRG Grid, Red Hat, Hadoop, Hadoop-YARN, YARN]
---

##Background##
The following post outlines a setup and configuration of a "personal hadoop" development environment that is much akin to a "personal condor" setup.
The primary purpose is to have a single source for configuration and logs along with a soft-link to development built binaries such that switching 
to a different build is a matter of updating a soft-link while maintaining all other data and configuration.

--- 

##Use Cases##
*   Comparison testing in a local sandbox without altering an existing system installation. 
*   Single source configuration and logs
*   ...

--- 

##References##

Inter-webz:

*   [http://wiki.apache.org/hadoop/HowToSetupYourDevelopmentEnvironment](http://wiki.apache.org/hadoop/HowToSetupYourDevelopmentEnvironment)
*   [http://vichargrave.com/create-a-hadoop-build-and-development-environment-for-hadoop/](http://vichargrave.com/create-a-hadoop-build-and-development-environment-for-hadoop/)
*   [http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/](http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/)
*   [http://wiki.apache.org/hadoop/](http://wiki.apache.org/hadoop/)
*   [http://docs.hortonworks.com/CURRENT/index.htm#Appendix/Configuring_Ports/HDFS_Ports.htm](http://docs.hortonworks.com/CURRENT/index.htm#Appendix/Configuring_Ports/HDFS_Ports.htm)

Books:

*   [Hadoop "The Definitive Guide"](http://www.amazon.com/Hadoop-The-Definitive-Guide-ebook/dp/B0082FE448/ref=dp_kinw_strp_1)

---

##Disclaimers##
*  Currently this is a non-native development setup that uses the existing maven dependencies.  For details on native packaging please visit [https://fedoraproject.org/wiki/Features/Hadoop](https://fedoraproject.org/wiki/Features/Hadoop)
*  The setup listed below is for creating "Single-Node-Cluster"

---

##Prerequisites##

###Configure Password-less ssh###

    yum install openssh openssh-clients openssh-server
    # generate a public/private key, if you don't already have one
    ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
    cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
    chmod 600 ~/.ssh/*
    
    # testing ssh:
    ps -ef | grep sshd     # verify sshd is running
    ssh localhost          # accept the certification when prompted
    sudo passwd root       # Make sure the root has a password

    
###Install Other Build Dependencies###

    yum install cmake git subversion dh-make ant autoconf automake sharutils libtool asciidoc xmlto curl protobuf-compiler gcc-c++ 

###Install Java And Deps###

    yum install java-1.7.0-openjdk java-1.7.0-openjdk-devel java-1.7.0-openjdk-javadoc *maven*

append to your .bashrc file:
    export JVM_ARGS="-Xmx1024m -XX:MaxPermSize=512m"
    export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=512m"

**NOTE:** These instructions have been updated to build against OpenJDK 7 on F18.  Currently (4/25/13), builds are clean but there are 
some test failures.  To get a complete list of failed tests run:

     mvn install -Dmaven.test.failure.ignore=true

---

##Building and Setting up a "personal-hadoop"##

###Building###

    git clone git://git.apache.org/hadoop-common.git
    cd hadoop-common
    git checkout -b branch-2.0.4-alpha origin/branch-2.0.4-alpha
    mvn clean package -Pdist -DskipTests

###Creating Your "personal-hadoop" Sandbox###
In this configuration we default to **/home/tstclair**
    
    cd ~
    mkdir personal-hadoop
    cd personal-hadoop
    mkdir -p conf data name logs/yarn
    ln -sf <your-git-loc>/hadoop-dist/target/hadoop-2.0.4-alpha home

###Override your environment###
append to your .bashrc file:
    # Hadoop env override:
    export HADOOP_BASE_DIR=${HOME}/personal-hadoop
    export HADOOP_LOG_DIR=${HOME}/personal-hadoop/logs
    export HADOOP_PID_DIR=${HADOOP_BASE_DIR}
    export HADOOP_CONF_DIR=${HOME}/personal-hadoop/conf
    export HADOOP_COMMON_HOME=${HOME}/personal-hadoop/home
    export HADOOP_HDFS_HOME=${HADOOP_COMMON_HOME}
    export HADOOP_MAPRED_HOME=${HADOOP_COMMON_HOME}
    # Yarn env override:
    export HADOOP_YARN_HOME=${HADOOP_COMMON_HOME}
    export YARN_LOG_DIR=${HADOOP_LOG_DIR}/yarn
    #classpath override to search hadoop loc
    export CLASSPATH=/usr/share/java/:${HADOOP_COMMON_HOME}/share
    #Finally update your PATH
    export PATH=${HADOOP_COMMON_HOME}/bin:${HADOOP_COMMON_HOME}/sbin:${HADOOP_COMMON_HOME}/libexec:${PATH}

###Verify your setup###
    source ~/.bashrc
    which hadoop    # verify it should be ${HOME}/personal-hadoop/home/bin  
    hadoop -help    # verify classpath is correct.

###Creating Initial Single Configuration Node Setup###
First copy in the default configuration files:
    cp ${HADOOP_COMMON_HOME}/etc/hadoop/* ${HADOOP_BASE_DIR}/conf

NOTE: As your configuration testing space expands it is sometimes useful to have your conf directory to also be a softlink of configuration templates.

Next update your *hdfs-site.xml* with the following: 
{% include_code xml/hdfs-site.xml %}

Append, or update, your *mapred-site.xml* with the following:
{% include_code xml/mapred-site.xml %}

Finally update your *yarn-site.xml* with the following:
{% include_code xml/yarn-site.xml %}

**NOTE:** You may notice that I've included default variables and their corresponding port numbers to ease default hunting.

##Starting Your Single Node Hadoop Cluster##

Format your namenode (only needed for the 1st setup):
    hadoop namenode -format
    #verify output is correct.

Start HDFS:
    start-dfs.sh

open a browser to http://localhost:50070 and verify you have 1 live node. 

Next start yarn: 
    start-yarn.sh

Verify the logs show it's running normally. 

Finally check to see if you can run an MR application:
    cd ${HADOOP_COMMON_HOME}/share/hadoop/mapreduce
    hadoop jar hadoop-mapreduce-example-2.0.4-alpha.jar randomwriter out


**HAPPY HACKING!!!**