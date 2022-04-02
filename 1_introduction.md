# Operating Systems: Reference 1

## Overview
This document covers foundational concepts of operating systems:
- ssh
- xv6
- history 
- os concepts
- exploration of linux

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
## Tools and Tips
1. install GDB (The GNU debugger project), see what's going on inside of an executing program
2. install `tmux` -> terminal multiplexer, a cool way to make the shell more useful
## What is an OS?

## Introduction to xv6