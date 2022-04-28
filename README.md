# operating-systems-reference
## Overview
This document covers foundational concepts of operating systems:
- ssh
- xv6
- history 
- os concepts

## SSH: secure shell
`ssh` is a protocol that allows two computers to communicate. The communication is encrypted.

### Terminal

1. open a session with hostname and username, `-J` jump to flip
```zsh
ssh os2.engr.oregonstate.edu -J flip.engr.oregonstate.edu -l <user_name>
```
2. enter password and 2FA when prompted
3. ensure an ssh key is available in the appropriate file `~ .ssh`
### VS code

1. create a config file inside the `ssh - Remote` extension
![extension](https://github.com/ztbochanski/operating-systems-reference/raw/main/images/remote_extension.png)
1. setup config with hostname, username, and key path
```zsh
Host flip1
    Hostname flip1.engr.oregonstate.edu
    
Host os2
    Hostname os2.engr.oregonstate.edu
    ProxyJump flip1

Host *
    ForwardX11Trusted yes
    User bochansz
    IdentityFile /Users/<user>/.ssh/id_ed25519
```
3. launch a terminal, ssh session, open a file/folder
### Successful Log in
![remote_machine](https://github.com/ztbochanski/operating-systems-reference/raw/main/images/o2.png)
## Tools
1. install GDB (The GNU debugger project), see what's going on inside of an executing program
2. install `tmux` -> terminal multiplexer, a cool way to make the shell more useful
3. strive for mouseless navigation and operation!

## OS History
- 1960s: multiprogramming, where different programs required their own memory an (OS)  to manage them.
- These early operating systems used segmented memory (paging is common today) and had little security
- MULTICS: a multi-user, virtualized system with security features.
- From MULTICS came `UNIX` note the `X` and not `c` for unic, to indicate UNIX was considered a gutted version of MULTICS.
- Many modern OSs are unix-like.

## OS Scheduling System Types
There are multiple ways to classify operating systems, how jobs are scheduled is one way to categorize this.
1. **Batch Systems**
   First come first serve, queue type system where jobs are executed in this fashion
2. **Real Time Systems (RTOSes)**
   Real time, microcontroller level of control is needed with the added operating system benefits. Examples are systems used in:
   - cars
   - rockets
   - extra-terrestrial equipment 
   1. **Hard Real Time**
      All deadlines for processes are met, applications where safety is critical. Synced to CPU cycle.
   2. **Soft Real Time**
      There is flexibility with scheduling, system calls are allowed to take more time within a specified range.
3. **General Purpose Systems**
   These are the common user systems, no real time constraints or scheduling restraints.

## Kernel Variants
Another way to classify an OS is by the style of kernel used.
1. **Microkernels**
   The core of the OS is the "micro-kernel", resilient against bugs, each process is a component and can be rerun if it fails. Uses message passing.
2. **Monolithic Kernels**
   Does not need to pass messages among components which is much faster but the whole kernel can crash if one part of it fails.
3. **Hybrid Kernels**
   The core of the OS can act as a monolithic kernel, while the drivers and utilities can act alone.

## Kernel Programming
Things to be aware of when writing code inside the kernel:
- per-process stack space is small
- no C library access
- floating point require too much overhead
- NO RECURSION
- kernel must be portable

## Perspectives of an Operating System

1. **OS as an abstract machine**
   Drivers allow a user perform tasks like writing to an external source without requiring the user to understand how. Abstraction.
2. **OS as a resource manager**
   In this view the OS manages all of the individual pieces like I/O, hardware, etc. and wraps it up in a unified, single system.

## 3 concepts of an OS
1. Virtualization
2. Concurrency
3. Persistence

## QEMU: machine emulation

![qemu](https://github.com/ztbochanski/operating-systems-reference/raw/main/images/qemu.png)
