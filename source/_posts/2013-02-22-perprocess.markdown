---
layout: post
title: "Per-Process Mount Namespaces"
date: 2013-02-22 10:00
comments: true
sharing: true
footer: true
categories: [computing, condor, HTCondor, grid computing, MRG Grid, Red Hat]
---

##Background##
"Isolation" in modern computing comes in many flavors and functions.  Virtual machines, c-groups, chroots/jails,
sanboxing, and namespaces all have a role to play.  In this post we will review process mount namespace isolation, which allows a process to isolate 
its mount points from the outside world.  Furthermore, it enables cleanup of that namespace, which is highly useful in grid applications.  

--- 

##References##

Before you dive too deep into the code below, it's important to read up on some kernel and system goodies:

*    [http://lwn.net/Articles/159077/](http://lwn.net/Articles/159077/)
*    [http://lwn.net/Articles/159092/](http://lwn.net/Articles/159092/)
*    [http://www.freedesktop.org/software/systemd/man/systemd.exec.html](http://www.freedesktop.org/software/systemd/man/systemd.exec.html)
*    [http://www.ibm.com/developerworks/linux/library/l-mount-namespaces/index.html](http://www.ibm.com/developerworks/linux/library/l-mount-namespaces/index.html)

---

##Deep Thoughts##
It may take some time to digest all of the reading and figure out how to apply namespaces to your application.  I recommend 
re-reading the use cases outlined kernel documentation on shared subtrees.  The nugget of goodness that we wish to apply here is under 
section 4B. 

    A process wants its mounts invisible to any other process, but
    still be able to see the other system mounts.

    Solution:

    To begin with, the administrator can mark the entire mount tree
    as shareable.

    mount --make-rshared /

    A new process can clone off a new namespace. And mark some part
    of its namespace as slave

    mount --make-rslave /myprivatetree

    Hence forth any mounts within the /myprivatetree done by the
    process will not show up in any other namespace. However mounts
    done in the parent namespace under /myprivatetree still shows
    up in the process's namespace.

In summary, you will want your parent process to recursively slave mount /your/loc, as not to pollute the inherited namespace especially
in cases where the mount points have shared propagation enabled (Default in Fedora).
    
---

##Testing##

So I've created a simple test application which shows how you to hide subprocess mount points.

{% gist 5015879 %}

For more details, please checkout my [tests repo.](http://github.com/timothysc/tests/tree/master/per_process_mounts)

---

##Applications##

*   Cleaning condor job mounts ;-) 
*   Application security
*   User security