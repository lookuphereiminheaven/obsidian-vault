### 1. Process Concept & Process States (5-State/7-State Model)

**1. Historical origin & design rationale**  
1960s batch systems (IBM OS/360) ran one job at a time; 1970s multiprogramming (Multics, THE, Unix V6 1975) introduced processes to interleave CPU/memory for interactive use + efficiency. 5-state model (Dijsktra 1968, solved resident set thrashing in memory-constrained systems <64KB). Extended to 7-state in Unix (ready/suspended-ready, blocked/suspended-blocked) for swap-out in 1970s VAX/VMS to handle >physical RAM.

**2. Exact low-level mechanics**  
**Linux 6.11**: `task_struct::state` (volatile long, SLAB-allocated in init_task).  
States:  
| Value | Macro | Meaning |  
|-------|--------|---------|  
| 0 | TASK_RUNNING | Runnable or running |  
| 1 | TASK_INTERRUPTIBLE | Blocked, wakeable by signal |  
| 2 | TASK_UNINTERRUPTIBLE | Blocked, ignore signals (I/O) |  
| 4 | __TASK_STOPPED | SIGSTOP/TSTP |  
| 8 | __TASK_TRACED | ptrace attach |  
| 32 | EXIT_ZOMBIE | Exit complete, waitpid pending |  
| 64 | TASK_DEAD | Final, refcount=0 → free |  
Transition: `set_task_state(tsk, state)` atomically writes via cmpxchg. 5-state abstraction: New→Ready (fork), Ready↔Running (schedule), Running→Blocked (wait_event), Blocked→Ready (wake_up), Running→Exit (do_exit).  
**Windows**: EPROCESS::State (0=Active, 5=Terminated); no explicit 7-state, but ActiveProcessLinks for ready list.  

**3. Security failure modes + exploitation**  
State races → UAF if TASK_DEAD freed prematurely but wake_up races insert to runqueue. Primitive: concurrent exit + signal. CVE-2025-40024 (vhost): early worker exit → vhost_task_wake UAF on task_struct (chain to cred steal). Exploit: trigger exit, race wake → read/write freed PCB → pid/cred overwrite for priv esc. Advantage: /proc/<pid> infoleak exposes state for timing.

### 2. Process Control Block (PCB / task_struct / EPROCESS) Layout and Fields

**1. Historical origin & design rationale**  
Unix V1 (1971): per-process u-area (512B) for regs/context; evolved to PCB for MMU isolation (VAX 1979). Solved context switch overhead in multiprogramming; fields track ownership (files, mem) to prevent leaks in fork/exit.

**2. Exact low-level mechanics**  
**Linux task_struct (~8-16KB, kmem_cache_alloc)**:  
```c
struct task_struct {
  // Identity
  pid_t pid, tgid;
  struct pid_link pids[PIDTYPE_MAX + 1];
  // State
  volatile long state;
  int exit_state;
  // Execution
  void *stack;  // kernel_stack (16KB THREAD_SIZE)
  struct pt_regs *regs;  // user_regs in exception frame
  // Memory
  struct mm_struct *mm;
  // Credentials
  struct cred *real_cred, *cred;  // refcounted
  kuid_t uid;  // effective uid
  // Resources
  struct files_struct *files;
  struct fs_struct *fs;
  struct nsproxy *nsproxy;
  // Scheduling
  struct sched_entity se;  // cfs
  int prio;
  // Signals
  struct signal_struct *signal;
  // Wait
  struct list_head children, sibling;
  // ...
};
```
Layout: first cacheline pid/state/stack for fast access.  
**Windows EPROCESS** (opaque, ~0x6f0 bytes Win11): offsets (WinDbg): 0x2e8 UniqueProcessId, 0x440 Token (exploitable), 0x4b0 ActiveProcessLinks, 0x2f0 ImageFileName.  

**3. Security failure modes + exploitation**  
UAF from premature PCB reuse: alloc → insert runqueue → exit/free → reuse as unrelated task. Primitive: high-load fork/exit spray. CVE-2025-21756 (vsock): binding preserved post-destruction → UAF task_struct → mm/cred corrupt. Exploit: spray PCBs, trigger UAF → arbitrary read (stack pivot via regs) → KASLR bypass → token swap. /proc/<pid>/status leaks fields for offset confirmation.

### 3. Process Creation (fork()/CreateProcess() – exact copy-on-write mechanics, credential copying)

**1. Historical origin & design rationale**  
Unix V6 (1975): fork for shell piping; COW added BSD 4.3 (1986) to reduce 100KB+ copy cost on VAX. Solved explosive memory use in recursive shells.

**2. Exact low-level mechanics**  
**Linux copy_process()**:  
- alloc_pid (pidmap bitset, sequential 1..pid_max=4194304)  
- dup_cred: child->cred = get_cred(parent->cred); child->real_cred = child->cred (ref++=2)  
- copy_mm: for_each_vma → dup_pmd_range (COW PTE: _PAGE_COW bit, wp_page_copy on write)  
- Code flow: clone → copy_process → wake_up_new_task  
Snippet:  
```c
p = copy_process(...);
if (!IS_ERR(p))
  wake_up_new_task(p);
```
**Windows NtCreateProcess**: PsCreateProcess → duplicate handles, inherit token (ObDuplicateObject). No COW, full page table copy.  

**3. Security failure modes + exploitation**  
COW races: concurrent write + fork/madvise. Classic CVE-2016-5195 DirtyCOW: gup_fast COW handling race → write RO mmapped file. Modern variant: CVE-2022-1015 (netfilter, indirect). Primitive: fork child, parent madvise(DONTNEED), race wp_page_copy. Exploit: overwrite /etc/passwd → root shell. Credential manip: race dup_cred before setuid → child inherits root cred.

### 4. Context Switching (register save/restore, kernel stack switching, TLB shootdown implications)

**1. Historical origin & design rationale**  
Multics (1969): timer interrupt switch; Unix V7 (1979) added voluntary (yield) for fairness. Solved CPU hogging in interactive systems.

**2. Exact low-level mechanics**  
**Linux __schedule() → context_switch()**:  
- prev = current; next = pick_next_task()  
- preempt_disable()  
- __switch_to(prev, next): asm (arch/x86/kernel/switch_to.S)  
Snippet:  
```asm
switch_to:
  mov %rsp, TASK_thread_info(prev)+THREAD_info_rsp
  mov TASK_thread_info(next)+THREAD_info_rsp, %rsp
  /* save user regs in pt_regs */
  ret
```
- Stack switch: prev->stack → next->stack (8KB irqstack + 8KB softirq)  
- TLB: if (prev->mm != next->mm) flush_tlb_mm(next->mm) (IPIs for SMP)  
**Windows KiSwapContext**: save to KTRAP_FRAME, load next EPROCESS context.  

**3. Security failure modes + exploitation**  
Race windows: scheduler lock missing → concurrent __switch_to UAF. Primitive: high-freq syscalls + timer. CVE-2021-22555 (netfilter UAF via scheduler race). Exploit: corrupt prev->stack during switch → ROP chain in kernel stack → priv esc. TLB shootdown DoS (CVE-2018-5390 retpoline).

### 5. Process Termination (exit(), resource leaks, zombie/orphan states)

**1. Historical origin & design rationale**  
Unix V6 exit(1975): release resources to parent via wait. Zombie for status delivery; orphan reparent to init (1979) prevents leak.

**2. Exact low-level mechanics**  
**Linux do_exit(int code)**:  
- set TASK_DEAD  
- exit_mm: if (mm) mmput(mm)  
- exit_files: put_files_struct(files) (fput each fd)  
- exit_fs: put_fs_struct(fs)  
- exit_notify: EXIT_ZOMBIE, wake parent wait  
- schedule() never returns  
Orphan: if parent dead, reparent to pid1 (init).  
**Windows ZwTerminateProcess**: ObDereferenceObject(EPROCESS).  

**3. Security failure modes + exploitation**  
Double-free in exit paths (refcount underflow). Primitive: dup fd → exit race fput. CVE-2023-0386 (io_uring UAF on exit). Exploit: double-free files_struct → heap feng shui → arbitrary alloc → cred overwrite. Zombie leak: un-waited → pid exhaust (DoS).

### 6. Process Scheduling Queues (runqueue, wait queues, scheduler classes, CFS)

**1. Historical origin & design rationale**  
Unix V6 round-robin (1975); Linux O(1) 2002 → CFS 2007 for low-latency desktop (vruntime fairness).

**2. Exact low-level mechanics**  
CFS: per-cpu rb_root_cached (min-vruntime heap).  
```c
struct cfs_rq {
  struct rb_root_cached tasks_timeline;
  u64 min_vruntime, max_vruntime;
};
```
enqueue_task_fair: rb_insert (se->vruntime = cfs->min_vruntime)  
pick_next: rb_leftmost  
Wait queues: wait_queue_head_t (list_head), add_wait_queue → wake_up (try_to_wake_up → ttwu_queue_remote if !on_rq). Classes: fair, rt, dl, idle.  

**3. Security failure modes + exploitation**  
Race in enqueue/dequeue: UAF if task freed mid-insert. Primitive: rt throttle + CFS race. CVE-2020-14386 (timer UAF via scheduler). Exploit: corrupt se → infinite loop DoS or vruntime manip → starve root tasks.

### 7. Interprocess Communication (shared memory, message passing, pipes, signals)

**1. Historical origin & design rationale**  
Unix pipes V3 (1973) for shell; shm SysV 1983 for IPC speed; signals 1970s for async.

**2. Exact low-level mechanics**  
Pipes: anon_inode_pipe → pipe_inode_info (2 f_ops: read/write ringbuf).  
Shared mem: shm_file → vma->vm_file.  
Signals: do_signal → sigaction → handler (SA_SIGINFO frame). Message passing: unix sockets.  
Waitqueues for futex/shm sync.  

**3. Security failure modes + exploitation**  
Signal races: sigaction TOCTOU. SROP primitive: craft sigframe (rt_sigreturn) → set regs (rip=rsp=kernel gadget). CVE-2019-13648 (powerpc sigreturn OOB). Exploit: stack overflow → SROP chain → execve(/bin/sh). Pipe UAF (CVE-2024-0775).

### 8. exec() family (image replacement, LD_PRELOAD tricks, shebang attacks)

**1. Historical origin & design rationale**  
Unix V6 exec (1975): overlay for commands without fork overhead.

**2. Exact low-level mechanics**  
**Linux execve**: bprm_execve → search_binary → flush_old_exec (exit_mm, exit_files partial) → load_elf_binary (map phdrs, set mm->start_code) → compute_creds (if setuid, cred swap). LD_PRELOAD: env var → dlopen in dynamic linker. Shebang: /proc/pid/exe → #!/bin/interp arg0.  

**3. Security failure modes + exploitation**  
TOCTOU in binary creds check. Primitive: replace setuid binary post-open pre-exec. CVE-2024-30051 (exec TOCTOU). LD_PRELOAD: if env not cleared in suid, load rogue lib. Exploit: overwrite ld.so.preload → root shell.

### 9. Zombie/Defunct Processes and wait()/waitpid() Races

**1. Historical origin & design rationale**  
Unix V7: zombie for exit status delivery without loss; wait prevents pid reuse races.

**2. Exact low-level mechanics**  
Exit → EXIT_ZOMBIE, signal->wait_chldexit enqueue. waitpid: do_wait → release_task (if EXIT_ZOMBIE, copy status, put_task_struct). Pid reuse only after wait/free.  

**3. Security failure modes + exploitation**  
TOCTOU PID reuse: exit child → parent delay wait → new process gets pid → signal/kill wrong proc. Primitive: fork bomb + selective wait. No direct kernel CVE, but userland enabler for CVE-2016-8655 (packet socket race). Exploit: race kill(pid) → escalate to sibling privs → infoleak /proc. Advantage: amplifies UAF by predictable pid.