# Learn you a Singularity

Looking for independence? Tired of waiting for your hpc admins to installed the latestest packages that were never meant to exist on `CentOs`? Or did they force you to come here so they could worry about other things like ...

Well you've come to the right place (maybe). This tutorial assumes you have minimal knowledge of HPCs or containers and you just want to treat your cluster as a much more expensive (unix) PC. 

Special thanks to the Singularity team who gave birth to this amazing creation.

## Introduction

First we'll need to lay down some common ground. If you think you know everything feel free to skip this section, but remember, you're here because you needed to learn something.


The reason you need to use a **container** is that your cluster may use a different operating system or does not want to deal with conflicting libraries (say needing `libfoo.10` vs `libfool.13`). A **container** is one of two popular methods for extending **environments**. An **environment** is a collection of libraries and binaries needed to run a particular application. The other popular method of extending an **environment** is a **virtual machine**(**VM**). Before I go into the differences, I would like to elaborate what a computer is. 

### The cake is not a lie
For the sake of this tutorial (equivalent to your parents saying "because I said so"), I will define a computer as comprising of three layers: 
 1. File System
 
     Contains the ever precious **environment** that brought you here in the first place
     
 2. Operating System (OS)
 
    Manages housekeeping and access to applications in the File System
    
 3. Kernel
  
    Mediates between the OS and the systems resources: CPU, IO, ...
    
    
## Are we human or are we Singularity?

Now back to the difference between **VMs** and **containers**. A **VM** essentially replicates all three layers, hence the *virtual*. The benifit of this is that the **VM** need not worry about the particulars of the host's kernel. This is useful when trying do perform some cross-platform computing (say ubuntu on windows). However, there is a major drawback: virtualization of all three layers significantly taxes your host. You can imagine this would not be optimal on a cluster, were you could have an army of **VMs** eating up resources. 

Containers do not have this amount of overhead (or why would we be using one?). They do this by merely extending the File System by bypassing the OS and talking to the kernel directly. However, to do this, it must assume that the host is using a particular kernel. Thus most containers rely on a linux kernel (because thats what cool kids use). So really most containers allow you to have any particular flavor of linux on another flavor of linux for minimal cost. 

Enter the new kid: Singularity
Why is it special? Their website does a better job of explaining it than I could but you're reading this because you're lazy so...
