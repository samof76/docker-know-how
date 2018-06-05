# Docker Know How

---

# Agenda

1. History
2. Linux Primitives
3. Containers
4. Docker
5. Docker breakdown

---

# History

- 2010: dotCloud YC funded
- March 2013: Solomon Hykes @PyCon 2013
- April 2013: Docker is released
- 2015: Open Container Initiative

---

# Linux Primitives

Idea of containers is not new and they use three main kernel primitives, namespaces, cgroups and capabilities, to achieve isolation, control and privileges.

- Namespaces
- Cgroups
- Capabilities

---

# Namespaces

One of the overall goals of namespaces to support container isolation, providing the process(es) within them an illusion that they are the only processes in the system.

- Mount namespaces (`CLONE_NEWNS`)
- UTS namespaces (`CLONE_NEWUTS`)
- IPC namespaces (`CLONE_NEWIPC`)
- PID namespaces (`CLONE_NEWPID`)
- Network namespaces (`CLONE_NEWNET`)
- User namespaces (`CLONE_NEWUSER`)

---

# CGroups

CGroups provide a mechanisms to the derive quotas on the use of linux subsystems, and define mechanisms for processes to be restricted within these quotas.

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

---

# Linux Capabilities

At a very basic level privileges in linux come twos, `regular user` and `root`. If a regular user requires the do some privileges operation of say, using a port number lesser than 1024, the user might require root priveleges. Unfortunately giving root privileges to a user would be giving access to all the resources without limitations, and when that user gets compromized the options are unlimited. To overcome this linux came up with `capabilities` that give more granular restrictions on what a particular user could do on the system. Hence reducing the attack surface if that user is compromised.

---

# Containers

Its is important to know that containers are not new this world by any means.

- BSD Jails
- Sun Zones
- OpenVZ
- LXC - Still Active Development

Think of Containers as chroot on steroids, minus the virtualized hardware, and but with all the isolations primitives of a VM.

---

# LXC

LXC provided good abrstactions over the Namespaces, CGroups and Capabilities, to contain linux images like ubuntu or fedora. This project is in active development, fixing many of the security and resource management flaws, as they try to me more mainstream. But somehow they failed to captivate the imagination as much as docker did. I feel that the reasons for these where

- It was positioned more as virtualization model than a packaging model
- Development was slow as it was a community owned project
- Barriers of entry was pretty, with much focus on the kernel primitives
- Interfaces were not conginitive enough for developer adoption
- Had not matured enough in the given time frame
- [Developed in C](https://github.com/lxc/lxc)

---

# lmctfy(lum-cut-ee-fy)

Seeing the emergence of docker, Google could no longer hide it's container ship behind the curtains of close source. In 2014, Google opensourced their container mechanism to work with a very strange name, lmctfy(Let Me Contain That For You). But failed to make any impact as the feature to feature comparison did not live up to docker's mechanics, also the [C++ codebase](https://github.com/google/lmctfy) meant it was for the elitists.

---

# Others in the Room

Well there were others([vagga](https://github.com/tailhook/vagga) and [rkt](https://github.com/rkt/rkt)) who challenged the monopoly that Docker was gonna be, but fell short. As much as I really want a true competition for Docker nothing really worthy has come around yet. In fact all that the challengers could do was the impact the course of docker, as you would see later in this writing.
