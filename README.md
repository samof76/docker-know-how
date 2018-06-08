# Docker know how

Well i took up this putting myself in the spot of teaching docker and its internals. This is complete notes I go on a question about learning dockers deeper, not just command, but also the history, and rational of something is the way it is.

First up a bit of history to how docker came about. [dotCloud](https://dotcloud.com), is YC funded company, with the ambition of changing how application are deployed, in moment I will talk about just that. But the company was not did not really take off as it would like, in [PyCon 2013](), Solomon Hykes gave this lightening talk...

[![Solomon Hykes at PyCon 2013](https://img.youtube.com/vi/wW9CAH9nSLs/0.jpg)](https://youtu.be/wW9CAH9nSLs)

As one of the comments says _"This is video should do into the computer history museum."_, I believe it should.

But getting back to the ground, that was the first time every, docker was introduced to general public, and soon after that conference in April 2013, Docker was Opensourced, and soon became a rage we as developers, deployers and operators cannot avoid. Now I am hoping to piece together the answers to the following questions(not necessarily in that order)

- What makes docker click?
- Whats the technologies behind?
- Why should I bother about docker?

## Containers

Its is important to know that containers are not new this world by any means, BSD had Jails and Sun had Zones, well before we had the big blue whale. So what are containers, think of them as chroot on steroids, minus the virtualized hardware, and but with all the isolations primitives of a VM. I did not bother to get into the exploring either Jails or Zones, its just so you know the concept is not new. And it is not new to Linux either will as the basis for creating containers [cgroups](https://kernelnewbies.org/Linux_2_6_24#Task_Control_Groups) and [namespaces](https://en.wikipedia.org/wiki/Linux_namespaces) very early on in linux. In 2008 [lxc](https://linuxcontainers.org/) was introduced and is still active development, since it suits the purpose lxc is seems to be an acronym for **L**iniu**X** **C**ontainers, but I really doubt the intentions. Anyway LXC is still in active developments and so the idea of containers is not new and they seem to use three main kernel premitive, namespaces, cgroups and capabilities, to achieve isolation, control and privileges. Here is some understanding what each of them does. 

### Namespaces


Namespaces, on linux, wrap around global resources and abstract that underlying resource so to the process in the namespace the resource seems isloted instance of the global resource. One the overall goals of namespaces to support container isolation, providing the process(es) within them an illusion that they are the only processes in the system. Namespaces can wrap around various types of linux resources, here is a list. The namespace related API as `clone()`, `unshare()` and `setns()`, namespaces are identified using the `CLONE_NEW*` flags.

- Mount namespaces (`CLONE_NEWNS`)
- UTS namespaces (`CLONE_NEWUTS`)
- IPC namespaces (`CLONE_NEWIPC`)
- PID namespaces (`CLONE_NEWPID`)
- Network namespaces (`CLONE_NEWNET`)
- User namespaces (`CLONE_NEWUSER`)

### CGroups

Linux consists of multiple subsystems, at the very basic level all the process share these subsystems without any limitations on what they are allowed on. CGroups provide a mechanisms to the derive quotas on the use of these subsystem, and define mechanism for process to be restricted within these quotas. This is become more essential when we try to use containers, as the each container is isolated by namespaces, and using cgroups we could make processes with those containers believe that they have only a defined amount resource subsystem of which they could utilize. So effectively using CGroups, we would be able to building tree of resource quota on the following resources.


- `cpuset` - assigns individual processor(s) and memory nodes to task(s) in a group;
- `cpu` - uses the scheduler to provide cgroup tasks access to the processor resources;
- `cpuacct` - generates reports about processor usage by a group;
- `io` - sets limit to read/write from/to block devices;
- `memory` - sets limit on memory usage by a task(s) from a group;
- `devices` - allows access to devices by a task(s) from a group;
- `freezer` - allows to suspend/resume for a task(s) from a group;
- `net_cls` - allows to mark network packets from task(s) from a group;
- `net_prio` - provides a way to dynamically set the priority of network traffic per network interface for a group;
- `perf_event` - provides access to perf events to a group;
- `hugetlb` - activates support for huge pages for a group;
- `pid` - sets limit to number of processes in a group.

**NOTE:** Cgroups are similar to the processes, in that they are hierarchical and a child cgroups inherits the properties of parent cgroups and so forth. But the fundamental difference is that they there are multiple such hierarchies, vis-a-vis a single tree of the process model. These multiple heirachies are attributed the multple subsystems are present for Cgroups. Seek to do, `cat /proc/crgroups`.

## Linux Capabilities

At a very basic level privileges in linux come twos, `regular user` and `root`. If a regular user requires the do some privileges operation of say, using a port number lesser than 1024, the user might require root priveleges. Unfortunately giving root privileges to a user would be giving access to all the resources without limitations, and when that user gets compromized the options are unlimited. To overcome this linux came up with `capabilities` that give more granular restrictions on what a particular user could do on the system. Hence reducing the attack surface if that user is compromised.  

Since linux capabilities is digression from the topic, its best keep in mind that use of the capabilities can give fine grained privileges to processes running with the container, and greatly reducing the attack vectors when a container is compromise.

---
The above a features that are provided by modern linux kernel makes isolation, control and security(or privileges) for containers much more easily addressable and flexible. With these mind lets step out into looking at containers and docker a little more.

## Containers Mainstream

As mentioned earlier, linux containers precede docker by an technological epoch. They broke into mainstream linux distros in almost the end of the first decade of the new mellinium. On popular linux distros they went by the name LXC(Linux Containers), and inside Google they were called lmctfy(Let Me Contain That For You).

Here is a diagram depict how conatainers compare with normal VMs.

![VM vs Container](https://github.com/samof76/docker-know-how/raw/master/images/vm_vs_container.png)

### LXC

LXC provided good abrstactions over the Namespaces, CGroups and Capabilities, to contain linux images like ubuntu or fedora. This project is in active development, fixing many of the security and resource management flaws, as they try to me more mainstream. But somehow they failed to captivate the imagination as much as docker did. I feel that the reasons for these where

- It was positioned more as virtualization model than a packaging model
- Development was slow as it was a community owned project
- Barriers of entry was pretty, with much focus on the kernel primitives
- Interfaces were not conginitive enough for developer adoption
- Had not matured enough in the given time frame
- [Developed in C](https://github.com/lxc/lxc)

The opinions on why might differ from person to person, but I have a strong feeling the above sums it up.

### lmctfy(Lum-cut-ee-fy)

Seeing the emergence of docker, Google could no longer hide it's container ship behind the curtains of close source. In 2014, Google opensourced their container mechanism to work with a very strange name, lmctfy(Let Me Contain That For You). But failed to make any impact as the feature to feature comparison did not live up to docker's mechanics, also the [C++ codebase](https://github.com/google/lmctfy) meant it was for the elitists.

### Others in the Room

Well there were others([vagga](https://github.com/tailhook/vagga) and [rkt](https://github.com/rkt/rkt)) who challenged the monopoly that Docker was gonna be, but fell short. As much as I really want a true competition for Docker nothing really worthy has come around yet. In fact all that the challengers could do was the impact the course of docker, as you would see later in this writing.

## Docker

Docker in the recent times has been the eye-candy of the software deployment industry, if you are software developer your chances are that you have heard about, but if you software deployer your chances are bleak if you have not heard about it. It has become the defacto delivery mechanism of all modern software on the cloud, and if you are not deploying using this you are not deploying to the cloud, sadly I am one of them. Nevertheless, the story is already etched for us to go the docker way- it is docker or die. Docker is several things and the following will differentiate it for you.
- Docker Inc., is the company behind docker
- Docker, is the name of the software
- `docker` is the name of the daemon(or service) and the client, will be explicit to mention

It is very important to know the stages of development that docker went through to understand the motivation for those changes. But at the very least it is good to know on Linux, Docker utilizes all the kernel features describe above, `namespaces`, `cgroups`, and `capabilities`. I will try to point out where ever needed, how it tries to achieve this.

### docker

This really early stages of Docker, when it was [first out in the open](https://github.com/docker/engine/tree/5130d3d6ec979ea2d1f9a0da9d5766028571dd9e), docker was a wrapper or facade to the LXC's capabilities. That all the container lifecycle operations were handled by `lxc`, and docker was the both the client and service. The client for the CLI tool to offer command, and subcommands, that would talk to docker engine through an API and docker engine will inturn speaks to `lxc` , for containers. The diagram shows the, control flow.

![Docker on LXC](https://github.com/samof76/docker-know-how/raw/master/images/docker_using_lxc.png)

The marriage will not last very long, as the unprecedented response to this docker's methodology led the way to a sea of requests for improvements. And docker had to look inwards for these two reasons.
- LXC was linux specific, and had no road map for the supporting multiple platforms
- LXC development was mostly community driven, and was not accountable to anyone

### libcontainer

`libcontainer` was answer to the LXC's limitation, and did replace the lxc completely, by directly interfacing with the linux primitives, `namespaces`, `cgroups` and `capabilities`. By this Docker was able to put itself on a track of faster growth. `libcontainer` landed in the Docker in the [0.9.0 release](https://github.com/docker/libcontainer/tree/bfcd86f32d22096ae69a140a1fafc1bc42391b1c). Though started by Docker Inc., the libcontainer became part the [Open Containers Initiative](), which we will come a little later. Now the architecture looked something like this.

### Breaking the Monolith

Though docker was moving along pretty fast but there was one problem, the docker engine codebase it was still a monolith doing all the following operations.
- Providing docker API to the client to operate on
- Providing the image management services for creating and managing images
- Managing the container lifecycle by using `libcontainerd`
- Providing the network subsystem for containers to communicate with the host as well other containers

Well these were some of them, actually except for running, stopping, starting and deleting of containers everything was done by the docker daemon. This meant that during the docker upgrades, the docker instances had to be stopped, and started again, which were quick but not desirable. So the motivations to break this monolith were primarily,
- Ease of innovation
- Fast development model
- The ecosystem demanded the change

This led to an immense effort on the part of the Docker Inc., to break docker daemon down into independent yet cooperating pieces.  The were other developments which led to some interesting changes to docker which will be discussed in sections below. 

### OCI - Open Container Initiative

Around the same time that docker began the exercise to a break down its monolithic daemon, [Open Container Initiative](https://www.opencontainers.org/) was formed to derive some standards in the container industry, to different players could play well with each other. So OCI, came up with.

- [Container Runtime Specification](https://github.com/opencontainers/runtime-spec)
- [Image Specification](https://github.com/opencontainers/image-spec)

These were defining standard in the course of Docker. Docker Inc., played an important role in coming up with these specifications and also contributing a good amount of code to initiative, like the `libcontainer` is now part of the OCI.

###  runc

`runc` is Docker Inc. reference implementation of OCI Runtime Spec, and implementing this Docker Inc., had removed all container runtime code out of its docker daemon. `runc` does just one task, and does it really well, to create and run a container with given parameters. `runc` wraps itself around the `libcontainer`, for bringing up containers. `runc` exits as soon as it creates a container.

### docker-containerd

The the interface glue between the the docker daemon and `runc` was built as the `docker-containerd`. The idea behind the `docker-containerd` is abstact a larger system like the docker daemon from the details regarding image snapshotting, container life-cycle, and through use of OCI runtimes the mechanisms to launch containers.  `docker-containerd` important provided all this with clean API interface for systems like  docker daemon to harness.

Iniatially, main function of the `docker-containerd`, was of _supervisor_ responsible for container lifecycle operations, and that was the only function that would do. But later on its functionality also include image push and pull images, and their management. Containerd was contributed to the CNCF project and live [here](https://github.com/containerd/containerd) on github. 

### docker-containerd-shim

The `docker-containerd-shim` allows for daemonless containers. It basically sits as the parent of the container's process to facilitate a few things.  

First it allows the runtimes, i.e. runc,to exit after it starts the container.  This way we don't have to have the long running runtime processes for containers.  When you start mysql you should only see the mysql process and the shim.  

Second it keeps the STDIO and other fds open for the container incase `docker-containerd` and/or docker daemon both die.  If the shim was not running then the parent side of the pipes or the TTY master would be closed and the container would exit.  

Finally it allows the container's exit status to be reported back to `docker-containerd`.

### docker daemon

Now that all of the functionality has been stripped off, what remains of the docker daemon, is the Docker API for the docker client to interface with. docker daemon is also responsible for providing support for UnionFS variants, aufs, zfs, btrfs,  for the images storage.

### Putting it All Together

The here is the diagram to depict how all the above pieces fit together to provide docker ability to create, run, start, stop and delete containers.

![Docker Current Picture](https://github.com/samof76/docker-know-how/raw/master/images/docker_current_state.png)

So when a user gives 
