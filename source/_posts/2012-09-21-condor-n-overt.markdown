---
layout: post
title: "Elastic Grid with Condor and oVirt Integration"
date: 2012-09-21 3:50
comments: true
sharing: true
footer: true
categories: [computing, condor, grid computing, MRG Grid, Red Hat]
---

##Background##
Gone are the days where an IT administrator could procure a dedicated compute cluster for a single task, so it is often the case where admins are asked to do more with existing resources where possible, especially those which are underutilized.  There are several existing solutions to oversubscription, but few that remain "general purpose" while adapting to the environment as the load within the cluster changes.  Enter Condor, which has been most well known for its batch processing capabilities, but can also be leveraged in many ways as an IaaS tool when coupled with oVirt. 

There have been numerous refs in the past to using the [two tools together](http://youtu.be/kPi8ickYN84), but in this post we will explore the idea of using Condor's integration with oVirt to spin the resources directly from Condor, and briefly cover how administrators could use this capability to spin resources "on demand".

--- 

##Setup##
Before you begin, you will need to configure [oVirt](http://www.ovirt.org/) and a [deltacloud server](http://deltacloud.apache.org/) such that your preconfigured images can be spun via the deltacloud api remotely.  To verify, you can run a simple test program to ensure that it works from a remote machine, as if it were run from condor.

{% include_code c/deltacloud_test.c %}

Once this is done, you will then need to install condor-deltacloud-gahp on the submit machines where you want to spin the resources.

---

##Spinning a oVirt Instance with Condor##
Provided you've setup all the pieces above, you should be able to just submit a grid universe job which referenced the images that you wish to start up.

    universe = grid
    grid_resource = deltacloud http://ovirt.yourdomain.com:3002/api
    executable = ovirt_spin_test
    deltacloud_username = vdcadmin@ovirt.yourdomain.com
    deltacloud_password_file = user_pwd

    # Just specify the rhevm instance name
    deltacloud_instance_name = kvm_test_image_64

    log = job_deltacloud_basic_$(cluster)_$(process).log
    notification = NEVER
    queue

---

##Potential Use Cases##
Given the tight level of integration from Condor and oVirt via deltacloud there are a liteny of use cases which could be crafted by administrators to enable the auto spinning of images from condor, which include:
    
*   Using the JobRouter to configure for overflow
*   Using condor-cron to spin images based on time based activites
*   Using DAGMan workflows along with a monitoring activity to spin and clean resources, based on availability
