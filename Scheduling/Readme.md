<h1 align="center">Process Management and Scheduling</h1>

##  :beginner: Overview
* This section is an attempt to understanding **Process Management** and **Scheduling** in **Xv6**.
* Current scheduler in Xv6 is an **unweighted round robin scheduler.**  
* In this section, I modified the scheduler to take into account user-defined process burst time and implemented:
    * **Shortest job first Scheduler**
    * **Hybrid Scheduler**
##  :round_pushpin: Some Prequisites
1) ### getNumProc function
* A system call that returns the total number of **active** processes in the system. This includes process in state :
     * **EMBRYO** (New process is being created)
     * **RUNNABLE** (Process is ready to run)
     * **RUNNING** (Process is executing)
     * **SLEEPING** (Blocked for an I/O)
     * **ZOMBIE** (Finished execution)
```C
int getNumProc(void)
{
  int active = 0;

  acquire(&ptable.lock);

  struct proc *process;

  for (process = ptable.proc; process < &ptable.proc[NPROC]; process++)
  {
    if (process->state != UNUSED)
    {
      active++;
    }
  }

  release(&ptable.lock);
  return active;
}
```
2) ### getMaxPid function
* A system call that returns the maximum **PID** amongst all the PIDs of the currently active processes in the system.
```C
int getMaxPid(void)
{
  int max_active = -1;

  acquire(&ptable.lock);

  struct proc *process;

  for (process = ptable.proc; process < &ptable.proc[NPROC]; process++)
  {
    if (process->state != UNUSED)
    {
      if (max_active < process->pid)
      {
        max_active = process->pid;
      }
    }
  }

  release(&ptable.lock);
  return max_active;
}
```
3) ### getProcInfo function
* A system call that returns 0 and assigns the **processInfo structure** accordingly on finding an entry about a process, else returns -1.
* Below is the definition of **structure processInfo**:
```C
struct processInfo
{
    int ppid;
    int psize;
    int numberContextSwitches;
};
```
* Include **numberContextSwitches** field in **struct proc**:
```C
int numberContextSwitches;   // Number of Context Switches
```
* Below is the definition of the system call
```C
int getProcInfo(int pid, struct processInfo * procInfo)
{
  struct proc *p;
  int idx = 0;
  acquire(&ptable.lock);
  for (p = ptable.proc; p < &ptable.proc[NPROC]; p++)
  {
    if (p->pid == pid)
    {
      procInfo->numberContextSwitches = p->numberContextSwitches;
      procInfo->ppid = p->parent->pid;
      procInfo->psize = p->sz;
      release(&ptable.lock);
      return 0;
    }
    idx++;
  }
  release(&ptable.lock);

  return -1;
}
```
4) ### Burst Time
* Include **burst_time** field in **struct proc**:
```C
int burst_time;              // Burst Time of Process
```
   * **set_burst_time** function call
   ```C
   int set_burst_time(int n)
   {
     struct proc *cur_proc = myproc();
     cur_proc->burst_time = n;
     return 0;
   }
   ```
   * **get_burst_time** function call
   ```C
   int get_burst_time(void)
   {
     struct proc *cur_proc = myproc();
     return cur_proc->burst_time;
   }
   ```
