# CS 350 Study Notes

## Topic 2: Threads

Q: What are the three views of an operating system? Describe them.
A:
Application: what can it do (that I care about)?
System: what is it supposed to do?
Implementation: how does it work?

Q: Define the kernel of the OS. What things are considered part of the OS but not the kernel?
A: kernel = part of the operating system that responds to system calls, interrupts and exceptions. OS has the kernel + utilities, shells, libraries, etc.

Q: What are some advantages of allowing more than one thread to exists simultaneously?
A: Can have multiple programs with different outputs/behaviours run at the same time.

Q: What is contained in a thread context? What is the thread control block?
A: CPU state + stack. TCB is a data struct that stores/points to those.

Q: Describe what happens during a context switch? What is dispatching?
A:
Context switch: pause a thread and start another.
Dispatching: save the old thread context (create a switchframe and possibly trapframe), install that of the other.

Q: What is the difference between thread_yield() and thread_sleep()?
A:
Yield: Yields voluntarily, can run immediately (ADDED TO THE QUEUE for scheduling).
Sleep: Blocked on something, must wait until awaken (for a resource to become available or some event to finish) (NOT ADDED TO THE QUEUE for scheduling).

Q: How is an involuntary context switch accomplished? Define quantum.
A: Timer interrupts after a thread runs for some period of time (aka. quantum). The interrupt forces the thread into kernel mode, go to the exception handling function, etc.

Q: Describe what happens when an interrupt occurs. Where do interrupts come from?
A: From devices/hardware. See below for details.

## Topic 3: Sync

Q: What are the three properties of a good implementation of synchronization?
A: fair, efficient, correct

Q: Can the OS know when a program is in a critical section?
A: When it can't be interrupted or if it holds a lock (not sure if could be done programmatically tho)

Q: When is it appropriate to disable interrupts? What happens when you disable interrupts on a multiprocessor? Give pros and cons of disabling interrupts.
A: For critical sections (code that you don’t want multiple threads to run concurrently). Con: doesn't work for multicore CPUs. Pro: doesn't need special hardware instructions.

Q: How does Test and Set work? How would you use it in a program? What is a potential problem with Test and Set?
A: sets a new value, returns old value. used for mutex (spinlock implementation). Starvation.

Q: What is a semaphore? Which two operations does it support? Are they atomic?
A: Data structure used to solve mutex problems. P() (decrement iff > 0) and V()(increment).

Q: Is the implementation of semaphores given in the notes starvation free? Why or why not?
A: Yes because this implementation of wait-channel is FIFO. In general, it’s not.

Q: Describe, at a high level, what a monitor does. How can we simulate their behaviour in C?
A: Lock + CV.

Q: What’s the difference between Mesa-style and Hoare-style semantics?
A: Mesa (os161) - signaler keeps lock. Hoare - signaler passes lock.

Q: Define deadlock. How does the system recover from a deadlock?
A: No thread can make progress because they’re waiting on each other. Kill a thread.

Q: If there is a cycle in a resource allocation graph, is there a deadlock? What about the converse of this?
A: Yes, yes.

Q: Give the deadlock detection algorithm, and describe the three deadlock prevention methods discussed in class.
A:
No hold and and wait - acquire all at once
Preemption - take resources away from a thread (not realistic)
Resource ordering - order the resources in a way that makes acquisition acyclic (can only request larger resources, e.g. larger account #s)

## Topic 4: Processes

Q: What information is contained within a process?
A: address space, threads, pid, current working directory, other resources

Q: What is the difference between a sequential process and a concurrent process?
A: one thread vs many threads

Q: Can threads from one process access memory from another process?
A: Not directly.

Q: What can the kernel do in privileged execution mode that user programs can not directly accomplish?
A: Control hardware, special instructions (HALT), access special memory addresses, manipulate processor

Q: Describe what happens when a system call is invoked.
A:
Switch to privileged execution mode
Save thread context in trapframe
Change PC to a pre-determined memory address. Kernel determines which syscall it is and does stuff

Q: In which registers does a system call expect the system call code? Arguments?
A: v0 = syscall code; a0-3 = sycall args

Q: In which registers will the system call place the return result? Return value?
A: a3 = result (0 if ok); v0 = return value/error # (just remember the “code” is generally in v0)

Q: What are the only three ways for the CPU to enter privileged mode?
A: syscall, interrupt, exception
Syscall: program wants to do something “special” that requires kernel assistance (like file I/O)
Interrupt: hardware does/finishes doing something (like finished writing file, timers)
Exception: something unusual happened, needs to be rescued/killed by kernel (divide by 0, page faults)

Q: Why does OS/161 have two stacks?
A: One for kernel mode and one for non-kernel mode

Q: The mips trap function is called to handle system calls, exceptions, and interrupts. How does it differentiate between these three cases?
A: tf->tf_cause determines which case it is.

Q: When returning from a system call, the program counter is not automatically advanced by the hardware. Why is this?
A: Because it's done by the kernel.

Q: What might an OS want to keep track of in a process table?
A: PID, owner, address space, threads, files/other resources, cwd, runtime and metadata

Q: Define timesharing. When can the CPU context switch to another thread?
A: Share a CPU between multiple processes. During syscall/interrupt/exceptions.

## Topic 5a: VM pre-midterm

Q: A system uses physical address and virtual addresses. How many physical address spaces are there in a computer? How many virtual address spaces?
A: | Physical | = | RAM | = | Virtual |
There’s only one physical address space per machine, and one virtual address space per process.

Q: What is a simple method of address translation? How does this work? What are its disadvantages?
A: Dynamic relocation. Check upper bound, then add offset. External fragmentation.

Q: In dynamic relocation, when does the value of the relocation register in the MMU change?
A: During context switch.

Q: What is the difference between external fragmentation and internal fragmentation?
A:
External: wasted space between allocated memory
Internal: wasted space within allocated memory

Q: What advantages does paging give over dynamic relocation?
A: Reduces external fragmentation b/c space between allocated chunks are divisible by page size, which is convenient b/c we allocate by whole pages

Q: How does the kernel use a page table to help translate virtual addresses to physical addresses?
A: Maps the virtual addr prefix to a physical addr prefix

Q: In address translation, what (4) roles does the OS serve? What (2) roles does the MMU serve?
A:
Kernel:
Manages MMU state on address space switches
Creates and manages page tables
Allocates physical memory
Handles (page fault) exceptions
MMU:
translates virtual → physical address
checks and raises exceptions

Q: What is the TLB? What does it stand for?
A: Translation Lookside Buffer. Fully associative cache of (page #, frame #).

Q: Why might the MMU need to invalidate the TLB? When would this happen?
A: Context switch - the mapping changes between different processes since address spaces are different

Q: What is the difference between a hardware-controlled TLB and a software-controlled TLB? Which is the MIPS TLB?
A: MIPS is the latter.
Hardware: does page table lookups
Software: hardware raises exception, kernel chooses which page to evict

Q: How does segmentation improve address translation? What is the motivation behind segmentation?
A: Reduces external fragmentation. Privileges can be set for segments (like you can’t overwrite the CODE segment).

Q: What are three approaches for locating the kernel’s address space? How do OS/161 and Linux do this?
A:
Kernel in physical space
Kernel in separate virtual addr space
Kernel mapped into portion of address space of every process (used)

Q: Describe the different parts of an ELF file. Why doesn’t the ELF file describe the stack?
A: Stack size hardcoded to 12 pages in MIPS, and changes during runtime. Plus you can’t predict which functions will run even when they’re compiled, b/c halting problem.
rodata - read only data; text - code; data - initialized global data; bss - uninitialized global data; sbss - small bss

Q: What are the different sections in an ELF file? What are the different segments?
A: Those things (rodata, text) are sections.
1st LOAD segment contains .text, .rodata (things you cannot change)
2nd LOAD segment contains data, bss, sbss (things you can change)
segments contain sections.

## Topic 5b: VM post-midterm

Q: Page tables for large address spaces may be very large. One problem is that they must be in memory, and must be physically contiguous. What are two solutions to this problem?
A: multi-level paging, or segmentation + paging

Q: What is the difference between a global page replacement policy and a local page replacement policy?
A: Local - only the faulting process's own frames can be swapped for new pages.
Global - any frame can be swapped.
Pros for local: faster, less frames to inspect. Pros for global: more fair, don’t have to allocate fixed # frames per process beforehand.

Q: What steps must the OS take to handle a page fault exception?
A:
block faulting process
copy missing page into memory
set P=1 in page table entry (P = present bit)
unblock process

Q: What are the 4 different page replacement policies discussed in lecture? Describe and compare them.
A: LRU - efficient, complicated, slow runtime; FIFO - inefficient, fair, simple; OPT - optimal, unrealistic; CLOCK - FIFO, with used bit

Q: What are the two types of locality to consider when designing a page replacement policy?
A: Temporal (pages used more recently are likely to be used again) and spatial (pages close to a recently used page are likely to be used next)

Q: In practice, frequency based page replacement policies don’t work very well. Why is this?
A: Impractical - takes too long to compute or uses too many resources

Q: LRU page replacement is considered impractical in virtual memory systems. Why is this?
A: Requires too many CPU cycles to compute

Q: Why are modified pages more expensive to replace than a clean page?
A: Have to write changes first

Q: Give 3 advantages and 2 disadvantages of large page sizes.
A:
Advantages: smaller (and faster) page table, less TLB misses, more memory swapped in per page fault (better spatial locality)
Disadvantages: more internal fragmentation, slower swap

Q: What is prefetching and what is its goal? What are the hazards of prefetching? Which kind of locality does prefetching try and exploit?
A: Guess which pages will be needed. May guess wrong. Spatial locality (swap in the next X pages).

Q: How large are the pages in OS/161?
A: 4 KB

Q: Describe the Working Set Model, working set, and resident set.
A: Only some pages are useful/heavily used at a given time, wrt a process. Working set = those heavily used set of pages in memory. Resident set = all the pages in the memory.

Q: How can information about a program’s working set be used in a page replacement policy?
A: Don't replace working set pages during page faults.

Q: What is thrashing and what are two solutions to it?
A: Too many page faults (not enough memory, high contention). Kill processes, or suspend & swap them out.

Q: Why is it better to suspend and swap processes than just to have them all run at once?
A: Less page faults, more CPU-cycles spent doing useful work vs mundane memory management. Overall higher thoroughput.

Q: Describe an ideal amount of a process’ working set to remain in memory in terms of suspending processes.
A: 100% of {working} set, 0% of {resident} \ {working} set (aka we keep something iff it is useful)

## Topic 6: Processor Scheduling

Q: What are the 4 scheduling algorithms discussed in lecture? Describe/compare/contrast them.
A:
FCFS: simple, avoid starvation, inefficient. Has preemptive variant called round-robin.
SJF (shortest job first): simple, efficient, may starve. Minimizes average turnaround time. Has preemptive variant called SRTF (remaining time).

Q: Which of those 5 algorithms is provably optimal?
A: SRTF probably, by greedy algorithm?

Q: Describe how a multilevel feedback queue works.
A: Maintain several round-robin queues of various priorities. Always choose from the highest priority, non-empty queue. Lower priority queue get longer quanta. Blocking keeps you at the same queue. Getting preempted sends you to a lower prio queue.

Q: In a prioritized scheduling environment, low-priority threads risk starvation. How can this be solved?
A: Occasionally move everyone to high prio. Or use Linux's completely-fair scheduler.

Q: Describe pseudo code for such a scheduling algorithm which eliminates the risk of starvation.
A: From all the tasks, compute their virtual runtime:priority ratio. Choose the lowest one (low runtime but relatively high priority).

Q: Compare and contrast the pros/cons of using one queue per core vs. one queue for all cores.
A:
Pro of 1 queue per core: less contention, more scalability
Pro of 1 queue for all: easy to load balance (this tradeoff came up during co-op ironically, for a logging request handling app)

## Topic 7: I/O

Q: What are the 3 registers each device maintains? How are they used?
A:
Command: processor communicates to the device
Status: device communicates to processor
Data: data exchanged between them (sometimes separated into data-in and data-out)
(CPU is master, device is slave, in a master-slave model. Or you can think of this wrt to a user process).

Q: Describe how SYS161 uses memory mapping for devices.
A: Those device registers are mapped into physical memory addresses, and accessible to programs via standard memory ops.

Q: What is the difference between program controlled I/O and Direct Memory Access? What are their respective advantages? Which does SYS/161 use?
A:
PC I/O (used): Device driver moves data iteratively. CPU is busy doing boring work. Synchronous and simple.
DMA: CPU is not busy, can do more interesting work. Async (controller interrupts processor when transfer is done).
How does it move the data then? Burst mode (all at once), cycle-stealing (bits at a time), transparent mode (when RAM is not used by CPU) 

Q: OS’s often buffer data that is moving between devices and application programs’ address spaces. What are the benefits/drawbacks of this?
A: Buffering: more efficient - exchange more data per move (e.g. disk writes). May be less responsive.

Q: What is the only way for a user application to talk to a device?
A: Syscalls (probably?)

Q: What does the statement ”disks are non-volatile” mean?
A: They're persistent (data sticks around between restarts/crashes)

Q: Using the simplified cost model for disk block transfer, what are the three components that make up ”request service time”?
A: seek time (time to move the head), transfer time (time to read the sectors), rotational latency (time to get to the right sector)

Q: If a disk spins at x rotations per second, what is the expected rotational latency in terms of x?
A: 1/2x

Q: If there are k sectors to be transferred and there are T sectors per track, what is the transfer time in terms of k, T, and x?
A: k/Tx

Q: If k is the required seek distance, what is the seek time (as a function of k)?
A: (k/max distance) * max seek time

Q: Why is sequential I/O more efficient than non-sequential I/O?
A: Because rotational latency = 0, since sectors are contiguous

Q: What are the 3 disk head scheduling algorithms discussed in lecture? Who is responsible for scheduling the disk head? Describe/compare/contrast them.
A: OS or controller or both.
FCFS: fair, simple, inefficient.
SSTF (shortest seek time first): unfair (starve long requests), simple, efficient.
SCAN: fair, not as simple, efficient.

## Topic 8: FS

Q: What is the definition of a file given in the notes? What is the definition of a filesystem?
A:
File: persistent, named data objects.
FS: collection of files in a common namespace.

Q: Describe the 6-7 operations described in class that an application can perform on a file.
A: open/close, read/write/seek (lseek), get/set metadata (stat/fstat/chmod)

Q: File position may be implicit (ie. not specified on each read/write call). Why is this, and because of this how is non-sequential I/O performed?
A: OS keeps track of the file position. Use seek (lseek) to change it.

Q: What does it mean for a file to be memory mapped? What pros and cons does memory-mapping give?
A: It means the file is mapped to virtual memory. Better IO perf. Uses precious memory.

Q: What is the difference between a hard link and a soft link?
A: Hard links have referential integrity (link exist => file exist). Soft doesn't.

Q: What limitations of hard links do soft links not have?
A: Can't cross FS boundaries.

Q: What problems can occur with the use of soft links?
A: Dangling references.

Q: Why can’t hard links cross file system boundaries?
A: Non-unique i-numbers between different FS.

Q: If we wish to operate with multiple file systems, what are the two most common options in organizing the system (ie. the two options discussed in class)?
A: Windows-style (FS at the front, like C:\, D:\), Unix-style (mount under any directory)

Q: What are the advantages/disadvantages of fixed-size chunk allocation vs. variable-size chunk allocation? Which system is more readily used in modern drives?
A: Internal vs external fragmentation tradeoff (fixed has more internal, variable has more external). Fixed is more popular by far.

Q: How are drives indexed? What does LBA stand for?
A: Logical block addressing - use 1 number to represent 1 block.

Q: What are the 3 file organization methods given in lecture? Describe how they work and their advantages/disadvantages.
A:
One single contiguous segment, best for random access, external fragmentation
Linked list of blocks, best for sequential access, internal fragmentation
A contiguous table of pointers to blocks, good for both, needs defrag
Q: What kind of meta-data is contained in a Unix i-node?
A: owner, size/type date last accessed/modified, date of last i-node update, permissions, # hard links, data pointers
(aka. things you’d see with ls -ali, + other hidden things)

Q: What are advantages of using single/double/triple indirect blocks instead of just direct blocks?
A: Can access more data (blocks) with a fixed, small inode size. Super-efficient for common case (small files), but exponentially scalable to larger sizes as well.

Q: Why is it advantageous to leave large gaps between sections of used space on disk?
A: To improve spatial locality and reduce fragmentation

Q: How are directories implemented?
A: As (special) files that can only be updated by kernel. Contains a list of inumber-filename pairings, essentially.

Q: Describe (at a high level) how to implement hard links and soft links.
A:
Hard links: as directory entries, with file name = whatever, file number = same i-number.
Soft links: as special symlink files, containing a string representing the (probably relative) path.

Q: What does the open() system call have to do differently when handling a symlink file?
A: It has to traverse the filesystem to try and find the file wrt. the path relative to the location of the symlink

Q: What are two ways for a file system to handle/clean up after a failure?
A:
Journaling (log of what was changed, so it can be replayed after failures)
Special-purpose consistency checkers (find/repair inconsistent file data structures, e.g. files w/o directory entries, free space not marked as free, etc.)

Q: How do we find an inode of an inumber log-structured FS?
A: checkpoint-region → most recent imap → most recent inode

Q: Why do we store imap copies after inode copies? Why use checkpoint regions?
A: You gotta keep something at a known location (CR). small imap copies are for efficiency.

## Topic 9: IPC

Q: What are the two general methods that can be used for two processes to share information?
A: Shared storage (i.e. files) and message-based (i.e. sockets)

Q: Describe and compare direct message passing vs. indirect message passing.
A:
Indirect: OS needs to buffer messages
Direct: OS doesn’t

Q: How can message passing mechanisms differ in terms of the following properties: directionality, message boundaries, connections, and reliability.
A:
(phone call) Streams: duplex, boundary-less, connection-oriented, reliable
(mail) Datagrams: duplex, boundary-ful, connectionless, unreliable

Q: What are two address domains discussed in lecture, and how do they differ?
A: Unix (same machine) and INET/Internet (across machines)

Q: Given that the INET address domain can be used for processes running on the same machine and on different machines to communicate, why is the UDS domain still useful?
A: Faster

Q: With datagram sockets, when will the recvfrom() and sendto() functions block?
A:
recvfrom() blocks if there are no messages to be received.
sendto() blocks when the system doesn't have any more space to buffer undelivered messages.

Q: With stream sockets, what is the difference between the passive process and the active process?
A: One passively waits for connections to be made via listen()/accept(), and one actively makes connections via connect()
(think client-server model)

Q: With stream sockets, the active process uses the connect() function to send a connection request. connect() will block until what happens?
A: Until the other socket accepts the connection request.

Q: With stream sockets, if the active process does not choose to bind an address to the socket, what will happen? What is the advantage of this?
A: OS will automatically assign one. It's good because the process does not need to hard-code an address before forming a connection

---

## BONUS questions by me

Q: How would you implement a sync’ed linked list shared between threads without using locks/semaphores (i.e. using CAS)? These are called wait-free data structures, and they do exist.

Q: What happens if you get interrupted during an interrupt? (e.g. at trapframe copying) Can you be interrupted?

Q: How would you efficiently implement a multithreaded system where multiple readers can access a shared resource without blocking, but writing threads block all others (including other writers and readers)? i.e. reader-writer locks

Q: How would you implement a system where pages are lazily (i.e. on demand) allocated to a process?

Q: Suppose seeks times are negligible compared to rotational latency. Design a fair, efficient 
scheduling algorithm to take advantage of this. What if rotational latency is negligible compared to seek times?

Q: Write pseudocode to compute which block a given byte of a file is, given the os161 inode structure. How would it scale if we have 4x-, 5x-, Nx-levels of indirection?

Q: When would a thread ever call thread_yield over blocking (thread_sleep)?

Q: How does a log-structured file system know if a space can be overwritten or not?

Q: Consider a distributed file system where we store data over X disks instead of 1. Design a filesystem that implements this, s.t. it is tolerant to strictly increasing the # of disks (i.e. you can’t remove disks). What would the inode look like?

Q: How would you implement a reliable system over datagram sockets? How would you implement a boundary-ful/connectionless system over stream sockets?

Q: Implement a performant filesystem where files have TTLs or expiry dates (this is a real use case for many logging systems).

Q: How would you implement aio.h (asynchronous file I/O system calls)?

Q: What are the pros/cons of large file blocks? Which ones overlap with the corresponding page size question?

