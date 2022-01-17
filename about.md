---
layout: page
title: Syllabus
description: Course policies and information.
---

# Course Syllabus
{:.no_toc}

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Course Overview

Virtual machines have transformed the ways in which we build, manage, and
interact with modern computer systems. A great deal of the network packets that
you send as you sit at your computer are handled by virtual machines that may
move to another side of the world in a matter of hours. If you have an Android
phone, every application you launch executes in a virtual machine. Have you
written a Java or Python program lately? It couldn't run without a virtual
machine sitting underneath it. VMs have enabled new development practices for
production applications, mobile applications, web applications, and operating
system kernels. They allow datacenter operators to more efficiently provision
hardware resources, saving money and energy. They allow service providers to
quickly react to failures in an efficient and clean way. They enable the
construction of portable and platform-agnostic programming languages by
decoupling applications from the hardware on which they run. While VMs have
been around since the early 1970s, modern developments, from cloudlets to
containers, are only increasing the utility of virtualization technologies. One
can expect that their importance and relevance will only increase.

This means that a basic understanding of the technological foundations
underlying virtual machines should be part of any computer scientist's
repertoire. This course will draw back the curtains and expose the magic that
makes various types of VMs work. By the end of the course, you will have a deep
understanding of hypervisors, system virtualization, machine emulation,
language virtual machines, binary translators, virtual resource management, and
more. You will gain exposure to a real-world hypervisor code-base. Furthermore,
you will actually build a virtual machine and develop an intuition for using
VMs to solve problems. The course will involve lectures, written assignments,
involved programming projects, and discussions of foundational research papers.

## Course Goals

The goals for this course will be for you to develop a deep understanding of
various types of virtualization techniques, their advantages and disadvantage,
and to be able to apply them in a practical setting. You will be able to build
basic VM constructs and understand how to evaluate them. You will therefore be
expected to strengthen your system programming skills. You will also learn
about new and upcoming technologies related to virtualization. In general, you
will learn about the following topics (and potentially others):

- General resource virtualization: virtual memory, virtualized CPUs, OS abstractions
- Basic Virtual Machines and historical context. Formal requirements for virtual machines
- Emulation: interpretation and binary translation
- Language VMs and Process VMs: JVM, Python, .NET CLR, Android’s Dalvik JVM, VCODE, JIT compilation
- Full system virtualization: Type I & II VMMs, paravirtualization, device virtualization
- Virtualization in practice: virtual resource management, live migration, memory ballooning, fault tolerance
- Network virtualization: virtual overlays, bridged networking, NAT, software-defined networks
- Virtual storage: storage virtualization, virtual disk formats, copy-on-write formats
- Advanced topics in virtualization: hardware assisted virtualization, overheads of virtualization, performant
VMs, interrupt virtualization, self-virtualizing devices, security, introspection, the semantic gap
- Current and future directions and research: VMMs as microkernels, Unikernels, chroot, jails, containers, cloudlets, virtual
appliances

## Things You'll Learn

- You will gain an understanding of the motivation for virtual machines and the history of their development
- You will learn about the different types of virtual machine architectures and their applications
- You will learn about full system virtualization as it is used in practice
- You will gain an understanding of the nuts and bolts that make modern cloud-based services such as Amazon
AWS work
- You will get extensive experience developing virtual machine architectures. This experience will range from
low-level kernel programming to implementing interpreters for high-level languages and emulators for instruction set architectures
- You will gain experience navigating and understanding a real-world hypervisor code-base
- You will learn about current and future topics in virtualization

## Topics
1. Computer architecture review (3 hours)
2. Virtualization Foundations and Theory (3 hours)
3. Full System Emulation (6 hours)
4. Process Virtual Machines (4 hours)
5. High-level Language Virtual Machines (9 hours)
6. System Virtualization (9 hours)
7. Virtual Machine Management (3 hours)
8. Network Virtualization (3 hours)
9. Lightweight Virtualization and Containers (3 hours)
10. Virtualization Performance and Optimizations (1.5 hours)
11. Current and Future directions (1.5 hours)

## Prerequisites
You must have taken CS 351 and CS 450. I will only waive this requirement in
truly exceptional cases—this will be a challenging course, and you will need to
already be quite familiar with the fundamentals of operating systems. I also
recommend having taken CS 551 and CS 470 or 570 (computer architecture).
You should be comfortable (ideally proficient) with programming in C. I won't take
time to cover basic C programming. 

## Lectures
It is critical you attend class sessions. We will Start out on Zoom, but after that we will
meet in SB 210, barring any changes by the University administration. 

## Textbooks
The following book is the only required textbook for this course. If you plan
on pursuing computer systems seriously, it is a great book to have as
a reference:

[Virtual Machines: Versatile Platforms for Systems and Processes](http://a.co/2s0kMO6) (1st Edition), by Jim Smith and Ravi Nair, 2005 Morgan Kaufmann.

## Grading
Your grade will be based mostly on the course projects. 

Your grade will be computed according to these criteria:
- 10% Attendance and Participation
- 70% Projects
- 20% Project Presentations

For internet and India sections, the grade break-down is:
- 80% Projects
- 20% Project Presentations (you can send me a recording)

### Late work policy
Each day assignment is late, it will incur a 10% penalty. If an assignment is more than 5 days late, it will not be accepted, and you will receive a zero.

## Exams
There will be no exams for this course.

## Academic Integrity

You are allowed to discuss your work with others in the class, but ultimately
what you submit must be solely yours. Cheating, plagiarism, copying from other
students, or any other sort of academic dishonesty may result in you getting
a zero on the assignment, may impact your grade negatively, and may result in
refurral to the Dean. 

Please see IIT's code of Academic Honesty [here](https://www.iit.edu/student-affairs/student-handbook/fine-print/code-academic-honesty).

## Disability Accommodations
Reasonable accommodations will be made for students with documented
disabilities. In order to receive accommodations, students must obtain a letter
of accommodation from the Center for Disability Resources. The Center for
Disability Resources (CDR) is located in Life Sciences Room 218, telephone (312) 567-5744 or disabilities@iit.edu.

## Sexual Harassment and Discrimination Information
Illinois Tech prohibits all sexual harassment, sexual misconduct, and gender
discrimination by any member of our community. This includes harassment among
students, staff, or faculty. Sexual harassment of a student by a faculty member
or sexual harassment of an employee by a supervisor is particularly serious.
Such conduct may easily create an intimidating, hostile, or offensive
environment.

Illinois Tech encourages anyone experiencing sexual harassment or sexual
misconduct to speak with the Office of Title IX Compliance for information on
support options and the resolution process.

You can report sexual harassment electronically
[here](https://iit.edu/incidentreport), which may be completed anonymously. You
may additionally report by contacting the Title IX Coordinator, Virginia Foster
at foster@iit.edu or the Deputy Title IX Coordinator at eespeland@iit.edu.

For confidential support, you may reach Illinois Tech’s Confidential Advisor at
(773) 907-1062. You can also contact a licensed practitioner in Illinois Tech’s
Student Health and Wellness Center at student.health@iit.edu or (312)567-7550

For a comprehensive list of resources regarding counseling services, medical
assistance, legal assistance and visa and immigration services, you can visit
the Office of Title IX Compliance [website](https://www.iit.edu/title-ix/resources).

## Academic Integrity

You are allowed to discuss your work with others in the class, but ultimately
what you submit must be solely yours. Cheating, plagiarism, copying from other
students, or any other sort of academic dishonesty may result in you getting
a zero on the assignment, may impact your grade negatively, and may result in
refurral to the Dean. 

Please see IIT's code of Academic Honesty [here](https://www.iit.edu/student-affairs/student-handbook/fine-print/code-academic-honesty).

## Disability Accommodations
Reasonable accommodations will be made for students with documented
disabilities. In order to receive accommodations, students must obtain a letter
of accommodation from the Center for Disability Resources. The Center for
Disability Resources (CDR) is located in Life Sciences Room 218, telephone (312) 567-5744 or disabilities@iit.edu.

## Sexual Harassment and Discrimination Information
Illinois Tech prohibits all sexual harassment, sexual misconduct, and gender
discrimination by any member of our community. This includes harassment among
students, staff, or faculty. Sexual harassment of a student by a faculty member
or sexual harassment of an employee by a supervisor is particularly serious.
Such conduct may easily create an intimidating, hostile, or offensive
environment.

Illinois Tech encourages anyone experiencing sexual harassment or sexual
misconduct to speak with the Office of Title IX Compliance for information on
support options and the resolution process.

You can report sexual harassment electronically
[here](https://iit.edu/incidentreport), which may be completed anonymously. You
may additionally report by contacting the Title IX Coordinator, Virginia Foster
at foster@iit.edu or the Deputy Title IX Coordinator at eespeland@iit.edu.

For confidential support, you may reach Illinois Tech’s Confidential Advisor at
(773) 907-1062. You can also contact a licensed practitioner in Illinois Tech’s
Student Health and Wellness Center at student.health@iit.edu or (312)567-7550

For a comprehensive list of resources regarding counseling services, medical
assistance, legal assistance and visa and immigration services, you can visit
the Office of Title IX Compliance [website](https://www.iit.edu/title-ix/resources).

## Similar Courses at Other Institutions
- NYU CS (CSCI-GA.3033-15): Virtual Machines: Concepts and Applications
- Northwestern EECS (EECS 441): Resource Virtualization
- UC Berkeley EECS (CS 294-113): Virtual Machines and Managed Runtimes

