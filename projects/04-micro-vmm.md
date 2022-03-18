---
title: Project 3
nav_order: 4
parent: Projects
summary: P3
---



# Project 3: Micro VMM

Released: Monday, 03/21/2022
**Due Date: Monday, 4/11/2022 11:59PM CST**

## Overview
For this project, you will build a small hypervisor using the [KVM API](https://www.kernel.org/doc/Documentation/virtual/kvm/api.txt). I'm not
going to give you much direction here, just a simple specification. You can
build the hypervisor using whatever language you like, but you'll want
something that has bindings to KVM, e.g., C, C++, Rust, possibly Go, or something which would
make it easy to add such bindings. You'll want to start by going through the
[LWN tutorial](https://lwn.net/Articles/658511/) on getting started with KVM. If you're going to work on a VM,
you'll want to make sure you have KVM installed (including the development
headers) and you'll want to make sure that nested virtualization is enabled in
your VM manager. For example, VMware calls this option "enable hypervisor
applications."

## VMM Requirements
- Your VMM will be designed for x86-64, and will run on Linux using KVM
- Your VMM must take a binary file as a command-line argument. This binary file
  will encapsulate some code (for a VM). This code will be relatively simple
  for you, but could eventually evolve into an OS kernel. Your VMM must be able
  to load this binary into guest memory and run it. To get started it will be
  easier to use a raw binary, but if you make significant progress, you may
  want to take a look at libelf to be able to parse and load ELF binaries. This
  would be a necessary step to getting an OS running.
- Your VMM should include a very simple line-buffered console device. It will
  have a very basic interface to the guest. If the guest writes a printable
  byte to IO port 0x42, that byte will be saved in a buffer. If a newline
  character is written, the buffer will be printed to the screen and the buffer
  will be flushed.
- Your VMM must include a virtual keyboard device. This does not have to be
  a full keyboard device, just a few keys are fine. Your VMM will accept input
  one byte at a time from STDIN and set an 8-bit register (IO port 0x44) to
  a value corresponding to the key pressed. It will notify the guest that there
  was a keypress by setting a status register (IO port 0x45) to 1. When the
  guest detects that a key has been pressed, it must ack the event by clearing
  the status register.
- Your VMM will include a virtual interval timer. The guest will write a time
  value (call it N) in milliseconds to IO port 0x46 and will enable the timer
  by writing a 1 to bit 0 of IO port 0x47. The timer is disabled by clearing
  the same bit. While the timer is enabled, it will periodically notify the
  guest on timer events by setting bit 1 on IO port 0x47. The guest must again
  ack and clear this "timer fired" bit when it notices the event. If the guest
  has not ack'd a previous timer event when a new one is fired, the event is
  lost. You may want to look at the GC timing code in Hawkbeans for inspiration
  here. You can also take a look at the Linux timer subsystem (e.g.,
  `timer_create()`).
- Your guest will initially be a simple binary, but you will have to extend it
  to include a console and keyboard driver. The keyboard and console drivers
  will operate according to the hardware spec above.
- You'll probably have noticed at this point that we haven't involved
  interrupts. To make our VMM and guest simpler, we're going to use polled I/O.
  This means that instead of receiving interrupts, the guest will poll on
  device registers to check for events. Your guest must thus enter an event
  loop, checking every device's status register (keyboard and timer) to see if
  an event must be handled. This will save you the pain of dealing with KVM's
  IRQCHIP logic and interrupt routing, and setting up an IDT and interrupt
  handlers in your guest.
- Once you've got this all in place, your guest will accept some input from the
  keyboard, and when the user hits enter, the guest will then echo that line to
  the console on every timer event, until a new line is entered.

## Tips
You probably are not going to want to hand assemble code for your guest (as in
the KVM example linked above). Here is a `gcc` invocation that might help:

```bash
as --32 -o smallkern.o smallkern.S
ld -m elf_i386 -Ttext 0x7c00 --oformat binary -o smallkern smallkern.o
```

This will compile and link your x86 assembly into a flat binary (not ELF). This
assumes that the code will be loaded at address 0x7c00, but you will obviously
want to change this. It's not hard to extend this to work with C code, but
you'll want to use the compiler flags `-ffreestanding`, `-nostdlib`, and `-m32`.

## Next Steps
If you've gotten this far and you're looking to make your VMM/guest more interesting, here are some tips:

- Figure out how to set up interrupts in KVM (see KVM's IRQCHIP calls, and
  irqfd functionality)
- Implement a virtual VGA device, e.g., a simple framebuffer console. See [here](https://wiki.osdev.org/Text_UI).
- Enhance your guest by allowing it to boot into protected and then long
  (64-bit) mode. You'll have to deal with setting up paging and setting up the
  GDT to achieve this. You can also set up the IDT and implement interrupt
  handlers. At this point you'll have a little OS kernel of your own.
- Implement virtio for your guest (paravirtualized network, disk, etc.) and
  VMM. This will allow your guest to use networking/storage without using
  drivers that are very complicated. 
- Take a look at Amazon's [Firecracker](https://github.com/firecracker-microvm/firecracker) or if you're ambitious QEMU for more
  ideas.

## Hand-in
You will hand in your code to me by sending me your code in tarball form (email
is fine). You must include a `Makefile` to build your VMM/guest and a `README`
which describes how to build and run a VM in your hypervisor. Ideally include
scripts to do this (e.g., `make run`).
