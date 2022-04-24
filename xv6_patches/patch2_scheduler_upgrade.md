# xv6 Patch 2 Report: Scheduling
**Multi-Level Feedback Queue and Proportional Scheduling**
## Methodology
>The purpose of this patch is to implement a different scheduling scheme into xv6. The decided design of this scheduling system is to add a multi level feedback queue with a lottery system. This means that features are taken from both types of scheduling methods. 

First, a quick description of what the scheduler does. The scheduler determine when a process gets to use the CPU and gives the process a timeslice to run on. When the process traps or the timeslice is over and triggers a trap then the schedular gets to run on the CPU and take the next process of the queue. An overview of multilevel feedback queue:
- there are multiple queues where each queue represents a different priority level
- a job that uses an entire timeslice gets a lower priority
- a job that gives up CPU time gets a higher priority (higher queue)
- after a certain amount of time that job is moved to the highest priority.

For proportional share scheduling, which also is implemented, alongside MLFQ features is optimized for each process receiving a fair share of the CPU. More specifically proportional share scheduling as an early implementation is called lottery scheduling. The key with lottery scheduling is to have user processes use alloted tickets to determine their access to the CPU when the ticket is randomly selected. The randomly selected winner's ticket value is used to loop through a list and check the ticket value.

This experiment involves building known system calls followed by adding queues to the existing scheduler function and then add random selection of a process's ticket to determine it's access to the CPU.

## Required Changes to xv6

For this implementation of the MLFQ with lottery features there are two know system calls that must be implemented.
Following the implementation of the two system calls then the business logic for assigning procs to queues and then selecting the proc based on a randomly selected ticket value is implemented.

Right off the bat it's known that variables must be added to the process's state to keep track of this information.
```c
// Per-process state
struct proc {
  ...
  int priority;
  int tickets;
  ...
```

Addition of the header file that is required for the implementation. Process stats that are passed to the user space.

```c
struct pstat{
    int inuse[NPROC];
    int pid[NPROC]; 
    int hticks[NPROC]; 
    int lticks[NPROC];
};
```




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
Notice that we chose to call `uptime()` **400x** and verified the correct number was incremented.
![test_syscallcount](https://github.com/ztbochanski/operating-systems-reference/raw/main/images/test_syscallcount.png)
