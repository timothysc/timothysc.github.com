---
layout: post
title: "Override HTCondor installation with sudo"
date: 2012-11-12 3:00
comments: true
sharing: true
footer: true
categories: [computing, condor, htcondor, grid computing, MRG Grid, Red Hat]
---

##Background##
As a developer, I often find myself wanting to iterate on a build, and test in a sandboxed environment.  Isolating the environment ensures that your changes work well in a controlled experiment, but it has one fundamental flaw, it's not realistic.  In order to truly test a complicated system it's best to put it into some production environment.  This is great for testing, but it has been known to give admins a headache, because you are mucking with a known good installation.  So in this post we will set about the task of overriding an installtion of HTCondor using sudo while keeping the following requirements in mind:

###Requirements###
*   Don't alter the existing system installation (binaries or config files).
*   Be able to reference any custom developer build.
*   Be able to easily change back to the known good installation with little/no effort.


--- 

##Getting Started##

Before you begin you will need a machine which already has HTCondor installed on it, and all the necessary development tools in order to create a custom build.  You will also need to obtain the [source tree for HTCondor](https://github.com/htcondor/htcondor), and follow the instructions on [how to build condor.](https://condor-wiki.cs.wisc.edu/index.cgi/wiki?p=BuildModernization)  I typically 'alias cmake' in my environment with all the clever developer magic to make a sandbox'd installation out of the gate.

    alias cmake='cmake -DCMAKE_INSTALL_PREFIX:PATH=${PWD}/release_dir \
    -DBUILDID:STRING=tstclair_local -DWANT_CONTRIB:BOOL=TRUE \
    -DWANT_FULL_DEPLOYMENT:BOOL=FALSE -D_VERBOSE:BOOL=TRUE \
    -DWANT_MAN_PAGES:BOOL=TRUE -D_DEBUG:BOOL=TRUE'

    cmake . && make install 

Lastly you will need an account which has sudo privs on the machine where you will be tinkering. 

---

##Setting up a Sandbox##

Once you've created build for the target machine that you would like to test, you will need to create a sandbox location which is also accessible by the 'condor' user, I typically use /tmp. 

    mkdir /tmp/mycondor
    cp -r release_dir /tmp/mycondor 
    
Next you will want to drop 3 files into your sandbox directory.

The first file is a simple bash script which kicks off your sandbox'd condor ensuring that all the correct environment variables are passed through sudo so that HTCondor can properly execute out of your sandbox.  
{% include_code scripts/sudo_condor.sh %}
If you are testing your client tools, you will also want to mundge your PATH in your testing shell as seen in the script.

The next file is a script which acts as a piped config script.  In HTCondor, there is a feature which allows admins to generate/mundge the parameters which can be passed in on intialization and reconfig.  It turns out this is useful in meeting our previously mentioned requirement of not mucking with the existing configs while still being able to customize as seen below:
{% include_code scripts/override.sh %}

The final file is an optional condor_config.local file, which you can create.  This file is appended to the end of the existing config and allows the developers, or admins, to lay out any configuration that they desire or even override the system to behave the way they would like.

Finally you will need to adjust the access permission so that the 'condor' user can access the shared location. 

    sudo chown -R "root:root" /tmp/mycondor

Now lets rock N' roll! 
  
    cd /tmp/mycondor
    ./sudo_condor.sh

You've now taken over a existing machine in your pool using all the configuration settings that were there, while still allowing your own config magic.  You should take heed though, if you are testing any changes to condors internal files, you could possibly corrupt what is there.  To avoid this, you can override the SPOOL and EXECUTE directories in your condor_config.local file. 

---

##Verify its correct##

The easiest way to verify correctness is to check that the preamble in the logs(/var/log/condor) contains your BUILDID string, in this case it was 'tstclair_local'.

---

##Potential Use Cases##
It's fairly obvious why a developer would want to do this, but it has many potential use cases outside of just development, which include:

*   Beta testing a new installation prior to a pool upgrade.
*   Testing version compatibility across releases. 
*   Playing with new shiney condor features in a existing pool.

