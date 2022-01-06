---
layout: home
title: CS 562
nav_exclude: true
seo:
  type: Course
  name: Virtual Machines
---

# {{ site.tagline }}
{: .mb-2 }
{{ site.description }}
{: .fs-6 .fw-300 }

{% if site.announcements %}
{{ site.announcements.last }}
[Announcements](announcements.md){: .btn .btn-outline .fs-3 }
{% endif %}

## Welcome!

This is the webpage for CS 562: Virtual Machines
[IIT](https://iit.edu).  This course is for graduates and advanced undergraduates in Computer Science.

## Course Communication
We'll be using [Discord](https://discord.gg/dXuteHWX) for course communication. 

## Books
The following book is the only required textbook for this course. If you plan
on pursuing computer systems seriously, it is a great book to have as
a reference:

[Virtual Machines: Versatile Platforms for Systems and Processes](http://a.co/2s0kMO6) (1st Edition), by Jim Smith and Ravi Nair, 2005 Morgan Kaufmann.

If you are looking for books to help you in this area, and for good reference texts to have on your shelf, please consider
the following:
- Anderson & Dahlin. Operating Systems: Principles and Practice, 2nd edition, 2014.
- Remzi & Andrea Arpaci-Dusseau. Operating Systems: Three Easy Pieces, available online [here](https://www.ostep.
org).
- Bovet & Cesati. Understanding the Linux Kernel, 3rd edition, 2005.

## Projects
Most of your time in this class will be spent working on projects. You'll learn more about them as we go. 

## Development Environment
For all projects I will provide you with a [Vagrant](https://www.vagrantup.com/) configuration which you can
use to spawn a virtual machine to do your work on. To use Vagrant, you'll need
a VMM installed on your machine. [VirtualBox](https://www.virtualbox.org/) and [libvirt](https://libvirt.org/) (built on QEMU/kvm) are
two free options. VMware Workstation or Fusion for Mac will work as well. If
you're on a Windows box, [HyperV](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/quick-create-virtual-machine) is another option. See [here](https://learn.hashicorp.com/collections/vagrant/getting-started) for getting started
with Vagrant VMs.


## Other Useful Links and Resources
- [Nice tutorial](https://try.github.io/levels/1/challenges/1) for learning git
- [Emulators](https://www.pcjs.org/) for various historically significant machines
- [MS BAISC](https://www.pagetable.com/?p=774) for 6502
- [Interrupt logic](https://www.pagetable.com/?p=410) on the 6502
- [6502 on the cloud](http://www.6502cloud.com/)
- [Visual 6502](http://www.visual6502.org/JSSim/index.html)
- [Godbolt](https://godbolt.org/) compiler explorer
- [6502 ISA reference](https://www.masswerk.at/6502/6502_instruction_set.html)
- Original MOS 6502 [programmer reference manual](http://users.telenet.be/kim1-6502/6502/proman.html)
- [The Morning Paper](https://blog.acolyer.org/)

