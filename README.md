# ContainerScripts

This is a series of scripts to help make it easier to manage and work with Podman.  It is opinionated and in some cases may not be of any use to you.  

Note:  Many of these have been replaced with Ansible scripts.  Those are the ones that start with a double underscore.  The others are used by some of the ansible scripts still.

## Use Case
You have one or more servers that you wish to utilize podman to run containerized applictions.  If you are using Pods or Kubernetes then these scripts definitely are of no use to you.  

In addition you desire to run these as much as possible in a rootless environment.  You also are probably running these in a more or less production scenario whereby you want to build a container that will then be utilized for "some" time across potential restarts, because you need stability and consistency between restarts of the container.  

## What This Attempts to Help With
**Rootless**

Rootless containers are not run as root.  They are run as users.  However, Podman provides no ability to get a complete overview from a super-user that can easily show all the containers that are running on the server from all rootless users.  This set of scripts provides a single command to get a complete overview of all containers that are currently on the system.

**Log Rotation**

Automatically sets up log rotation for the container

**Easy access to Log File**

Provides easy access to the log file from the /var/log directory.

**User Name Spacing**

System helps with user name spacing.  Both Setuid and Setgid are updated to use an easier to utilize id scheme.  The default linux just adds 65536 to the previous entry to get the starting id of the new user / group.  These scripts delete that user/group and recreates it with an easier to utilize id.  It rounds numbers to the 100,000th.  So, 200,000; 300,000;...900,000;1000000;1100000, etc.


# How To Use
There are at present 4 scripts that are used.

**ContainerServerSetup**

This script is only run once, the first time you prepare the server to utilize containers.
It does a couple of important things:
* Creates a dummy userID - 13000 - so that we can reserve a range of of ID's (7000 - 12999) for container users.  This forces the Linux system to create new users after this ID, ensuring we have that range (7000 - 12999) for container users.
* Creates a dummy groupID - 13000.  This is for the same reason as the dummy user we just discussed.
* Creates a group called podman.  All container users are made members of this group.  It enables access to certain things the scripts need.


**Creating Containers**

The next 3 scripts are utilized when you wish to create a container.  As stated above, these scripts are not for development situations or situations where you are creating lots of the same containers.  These scripts are more for when you have dedicated applications you wish to run in a container with consistent data.

The scripts will create a user, so that the container can be run rootless.

**ContainerUserCreate**

This must be run as root.  It will create the user to be used to run the container.  It does prompt for some information about the user, but this can all be left blank.  The password will be left blank.  You can change the password later if you desire.  It adds the user to several groups that it needs access to.  It updates the /etc/subuid and /etc/subgid files.

## Building the Container
After you have created the container user, you will need to create a script that will be used anytime you build the Container.  Here is an example of a script that creates a CoreDNS container:
**coredns_create.sh**

`
#!/bin/bash

podman run -d --name=coredns \
                -p 53:53/udp  \
                -v /opt/data/coredns:/coredns-config \
                coredns/coredns:latest -conf /coredns-config/Corefile
`

Next make sure you change to the container user.  In this case - coredns is our user.
`
su -l coredns
`

You will then need to pull the image:
`
podman pull coredns/coredns:latest
`

Then run the Container creation script you just created - coredns_create.sh

**ContainerPostCreation**
After you have run the container creation script, you need to run the ContainerPostCreation.  After that script has run you now have the system setup.

You can start / stop the container as this user.  You can create a systemd script to auto start these as system start.





