<h1 align="center">Thread and Synchronization</h1>

##  :beginner: Overview
* This section is an attempt to implement **Kernel level Threads** and **Spinlocks** for it's synchronization and correctness. <br>
* Threads ar very similar to processes. They allow for parallel execution and share same address space. <br>
* Threads of a process have different stack, registers, program counters and state. <br>
* Locks are a common way to ensure that access(read/write) to shared data occurs appropriately and correctly. <br>

## :thread: Thread Implementation
To implement threads, I made use of the codes provided for **fork**, **wait**, **exit** and modified them to create threads, rather than processes.
* ### thread_create function
```C
int thread_create(void(*fcn)(void*), void *arg, void*stack)
{
  int i, pid;
  struct proc *np;
  struct proc *curproc = myproc();

  // Allocate process.
  if((np = allocproc()) == 0){
    return -1;
  }

  // Page Table is same.
  np->pgdir = curproc->pgdir;   

  np->sz = curproc->sz;
  np->parent = curproc;
  *np->tf = *curproc->tf;

  // Clear %eax so that fork returns 0 in the child.
  np->tf->eax = 0;

  // Stack Size is of 1 page
  np->tf->esp = (uint)stack + 4096;

  // Load Arguments in Stack
  np->tf->esp -= 4;
  *((uint*)(np->tf->esp)) = (uint)arg;

  // Save Return Address in stack
  np->tf->esp -= 4;
  *((uint*)(np->tf->esp)) = 0xffffffff;

  // Set instruction Pointer
  np->tf->eip = (uint) fcn;

  for(i = 0; i < NOFILE; i++)
    if(curproc->ofile[i])
      np->ofile[i] = filedup(curproc->ofile[i]);
  np->cwd = idup(curproc->cwd);

  safestrcpy(np->name, curproc->name, sizeof(curproc->name));

  pid = np->pid;

  acquire(&ptable.lock);

  np->state = RUNNABLE;

  release(&ptable.lock);
```
