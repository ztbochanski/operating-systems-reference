# xv6 Patch 1 Report

## Methodology
To implement a new system call in xv6, it seems the simplest approach is to recognize the patterns that the other system calls  follow in xv6. The document outlines how a new system call `getsyscallinfo` is implemented. For this implementation, `getpid()`, get process id is used as the model to follow. With the construction of this system call it is important to note where system calls are being implemented as well as where the method signature is being defined. 

The idea of `getsyscallinfo` is to return a total count of system calls that are made while the operating system is booted up and running. There are some important behaviors that must be considered:
- Incrementation of system call record right before a new successful system call
- Ability for the new system call to return the count record

## Required Changes to xv6
The following files are changed to add an additional system call to xv6

### Makefile
An administrivia related change, we need to make sure that the makefile knows where to find qemu (which will run xv6 for us).
```c
QEMU = /usr/libexec/qemu-kvm
```

Add the testing file to UPROGS so it can get compiled during make
```c
UPROGS=\
 	_usertests\
 	_wc\
 	_zombie\
	_test_syscallcount\
```

### syscall.c
Global system call function inside the unlink method.
```c
extern int sys_getsyscallinfo(void);
```
Make sure to define the a function pointer for the new system call and not forget to add the index where we want the new system call in the array, which will be defined in `syscall.h`
```c
[SYS_getsyscallinfo] sys_getsyscallinfo,
```
The rest of the main functionality of `syscall` is to get the current system call number and get the correct system call from the array and put that value into `eax`

The reason for all of this is so we can simply add a new system call to an array of functions. This way the function number just relates to SYS_<number>. This is better than the alternative of writing the assembly for each new function. 

### syscall.h
Defining the index for the new system call function so the MACRO knows what index to use. These are linked together in the `syscall.c` function.
```c
#define SYS_getsyscallinfo 22
```

### sysproc.c
This is where the business logic is performed. A new function is created that is void and returns an int which is our count. The global count variable is used. There could potentially be a better way of implementing this to avoid the use of a global. Incrementation of the counter occurs in another file that handles the interrupts (this way we can implement it right before a syscall is made).
```c
int sys_getsyscallinfo(void)
{
  extern int count;
  return count;
}
```

### trap.c
Interrupts are handled here. One thing with xv6 and other operating systems is that a system call is grounds for an interrupt to occur. Therefore the `syscall` function will be called in the kernel’s trap if a system call is made. In the conditional statement right before syscall is called this is where the `count` variable can be incremented.
```c
// define count
int count = 0;

// add incrementer
trap(struct trapframe *tf)
     if(myproc()->killed)
       exit();
     myproc()->tf = tf;
 +   count++;
     syscall();
     if(myproc()->killed)
       exit();
```

### user.h
I’m not 100% sure why we define the signature here. Perhaps something to do with debugging or user access?
```c
int getsyscallinfo(void);
```

### usys.S
Defines the macro for the array of system call functions. When it receives the function it loads the value onto eax and triggers the interrupt which is handled in `trap.c`
```c
SYSCALL(getsyscallinfo)
```
## Testing
To test this implementation the idea is not see if the method returns an arbitrary integer. The questions that must be asked:
1. How can it be certain that count is incremented appropriately?
2. How is it known that the count is called once for each successful new process?
3. Is the location of incrementing in trap.c where new sys calls are made the correct location?

**The test is designed as such:**
1. In the test code, explicitly invoke a known number of system calls.
2. Use the new syscall `getsyscallinfo()` to get an initial and final count of system calls.
3. Add the initial count to the known number of calls and if the sum is the same as the final count then the counter increments system calls correctly.
4. This also verifies that the counter is working properly in `trap.c` and we are in the correct location.

**Code snippets describing the test design:**
1. Executing a known number of user entered calls.
```c
for (index = 0; index < user_triggered_calls - 1; index++)
  {
    uptime();
  }

2. Get initial a final counts
```c
initial_count = getsyscallinfo();

for (index = 0; index < user_triggered_calls - 1; index++)
{
    uptime();
 }

final_count = getsyscallinfo();
```

3. Sum initial and know number, compare to final count
```c
if ((initial_count + user_triggered_calls) == final_count)
```

### TEST OUTPUT
![test_syscallcount]()
