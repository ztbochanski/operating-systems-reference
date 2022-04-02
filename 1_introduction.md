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
![Remote Extension](/images/remote_extension.png)
2. setup config with hostname, username, and key path
3. launch a terminal, ssh session, open a file/folder
### Successful Log in
![Remote Extension](/images/o2.png)
## Tools and Tips

## What is an OS?

## Introduction to xv6