---
title: Project 4
nav_order: 5
parent: Projects
summary: P4
---

# Project 4: Hawker Container Engine

Released: Tuesday, 04/12/2022
**Due Date: Wednesday, 05/04/2022 11:59PM CST**


## Overview
This project is meant to give you an understanding of containers. You will
be completing the implementation of a very basic container management engine,
called *Hawker*. You'll be implementing Hawker for Linux, and you'll need to
understand the two mechanisms that the Linux kernel provides to enable them,
namely [namespaces](https://lwn.net/Articles/531114/#series_index) and [cgroups](https://man7.org/linux/man-pages/man7/cgroups.7.html). 

## Hawker
You will not be implementing full container functionality, but enough to get
the look and feel of working with more traditional container platforms.
Basically, your system will allow us to run either a shell or command *inside* of
a container. This container will be isolated in the following namespaces: UTS,
PID, user, mount point (filesystem), network, and IPC. Your code will only need
to deal with the first four (so we are assuming that you won't be engaging in
any networking or IPC activities inside your Hawker container).

## Setup
For this project, you'll want to make sure to develop in a Linux environment.
I'm using a Fedora 30 VM with Linux kernel 5.3.8. You can get away with using
another distro, but you probably want to use one that has `systemd`, otherwise
you can't really count on the cgroup system being started at bootup (you can do
this manually however, as outlined in the cgroups documentation linked below).
You might first want to play around with [Docker](https://www.docker.com/) or [LXC containers](https://linuxcontainers.org/) to get a feel
for how they should work. You should plan on using the LWN series on [cgroups](https://lwn.net/Articles/604609/)
and [namespaces](https://lwn.net/Articles/531114/#series_index) as references.

## Part 1: Getting started

You'll first want to install some dependencies, namely the development packages
for `libcurl` v7.61 or greater and `libarchive`. Hawker uses these to grab images
from the network. You should also install `libcap`. For example, on Fedora:

```bash
sudo dnf install -y libcurl-devel libarchive-devel libcap-devel
```

You can then check out the project code. You must run the setup script before
attempting to use hawker. Otherwise, things will break. The setup script needs
`sudo` access, so you will be asked for your `sudo` password when it runs.

```bash
git clone https://github.com/khale/hawker-skeleton
cd hawker-skeleton
sudo ./setup.sh
make
```

The last command will build the skeleton, but don't expect it to work properly
yet! You will have to complete some functionality first. You should be able to
run `hawker` now, but it won't be very useful. To get an idea of how its used,
you can run it without any arguments or like so:

```bash
./hawker -h
```

I've also provided a reference binary for you if you'd like to see a working
copy doing its thing. You can get it like so:

```bash
curl http://cs.iit.edu/~khale/class/vm-class/project-files/hawker-ref > hawker-ref
chmod +x hawker-ref
./hawker-ref -h
```

I've provided a single container image for you to test with (called `test`). You
can use it with the reference binary to create an interactive shell inside the
container like so:

```bash
sudo ./hawker-ref test /init
```

If you navigate around this container shell, you'll see that this is just
a very simple userspace environment (It's a tiny [BusyBox](https://busybox.net/) distribution).

Your first job is to get `hawker` to spawn off another process that will form the
shell of the container. You'll then use namespaces to isolate this process from
the rest of the system and set up its environment.

## Part 2: Setting up isolation


### Coding Task 1: Setting up namespaces
When you first get the code, it's not going to actually create a container. You
should start by looking in `hawker.c`. This is where the heart of the engine is,
and where you'll be doing all of your work (you don't need to edit `net,img.{c,h}` but
you should understand what they're doing!). All of the functionality that you need to implement is tagged
with `FILL ME IN` comments in the source.

You should start with `main()` in `hawker.c`. The first thing the engine does is
initialize its image server, which involves starting a network subsystem and an
image cache (in a `.hawker` directory under your home directory). The image must
be specified at the command line (just like the `docker` command). Hawker will
then look for the image in its cache, and if it doesn't find it, it will try to
download the image from my image server (hosted on my IIT page). If it
succeeds, it will extract the compressed image into the `~/.hawker/images/`
directory.

This is where the fun begins. After user arguments are parsed, we need to
create a new child process for the container. We will be using the `clone()`
system call for this purpose. You'll want to make sure the proper flags to
`clone()` are set. The source tells you which namespaces you'll need, and you can
use the `man` page for `clone()` to determine the exact flags. At this point, the
program will exit. You should remove this call to `exit()` and allocate a stack
for the new process according to the comments in the source. It's up to you
whether to use a `malloc()` variant or `mmap()` to allocate your child process
stack; both have their merits. A default stack size is given for you
(`DEFAULT_STACKSIZE`) in `hawker.h`.

Once we have a stack, we can `clone()` with it. I've done this for you. Note that
after the `clone` completes, we'll have two processes running. The child process
will be running in separate namespaces (according to the `clone` flags we set
up). However, before we let the child container process loose, we must set up
its environment, so we have to make it wait for us (the parent process) to
finish that setup. I achieved this using `pipe()` (which you should remember from
your OS class). We're using it here as an asynchronous event notification mechanism.
Essentially, the child waits on one end of the pipe for the parent to hang up
the pipe, at which point the child can continue doing whatever it wants. Other
notification mechanisms are possible.

The main thing we need to do here is set up the UID namespace mapping. By
default, Linux does not set up a mapping from UIDs outside the container to
UIDs inside the container, so the process will be setup with a default UID
(2^16-1). However, we'd like our container to be running as root (UID 0) with
group ID (GID) 0.

There are three files we need to modify (after the child is running) to set up
these maps. `/proc/<PID>/uid_map`, `/proc/<PID>/setgroups` and
`/proc/<PID>/gid_map`. See the [LWN article](https://lwn.net/Articles/531419/)
and the [Linux man page](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html) on PID
namespaces for more details. You might want to use the `DEFAULT_MAP` constant
provided in `hawker.h`.

Note that we don't need to do this for the PID namespace, since Linux does this
for us. The command we run will run with PID 1 (essentially as the init
process). This creates some issues with signal handling and zombie processes
since the kernel treats PID 1 as special. While we won't be worrying about
this, some commercial container systems [get around this](https://engineeringblog.yelp.com/2016/01/dumb-init-an-init-for-docker.html) by spawning a special
init process which then launches the user command as PID 2.


------------

If we build and run hawker at this point, a new process will be created in
a separate set of namespaces (you should verify this with the `ps` command
inside and outside of the container, and the `id` command inside the container),
but things will still be broken because we haven't set up the container's
environment (we're not making use of the container image that the network/image
subsystem is providing us). Your next task will address this.

### Coding Task 2: Setting up the container environment

We now move our attention to the code for the child process (`child_exec()` in
`hawker.c`). At this point, the child is waiting for the parent to notify it that
it should continue, but then it simply exits. You will fix this. We need the
child to set up its new environment. To do that, we need to do four things:

1. Change our root directory to the new directory for the image. This directory
   will be `~/.hawker/images/<image-name>`. You should add code that changes to
   this directory using the `chroot()` system call. You might want to make use of
   the `hkr_get_img()` function provided from `img.h`. This will require the image
   argument stored in the `struct parms` struct filled out by the argument
   parser.
2. Change our current directory to our new root directory. Use the `chdir()`
   system call here, but be careful! What directory path should we actually use
   here?
3. Set our new hostname. See the `sethostname()` system call. A default hostname is provided for you in `hawker.h`.
4. Execute the command provided by the user at the command line. We need to use
   the `execvp()` system call for this, which will create a new address space for
   us (the child process), load the binary file of the command provided, and
   execute it. Note that `execvp()` also takes arguments; these should be the
   arguments of the command (which can also be derived from the `struct parms`
   struct.

--------

At this point, hawker should work correctly with namespaces. You can now test
your version with the test image provided for you. For example, I might run:

```bash
sudo ./hawker test /bin/ls
```

Which should print out the contents of the root directory for the test
container image. If I want to get an interactive shell for the container, I can
run:

```bash
sudo ./hawker test /init
```

Note that this is a bit unlike how this is done in Docker. There are some
subtleties when working with an init process and when allocating ttys (see the
extra credit if you'd like to help make this closer to Docker). You should be
able to run commands like `ls`, `cat`, and `ps` from the container now. Note the
PIDs, hostnames, etc. that you see. You should also be able to use other
commands provided by BusyBox (the test image is a BusyBox image).

There's another special command in the container image called `hog`. All it is
going to do is use as much CPU as possible. Open up another terminal and run
`top`` or `htop` in it. Then in your original terminal, run:

```bash
sudo ./hawker test /hog
```

Watch the CPU meter in your other terminal. Indeed, the `hog` command is doing
what we thought it would, pegging your CPU. We want to prevent this from
happening with our containers, so we need to introduce some notion of resource
control.

## Part 3: Setting up resource control

We want to be able to limit the amount of certain resouces that our container
can use. For this project, you'll only be dealing with the maximum amount of
memory the container can use, and the amount of CPU it can use. Run `./hawker -h`
to get an idea of how to use them. As it stands, if we pass values for the `-m`
and `-c` flags, they'll just be ignored. Your task here will be to fix this using
Linux's cgroup subsystem (make sure to do the reading on cgroups above).

### Coding Task 3: cgroups programming

For this task, you'll want to bring your attention back to the `main()` function
in `hawker.c`, right at the point with the comment that says `BEGIN RESOURCE
CONTROL`. There are two things you must do here, setting CPU limits and memory
limits. The cgroups subsystem is manipulated through the VFS subsystem (by
reading and writing virtual files) rather than through standard system calls.
Make sure to do the reading on cgroups to get an idea of how they're used. The
appropriate cgroup directories are already created for you (this is what the
`setup.sh` script did).

You'll have to modify a few files in these directories based on the values
passed to the `-m` and `-c` flags by the user. The files of interest for CPU live
in `/sys/fs/cgroup/cpuacct/hawker/<container-PID>`. The files are
`cpu.cfs_quota_us` and `cpu.cfs_period_us`.

For memory, the file lives in `/sys/fs/cgroup/memory/hawker/<container-PID>`. The
file of interest is `memory.limit_in_bytes`. This will prevent your container
from using too much memory.

You'll need to write appropriate values to these files to setup the proper
resource control. However, you're not done yet, because the resource controls
you've set up have to be associated with a PID that is bound to them! For that,
you'll need to modify the `tasks` file in both of the above cgroups directories.
You should think carefully here; which PID should be written to those files?
Hint: why is the parent the one writing these files?

That's it for this part!

-------

If you did the above task correctly, you should now be able to control the
resources of your container. For example, if we re-run our previous example
(using the `hog` program), we should see its resources being limited. Open up
another terminal with `top` or `htop` again and run this:


```bash
sudo ./hawker -c 50 test /hog
```

You should now see that your container is only using 50% of the CPU! Cool.

That's it! You're done now. If you've finished quickly, take a look at the extra credit examples. Otherwise, you're ready to handin.

## Handin

To hand in your project, you should run `make handin` while connected to the IIT
network (or via VPN), as in previous projects. If you can't hand-in over the VPN, send the 
tarball to me via email.

## Extra Credit Opportunities

If you have time and want to extend your implementation, I'm willing to give you extra credit. You can propose your own extensions, but here are some ideas to start with:

- Have more than one container (see `docker ls` etc.)
- Implement an actual backend image server with a database (e.g., in node.js)
- Extend functionality to include network setup
- Extend functionality to include IPC connections
- Implement a serverless platform with `hawker` as a backend
- Add functionality for piping commands to/from a container by connecting STDIN (see `docker -i`) and STDOUT properly
- Add proper terminal handling (should allocate a ptty, see `docker -t`)
- Add mount propagation to make changes within container permanent in the system
- Make it possible to create device files (e.g., with `mknod`) from the new mount namespaces
- Add ability to modify create/modify images using a `Hawkerfile` manifest (see `docker create`, `docker build`, `Dockerfiles` and company)

## Further Reading
- How [Facebook uses containers](https://lwn.net/Articles/764761/)
- Container orchestration with [Kubernetes](https://kubernetes.io/)
- [Performance of VMs vs Containers](http://cnp.neclab.eu/projects/lightvm/lightvm.pdf)










