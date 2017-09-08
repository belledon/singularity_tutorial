# Learn you a Singularity

Looking for independence? Tired of waiting for your hpc admins to installed the latestest packages that were never meant to exist on `CentOS`? Or did they force you to come here so they could worry about other things like ...

Well you've come to the right place (maybe). This tutorial assumes you have minimal knowledge of HPCs or containers and you just want to treat your cluster as a much more expensive (unix) PC. 

Special thanks to the Singularity team who gave birth to this amazing creation.

## Introduction

First we'll need to lay down some common ground. If you think you know everything feel free to skip this section, nerd.

The reason you need to use a **container** is that your cluster may use a different operating system or does not want to deal with conflicting libraries (say needing `libfoo.10` vs `libfool.13`). A **container** is one of two popular methods for extending **environments**. An **environment** is a collection of libraries and binaries needed to run a particular application. The other popular method of extending an **environment** is a **virtual machine**(**VM**). Before I go into the differences, I would like to elaborate what a computer is. 

### The cake is not a lie
For the sake of this tutorial (equivalent to your parents saying "because I said so"), I will define a computer as comprising of three layers: 
 1. File System
     Contains the ever precious **environment** that brought you here in the first place

 2. Operating System (OS)
    Manages housekeeping and access to applications in the File System

 3. Kernel
    Mediates between the OS and the systems resources: CPU, IO, ...
    
    
### Are we human or are we Singularity?

Now back to the difference between **VMs** and **containers**. A **VM** essentially replicates all three layers, hence the *virtual*. The benifit of this is that the **VM** need not worry about the particulars of the host's kernel. This is useful when trying do perform some cross-platform computing (say ubuntu on windows). However, there is a major drawback: virtualization of all three layers significantly taxes your host. You can imagine this would not be optimal on a cluster, were you could have an army of **VMs** eating up resources. 

Containers do not have this amount of overhead (or why would we be using one?). They do so by merely extending the File System by bypassing the OS and talking to the kernel directly. However, to do this, it must assume that the host is using a particular kernel. Thus most containers rely on a linux kernel (because thats what cool kids use). So really most containers allow you to have any particular flavor of linux on another flavor of linux for minimal cost (like those strange people who like pinapple on their pizza). 

> Don't drink too much koolaid:
>  You have to have an extra setup of you want to either run Singularity on a non-Linux machine (including Mac). This requires a **VM**.

Enter the new kid: Singularity
Why is it special? Their website does a better job of explaining it than I do but you're obviously not one for reading manuals.

1. Singulary prevents escalation

    Unfortunately for Nixon this means that the type of permissions you have outside of the container are what you have inside. In otherwords, if you have `sudo` on your host you will have `sudo` in the container and vice-versa.
    
2. Singularity is portable
    
    Unlike other containers *(such as Docker which we will get to later)*, the entire singularity environment exists on one file. Essentially this is like having a portal to a universe that you can copy over and over again, creating a multiverse for the inevitable event that you Kronenburg your first one. 
    
2. Singularity enables scientific reproducability
    
    You may not care about this but having your computational environment in a handy Singularity container allows researchers decades from now to run the identical setup you had. No more digging up deprecated libraries that required blood sacrifice to install. Your collegues can now quickly realize that you suck at science. 
    
### The blue pill

For the rest of this tutorial I will need you to hold the following commandments to your heart.

1. When you are modifying a container, you must treat it as a computer independent of your host
2. When you are executing programs via the container you must treat it as an application in your host

Don't worry if that doesn't make sense. Hopefully, by going through the paces, these principles will become more comfortable

## Your first container

### Setting up
[http://singularity.lbl.gov/]:sing_website
Follow the installation instructions from their [website](sing_website). I would strongly recommend installing from source since there are frequent updates.

For Mac users, the **VM** `vagrant` is required.  I would recommend `Option 1` that uses the Singularity team's virtual box.  However, I would make one edit in the `VagrantFile`:
> Un-comment line 40 and include `config.vm.synced_folder "<container folder>", "/vagrant_data"`

The path `<container folder>` leads to the path where you would like to keep your containers. This edit allows `vagrant` to bind that path to `/vagrant_data` with the VM.

### Babies and Bootstraps
Now that we have `singularity` installed in our system (you can test this by running `singularity --version`), we can now get started with making our first container.

This is were we should go back to the first commandment:
> When you are modifying a container, you must treat it as a computer independent of your host

Yes, this means that whatever drivers, libraries, nuclear launch codes you have on your host will **NOT** interact the container unless you ask it to (remember please and thank you).

With the command `sudo singularity create <name>.img`, we generate an empty image.

```
$ sudo singularity create baby.img
Initializing Singularity image subsystem
Opening image file: baby.img
Creating 768MiB image
Binding image to loop
Creating file system within image
Image is done: baby.img
```

A usefull derivative is `sudo singularity create -s <size in MB> <name.img>` which allows the user to specify the size of the image (default is `768MB`)

```
$ sudo singularity create -s 1024 baby.img
Initializing Singularity image subsystem
Opening image file: baby.img
Creating 1024MiB image
Binding image to loop
Creating file system within image
Image is done: baby.img
```
We have now created a blank image, a canvas that we will pour our computational soul into and totally not be dissapointed when it doesn't like the same sports team we do. To do that we will need a boostrap file (also known as a singularity or definition file). This file is a simple bash file that describes the operating system, installed libraries, and persistent environmental variables. Below is an example cribbed from the singularity [docs](http://singularity.lbl.gov/quickstart) that is unorigonally called `Singularity`.


```
# Singularity
Bootstrap: docker
From: ubuntu:latest

%runscript
exec echo "The runscript is the containers default runtime command!"

%files
/home/crabby_patty/secret_recipe.txt /data/not_a_secret_recipe.txt

%environment
WHERESWALDO="/Ireland/Dublin/Merrion_Square_2011"
export WHERESWALDO

%labels
AUTHOR Thomas Anderson

%post
apt-get update 
apt-get -y install  python3 \
                    git \
                    wget
mkdir /data

%test
echo "Greetings, parental unit"
```


>Before we get into the details, I would like to emphasize that creating a new container is an iterative process as you may not know what all of the dependencies are when first starting out. Try an jam as much of the setup as possible into the definition file. This not only goes in line with Singularity's principle on reproducability but can also save you **allot** of time in case you need to remake your container (which you will). So as you mold you computational child, remember to update the definition file regularly.

#### Bootstrap mode and source
Time to break it down, in order of importance rather than appearance in the boostrap file. The first section is the header:
```
Bootstrap: docker
From: ubuntu:latest
```
The `Bootstrap` field determines the mode. For the most part you will probably be using :
1. `debootstrap`:
 This tells singularity that you are building from a mirror on a debian host. For other linux flavors see the Singularity examples [page](https://github.com/singularityware/singularity/tree/master/examples) on their github repo. 

2. `docker`:
    More on this in the next section but essentially it tells Singularity that you plan on useing an already-made Docker container.

The `From` field describes where the data for the container will be pulled from. If you are using `debootstrap`, this is a mirror url (see the github examples). If you are boostraping from `docker` this will be a Docker tag. 

#### Post
Now onto the `post` section. This describes the contents of the File System including additional libraries and directories. 
```
%post
apt-get update 
apt-get -y install  python3 \
                    git \
                    wget
mkdir /data
```

The following sections are optional but are listed in order of importance

#### Environment
```
%environment
WHERESWALDO="/Ireland/Dublin/Merrion_Square_2011"
export WHERESWALDO
```
These exports are done whenever the container is initialized. Singularity **does** import any environmental variables of the host but do not rely on this for settuping a global use case. 

For example, imagine that you have installed some handy-dandy video driver in the `%post` section and you want to make sure that its libraries get added to your path. You may be tempted to be a horrible human being and export the path on your host such as :

```bash
#!/bin/bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path_to_driver_libraries
singularity run some_script.sh
```

DON'T do it, don't be bad parent. If you know that you want a variable available in all of your future use cases, then just include it in the `%environment` section. "What if I don't know the path I want to add before boostraping?" you make ask. We will go over the hands on section of container setup in [Singularity Shell](#singularity-shell) but just like additional dependencies, if you can, update your bootstrap file regularly. 

#### The command

Anxious to actually bootstrap your container? Me too. Here is the command
```
sudo singularity bootstrap baby.img Singularity
```
We require `sudo` because `bootstrap` modifies the container. 

### Docking to Strangers on the internet

Now that we have a conceptual understanding of the boostrap file, I will go describe the relationship between Singularity and Docker more closely. Docker is a commonly used container that hosts hub.docker.com which allows users to download, also known as `pull`, already-made containers from their public collection (think github). The wonderful people over at the Singularity team have made Singularity compatible with Docker containers, meaning that you could browse Docker hub for a container that matches your needs and use it in your bootstrap file. Simply include the name and tag of the Docker container in the `From` section in the form `From: name:tag`. 

For example, the developers of `R`, have hosted (at the time of writing this) a container with the `R` language under the name `r-base`. Of the tags, one might want the most recent version and thus uses `latest. Below would be the correct header for a Singularity container wishing to use this Docker container as a starting point.

```
Boostrap: docker
From: r-base:latest
```
Interacting with people on the internet possibly re-defined humanity. Every potential converstation could lead to new insight and refinement of the human condition. Online, publicly curated, encyclopedias have increased the proliferation of knowledge around the world. It also has gotten kids an `F` in history class. Using Docker hub isn't any different. I do recommend that you search the environment you want on hub.docker.com but be mindful that the container has no gaurantee of effectiveness. Fortunatly, most standard packages (`R`, `CUDA`, `Node`, etc) also stage their own docker containers on Docker Hub. 

> Don't be alarmed when Singularity says things are exploding, this is normal. They only other thing that should be exploding when you follow this tutorial is your mind

### Singularity with Strangers on the internet

>Containers on Singularity hub aren't much safer that Docker hub in terms of efficacy. However, because of the inherent security differences between the two, Singularity containers are less likely to leave your back door unlocked so Michael Myers can't invite himself over with chainsaw.

### Singularity Shell
Now that we have at least the core of the container built (via bootstrap) we can start interacting with our child. But like every proper parent-child relationship we must establish some ground rules and pray that they don't figure out that we don't know what we're doing.

To begin we type the following command
```
sudo singularity shell -w baby.img
```

> Don't worry about the random `-w` flag. This means that we plan to modify the container in our shell session. Also don't think about why we require this flag and `sudo` at the same time. Patience young padiwan. 

Since we are operating under commandment 1, we are not suprised that the only files visible when shelled are files inside of the container. To the unfortunately curious, this is because Singularity prevents default access to the rest of your host's file system in case you decide to delete `/usr/bin`.

Now that we're in the matrix, we can do anything. No actually.. Since we `sudo`-ed into the container, Singularity essentially prepends `sudo` to every user command. Bet you never though you'd use `sudo echo` before huh?

```
echo $POWERLEVEL
9001
```

You can also do useful things like:
``` 
apt-get install gedit
```
It will take some getting used to not having to type `sudo` everytime. If you use any custom libraries that rely on a bash script which uses `sudo` you will have to fix those commands. Why? Because of this (in case you haven't already tried).

```
sudo echo $ANSWERTOLIFE
bash: sudo: command not found
```

Shell can be a great tool to explore your container and has allot of valid uses, especially as having your own personal linux machine on the cluster. But here's a big but! Do not think of this as the normal use case. For everday use (submitting jobs on the HPC) you should follow commandment 2

>  When you are executing programs via the container you must treat it as an application in your host

### Run Singularity, Run!


## Jedi Master's Only

### The black magic of Vagrant

### x2go to hell and back








