<h1 align="center">Memory Management</h1>

##  :beginner: Overview
* This section is an attempt to understanding **Memory Management** in **Xv6**.
* We will be implementing **lazy allocation** of heap memory, as most operating systems do, in **Xv6**.
* We will also implement the mechanism required for **swapping in** and **swapping out** of pages, an important feature lacking **Xv6**.
##  :round_pushpin: Basic Understanding of Memory Management in Xv6
* **Sbrk** is the system call for a process to shrink or grow its memory.
*  The **sbrk(n)** system call grows the process's memory size by **n** bytes, and then returns the start of the newly allocated region (i.e., the old size).
```C
int sys_sbrk(void)
{
  int addr;
  int n;

  if (argint(0, &n) < 0)
    return -1;
  addr = myproc()->sz;
  if (growproc(n) < 0)
    return -1;
  return addr;
}
```
* If n is postive, growproc(n) allocates one or more physical pages and maps them at the top of the process’s address space.
* If n is negative, growproc(n) unmaps one or more pages from the process’s address space and frees the corresponding physical pages.
```C
int
growproc(int n)
{
  uint sz;
  struct proc *curproc = myproc();

  sz = curproc->sz;
  if(n > 0){
    if((sz = allocuvm(curproc->pgdir, sz, sz + n)) == 0)
      return -1;
  } else if(n < 0){
    if((sz = deallocuvm(curproc->pgdir, sz, sz + n)) == 0)
      return -1;
  }
  curproc->sz = sz;
  switchuvm(curproc);
  return 0;
}
```
* 
