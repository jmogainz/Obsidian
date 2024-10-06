- How does start_kernel() get set as the entry point in the assembly code? I thought the compiler looked for 'main' specifically
- How does the kernel after startup, get into a state, practically, in code, where it responds to events (system calls and hardware interrupts, page fault exceptions)?
- User mode code cannot access kernel mode code without switching into kernel mode via a system call or interrupt...is this through a trap? And what actually prevents the user mode from accessing the kernel mode memory/code? Is that done by the mmu or higher than that? Segmentation level right? Not paging level?
- Why is it called a trap?
	- You are 'catching' or 'interceding' execution with a privelege escalation and other code execution

# Learned
- Scheduler is the program that continuously runs in the kernel... it seems
	- System calls, interrupt handlers, exceptions seem to be more of library usage
- Trap mechanism uses the trap gate which is defined in the IDT, transfers control to a predefined kernel handler
	- The context of the user-space process is saved so it can be restored later
		- I dont think this is a true context switch though that would occur when a thread switches betweens processes...
		-  **Modern CPUs are designed to handle traps** as part of their architecture. They come equipped with mechanisms to support system calls and interrupts by having dedicated **instructions** (like `syscall` or `int 0x80` on x86) and **trap handling routines** built into their design.
- This event-driven model means the kernel doesn’t need an explicit **event loop** like you would see in user-space event-driven systems. Instead, it responds to external events as they occur via **hardware interrupts** or software **traps**.
- To ensure that the kernel remains responsive, interrupt handlers often defer heavier work using mechanisms like:
	- Bottom Halves: The kernel has a concept of bottom halves (softirqs and tasklets) to delay non-critical parts of interrupt processing. The interrupt handler only handles critical tasks immediately, then schedules the remaining work as a bottom half, which the kernel will process later when it’s convenient.
	- Softirqs and Tasklets: These are special kernel structures used to handle deferred work, typically run after interrupt handlers finish and before returning to the user process. Tasklets and softirqs are part of the event-driven infrastructure that allows the kernel to manage concurrent operations without blocking CPU scheduling.
- The are threads, not in same sense as user-space process threads though. They are kthreads or kworker threads used to perform background tasks (disk flushing, deferred work). They run in kernel mode but are scheduled like regular processes!

### When is the **IDT (Interrupt Descriptor Table)** Initialized?

The **Interrupt Descriptor Table (IDT)** is initialized during the **booting process** of the kernel. It is a critical component of the CPU's interrupt handling system and is set up early during kernel initialization.

#### IDT Initialization in Detail:

1. **Early Boot Process**: During kernel boot-up, after the **Global Descriptor Table (GDT)** is initialized (which manages CPU privilege levels and memory segmentation), the kernel initializes the **IDT**.
    
    - The IDT is a table that holds pointers to the **interrupt service routines (ISRs)** and **exception handlers**. These routines handle various interrupts, including hardware interrupts (like I/O devices) and software-generated interrupts (like system calls or traps).
2. **Setting Up the IDT**:
    
    - The kernel populates the IDT with the appropriate handler addresses for each type of interrupt or exception. For example:
        - Certain entries correspond to hardware interrupts, like a timer interrupt or keyboard input.
        - Others correspond to software-generated traps, such as system calls or exceptions like page faults.
3. **Loading the IDT**:
    
    - Once the IDT is set up, the CPU is told where the table is located in memory through the **`lidt` (Load IDT)** instruction. This instruction loads the base address of the IDT into a special CPU register called the **IDTR (Interrupt Descriptor Table Register)**.
    - From this point on, whenever an interrupt or trap occurs, the CPU knows to look up the appropriate handler in the IDT and transfer control to it.