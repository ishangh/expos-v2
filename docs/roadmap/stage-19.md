---
title: 'Stage 19 : Exception Handler (6 Hours)'
original_url: https://exposnitc.github.io/Roadmap.html
---

!!! note "Learning Objectives"
    - Familiarize with page fault exception in XSM.
    - Implementation of Exception handler.
    - Modify the exec system call to load code pages of a process on [demand](https://en.wikipedia.org/wiki/Demand_paging).

!!! abstract "Pre-requisite Reading"
    It is absolutely necessary to have clear understanding about [Exception handling in XSM](../tutorials/xsm-interrupts-tutorial.md#exception-handling-in-xsm) before proceeding further.


This stage introduces you to exception handling in eXpOS. There are four events that result in
generation of an exception in XSM. These events are a) illegal memory access, b) illegal
instruction, c) arithmetic exception and d) page fault. When one of this events occur, the XSM
machine raises an exception and control is transferred to the exception handler. The exception
handler code used in previous stages contains only halt instruction which halts the system in
the case of an exception. Clearly it is inappropriate to halt the system (all the processes are
terminated) for exception occured in one process. In this stage, we implement the exception
handler which takes appropriate action for each exception. The exception handler occupies
**page 2 and 3 in the memory and blocks 15 and 16 in the disk**. See disk and memory organization
[here](../os-implementation.md). There are 4 special registers in XSM
which are used to obtain the cause of the exception and the information related to the
exception. These registers are **EC, EIP, EPN and EMA**. The cause of the exception is
obtained from the value present in the EC register.

Exception handler mechanism gives a facility to resume the execution of the process after the
corresponding exception has been taken care of. It is not always possible to resume the
execution of the process, as some events which cause the exception cannot be corrected. In this
case, the proper action is to halt the process gracefully. For the events 1) illegal memory
access (EC=2) 2) illegal instruction (EC=1) and 3) arithmetic exception (EC=3), the exception
handler just prints the cause of the exception. These cases occur because the last instruction
executed (in the currently running user process) resulted in the corresponding error condition.
As the OS is not reponsible for correcting these conditions (why?), the exception handler halts
the process gracefully and then invokes the scheduler to run other processes.

The **page fault exception** (EC=0) occurs when the last instruction in the currently running application tried to either - 

1. Access/modify data from a legal address within its address space, but the page was set to invalid in the page table or
2. fetch an instruction from a legal address within its address space, whose page table entry is invalid.

In either case, the exception occured not because of any error from the side of the
application, but because the OS had not loaded the page and set the page tables. In such case,
**the exception handler resumes the execution of the process after allocating the required page(s) for the process and attaching the page(s) to the process (by setting page table entries appropriately).  If the faulted page is a code page, the OS needs to load the page from the disk to the newly allocated memory.**

!!! note ""
    But why should the OS not allocate all the pages required for a process when the process is
    initialized by the Exec system call, as we were doing in the previous two stages? The reason
    is that this method of pre-allocation allows fewer concurrent processes to run than with the
    present strategy of "lazy allocation" to be described now. The strategy followed in this
    stage is to start executing a process with just one page of code and two pages of stack
    allocated initially. When the process, during execution, tries to access a page that was not
    loaded, an exception is generated and the execption handler will allocate the required page.
    If the required page is a code page, the page will be transferred from the disk to the
    allocated memory. Since pages are allocated only on demand, memory utilization is better (on
    the average) with this approach.


In previous stage, exec system call allocated 2 memory pages each for the heap and the stack.
It also allocated and loaded all the code pages of the process. We will modify exec to allocate
memory pages for only stack (2 pages).**No memory pages will be allocated to heap.
Consequently, the entries in the page table corresponding to heap are set to invalid. For
code blocks, only a single memory page is allocated and the first code block is loaded into
that memory page.**In previous stage, the job of allocating a new memory page and loading
a code block into that memory page is done by<i>Get Free Page</i>and<i>Disk Load</i>functions respectively. 
Now, we will write new module function **Get Code Page ** in the [memory manager module](../modules/module-02.md) for simultaneously allocating a memory page and loading a code block. 
This function will be invoked from exec to allocate one memory page and load the first code block into 
that memory page. Note that only the first code page entry in the page table is set to valid, while 
remaining 3 entries are set to invalid.


Each process has a data structure called [Per-process Disk map table](../os-design/process-table.md#per-process-disk-map-table). The disk map table stores the disk block numbers corresponding to the 
memory pages used by the process.**Each disk map table has 10 words** of which one is for user area page,
two for heap, four for code and two for stack pages. Remaining one word is unused. Whenever the copy of the memory page of a process is
present in some disk block, that disk block number is stored in the per-process Disk Map Table
entry corresponding to that memory page. This is done to keep track of the disk copy of memory
pages. The SPL constant [DISK_MAP_TABLE](../support-tools/constants.md)
gives the starting address of the Disk Map Table of process with PID as 0. The disk map table for any process is 
obtained by adding PID*10 to DISK_MAP_TABLE.

In this stage we will modify the exec system call to initialize the disk map table for the
newly created process. The code page entries of the process's Disk Map Table are filled with
the disk block numbers of the executable file being loaded from the inode table. Remaining
entries are set to invalid (-1). (In later stages, when we swap out the process to disk, we
will fill the stack and the heap entries with the disk block numbers used for swapping. More
about this will be discussed in later stages).

**TheGet Code Page function takes as input the block number of a single code block,
and loads that block into a memory page.** Code pages are shared by the processes running
the same program. The purpose of this function is to find out if the current code block is
already in use by some other process. This is done by going through the disk map table entries
of all the processes checking for the code block (block number provided as argument). If found,
then the ** Get Code Page**checks if the code block is loaded into a memory page (entry in
the corresponding page table should be valid). If the code block is already present in some
memory page, then Get Code Page function just returns that memory page number. If not, a new
memory page is allocated by invoking the **Get Free Page** function of the [memory manager module](../modules/module-02.md) . This is followed by loading the code block intothe newly 
allocated memory page using the **Disk Load**function of the [device manager module](../modules/module-04.md). The Get Code Page function finally returns the memory page number.

The exception handler first switches to the kernel stack and backs up the register context as
done by any other hardware interrupt routine. The exception handler then uses EC register to
find out the cause of the exception. If the cause of the exception is other than page fault,
exception handler should print the appropriate error message to notify the user about the
termination of the process. As these exceptions cannot be corrected, exception handler must
terminate the process by invoking the **Exit Process** function of [process manager module](../modules/module-01.md) and invoke the scheduler to schedule other processes.


The register EIP saves the logical IP value of the instruction which has raised the
exception. The register EPN stores the logical page number of the address that has caused the
page fault.**Note that eXpOS is designed such that, page fault exception can only occur for
heap and code pages. **Library pages are shared by all processes so they are always present
in the memory. Stack pages are neccessary to run a process and are accessed more frequently. So
both library and stack pages for a process should be present in the memory.
Based on the value present in Exception Page Number (EPN) register, the exception handler finds
out whether page fault has caused for heap or code page. When page fault has occured for heap
page (EPN value 2 or 3), exception handler allocates 2 new memory pages by invoking the
**GetFree Page **function in[memory manager module](../modules/module-02.md). If the page fault has occured for a code page, then the exception handler invokes
the **Get Code Page** function in memory manager module. The page table of the process is
updated to store the page number obtained from Get Code Page or Get Free Page functions. After
handling the page fault exception, the exception handler restores the register context,
switches to user stack and returns to user mode.

!!! note 
    When page fault occurs for one heap page, the current eXpOS 
    design allocates two pages for the heap.  This can be optimized further to
    make the allocation lazier by allocating just one heap page and deferring 
    allocation of a second page till a page fault occurs again for the second page. 
    However, the lazier strategy causes some complications in the implementation of 
    the Fork system call in the next stage.  Here, we have chosen to keep the design simple
    by allocating both the heap pages when only one is demanded.

!!! tip ""
    Upon return to user mode, the instruction in the application that caused the
    exception must be re-executed. This indeed is the correct execution semantics as the machine
    had failed to execute the instruction that generated the
    exception. The XSM hardware sets the address of the instruction in 
    the EIP register at the time of entering the exception.After completing the actions of the exception handler,
    the OS must place this address on the top of the application program's stack
    before returning control back to user mode.An OS can implement **Demand Paging**, as we will be doing here, only if the underlying
    machine hardware supports re-execution of the instruction that caused a page fault.


The **Free Page Table** function of the [process manager module](../modules/module-01.md)
decrements the memory reference count (in the[memory free list](../os-design/mem-ds.md#memory-free-list) ) of the memory pages acquired by a process. If some stack/heap page is swapped in the disk, the reference count of the corresponding disk block is decremented in the [disk free list](../os-design/disk-ds.md#disk-free-list). Note that in the present stage, we allocate the stack/heap pages of a process
in memory and never allocate any disk block to store stack/heap pages. Thus, the disk free list
decrement is a vaccous step in the present stage. However this will be useful for later stages.
Hence we design the module function in advance to meet the future requirements. The following
is a brief explanation on why this step can be useful later.

As already seen in Stage 2, eXpOS maintains the [disk free list](../os-design/disk-ds.md#disk-free-list)to keep track of disk block allocation.**In later
stages, the OS will allocate certain disk blocks to a process temporarily.
This is done to swap out the heap/stack pages of a process when the OS finds shortage of
memory space to run all the processes.**
If a heap/stack page of a process is swapped out
into some disk block, the page can be released to some other process. In such cases, the page
table entry for the swapped out page will be set to invalid, but the entry corresponding to the
page in the [disk map table](../os-design/process-table.md#per-process-disk-map-table)will contain the disk block number to which the page has been swapped out. The
disk free list entry for the block will be greater than zero as the block is no longer free.
(It can happen that multiple processes share the block. The disk free list entry for the block
will indicate the count of the number of processes sharing the disk block.)


**When the page table entries of a process are invalidated using the Free Page Table
function of the process manager module,**(either when a process exits or when the exec
system call replaces the current process with a new one)** it is necessary to ensure that any
temporary disk blocks allocated to the process are also released. **Hence the free page
table function checks whether the disk map table entry of a stack/heap page contains a valid
disk block number, and if so decrements its disk free list entry by invoking the
** Release Block** function of the memory manager module.

<figure>
<img src="../../assets/img/roadmap/exec3.png"/>
<figcaption> Control flow for <i>Exec </i>system call</figcaption>
</figure>


#### Modifications of [exec system call](../os-design/exec.md)

1. Don't allocate memory pages for heap. Instead, invalidate page table entries for heap.
2. Change the page allocation for code pages from previous stage. Invoke the **Get Code Page** function for the first code block and update the page table entry for this first code page.Invalidate rest of the code pages entries in the page table.
3. Initialize the disk map table of the process. The code page entries are set to the disk block numbers from inode table of the program (program given as argument to exec). Initialize rest of the entries to -1.

With these modifications, You have completed the final implementation of Exec system call. The full algorithm is provided [here](../os-design/exec.md).

#### Get Code Page (function number = 5,[memorymanager module](../modules/module-02.md))

1. Check the [disk map table](../os-design/process-table.md#per-process-disk-map-table)entries of**all the processes**, if the given block number is present in any entry and the corresponding page table entry is valid then return the memory page number.Also increment the memory free list entry of that page. Memory Free list entry is incrementedas page is being shared by another process.
2. If the code page is not in memory, then invoke **Get Free Page** function in the [memory manager module](../modules/module-02.md)to allocate a new page.
3. Load the disk block to the newly acquired memory page by invoking the **Disk Load** function of the [device manager module](../modules/module-04.md).
4. Return the memory page number to which the code block has been loaded.


#### Modification to the Free Page Table (function number = 4,[process manager module](../modules/module-01.md))

1. Go through the heap and stack entries in the disk map table of the process with given PID. If any valid entries are found, invoke the **Release Block**function in the [memory manager module](../modules/module-02.md).
2. Invalidate all the entries of the disk map table.


#### Release Block (function number = 4,[Memory Manager Module](../modules/module-02.md))

1. Decrement the count of the disk block number in the memory copy of the Disk Free List.
2. Return to the caller.

!!! note 
    **Get Code Page**, **Free Page Table** and **Release Block** functions implemented above are final versions. They will not require modification in later stages.

#### Implementation of [Exception Handler](../os-design/exe-handler.md)
<figure>
<img src="../../assets/img/roadmap/exception.png"/>
<figcaption>Control flow for Exception handler</figcaption>
</figure>

1. Set the MODE FLAG to -1 in the [process table](../os-design/process-table.md) of the current process, indicating in exception handler.
2. Switch to the kernel stack and backup the register context and push EIP onto the stack.
3. If the cause of the exception is other than page fault (EC is not equal to 0) or if the user stack is full (when userSP is PTLR*512-1, the return address can't be pushed onto the stack), then print a meaningful error message. Then invoke the **Exit Process**
function to halt the process and invoke the scheduler.
4. If page fault is caused due to a code page, then get the code block number to be loaded from the [disk map table](../os-design/process-table.md#per-process-disk-map-table). For this block, invoke the**Get Code Page**function present in the
[memory manager module](../modules/module-02.md). Update the page table entry for this code page,set the page number to memory page obtained from**Get Code Page**function and auxiliary information to "1100".<br/>

5. If page fault is caused due to a heap page, then invoke the Get Free Page function of the [memory manager module](../modules/module-02.md)twice to allocate two memory pages for the heap. Update the page table entry for these heap pages, set the page numbers to the memory pages obtained from Get Free Page function and set auxiliary information to "1110".
6. Reset the MODE FLAG to 0. Pop EIP from the stack and restore the register context.
7. Change to the user stack. Increment the stack pointer, store the EIP value onto the location pointed to by SP and return to the user mode. (Address translations needs to be done on the SP to find the stack address to which EIP is to be stored)

The Exception handler implementation given above is final. The full algorithm is given [here](../os-design/exe-handler.md).

#### Modification to the Boot Module

Initialize the disk map table entries for the INIT process. Load the Disk Free List from the
disk block 2 to the memory page 61. (See disk and memory organization [here](../os-implementation.md).)

  
#### Making things work

Compile and load the modified and newly written files into the disk using the XFS-interface.

??? question "Q1. Does EPN always equal to the logical page number of EIP?"
    No. Page fault can occur in two situations. One possibility is during insturction fetch - if the instruction pointer points to an invalid page. In this case, the missing virtual page number (EPN) corresponds to the logical page number of the EIP. The second possibility is during instruction execution when an operand fetch/memory write accesses a page that is not loaded. In this case EPN will indicate the page number of the missing page, and not the logical page number corresponding to EIP value.

??? question "Q2. Why does the exception handler terminate the process when the userSP value is PTLR*512-1 ?"
    The XSM machine doesn't push the return address into the user stack when the exception occurs, instead it stores the address in the EIP register. Hence, for the exception handler to return to the instruction which caused the exception, the EIP register value must be pushed onto the top of the user stack of the program. However, when the application's stack is full (userSP = PTLR*512-1), there is no stack space left to place the return address and the only sensible action for the OS is to terminate the process.

??? question "Q3. Why does the exception handler save the contents of the EIP register immediately into the kernel stack upon entry into the exception handler?"
    The execption handler may block for a disk read and invoke the scheduler during it's course of execution. The value of the EIP register must be stored before scheduling other processes as the current value will be overwritten by the machine if an exception occurs in another application that is scheduled in this way.

!!! assignment "Assignment 1"
    Write an ExpL program to implement a linked list. Your program should first read an integer N, then read N intergers from console and store them in the linked list and print the linked list to the console. Run this program using shell version-I of stage 17.

!!! assignment "Assignment 2"
    Use the [XSM debugger](../support-tools/xsm-simulator.md) to dump the contents of the Exception Flag registers upon entry into the Exception Handler. Also, print out the contents of the Disk Map Table and the Page Table after the Get Code Page function (inside the Memory Manager module).