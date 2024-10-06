1a. 
- os1 should allocate longer, less frequent, contiguous time slices so that the cpu-intensive tasks can run longer without interruptions from context switches
- os2 should allocate shorter, frequent, time slices distributed throughout the time vector, potentially based on games need for frequent input processing and frame updates.
1b.
- Kernel handles interrupt
- upcalls to library os interrupt handler to save state
- save state time is bounded, so record how long it takes and potentially penalize of taking too long
- we have a s-tlb, we need to save os1s stuff into it...we probably need to flush the hardware tlb, and load up os2s s-tlb into the hardware tlb
- update page table base register to point to os2s page table
- when library os done, hand off control to os2, allowing it to execute its handler
1c.
- Makes sure that give equal total time to both library oses
- Bounded time enforcement, relinquish cpu and save context
- adhereing to time slices, give time proportional to each oses time slices
- penalizing potentially forfeiting future time slices

2a.
- Yes, i agree with my friend because
- It will reduce the explicit cost on context switching as we do not have to tlb flush or update the ptbr since segment registers are used to keep the two process in the same address space. No border crossings needed
2b.
- a, b, d
2c.
- a mechanism would be something like api to enqueue threads
- a policy would be something like the scheduling algorithm priority-based like FCFS
2d.
-  the segment registers provide the address range for a process, providing isolation without changing the address space. Since hardware address space remains the same, the TLB entries are stil valid. Flushing is not needed. FALSE.
2e.
- The PTBR will need to change, so the TLB will need flushing

1.
- the virtualized environment traps to kernel for privelege level need
- the kernel needs elevated permissions to execute its desired instruction, so it will trap to the hypervisor, the hypervisor intercepts trap
- the hypervisor intercepts instruction and updates it shadow page table to set page x to read-only
- the hypervisor invalidates tlb entries for virtual page x to ensure the change takes effect immediately, hv returns control to guest os and guest os is unaware what needed to take place
2.
- 9 MB, 1 for each guest, 1 spt for each process, 1 mb for a shadow page table
3.
- since its page sharing, it will cause a page fault, which the hypervisor intercepts, hypervisor will acknowledge access violation by making new page with cow, mark it as wriable, copy content to it from mpn1, update stp to point vpn2 to mpn2, the reference count for mpn1 will be decreased, tlb entry for vpn2 will be invalidated, and resume execution for os2 where write operation is retried.
4.
- It will tax each vm 50%, which would all added up to 800 MB, with inflation in each vm to reclaim memory, using the recording physical pages communicated over the oovert channel then reclaims the memory, and defaltes vm4 800 MB, no second cycle was needed
5.
- dependence on the guest os, it needs to have balloon driver installed and functioning properly
- could impose llimitations on balloon size
- performance degradation when we potentially starve the processes inside the vm of memory

1.
- For ncc, we need runtime language and os abilities to enforce cache coherence at the software level, and we also need to able to trust that instructions are not reordered in a different order than we see in program, we need atomicity in our instructions read and writes, both
2.
- For cc, we can rely on hardware cache coherence, so then we just need to know that we have program order preserved, and that have atomicity in our instructions
3.
- We need to be able to trust there there is a unified memory across both partitions, meaning we have a consistent view of memory without additional software intervention, we need software abiliities to manage cache coherence because of the lack of it in the ncc component. Memory operations must not violate sequential consistency on both partitions, we need explicit synchronization when accessing data across partitions. So the only thing needed from the architecture is really just the unified memory. program order needs to be there, 
4.
- It requires test and set function in its system api calls
- Needs to be fully atomic ensuring mutal exclusiion when attemtping to acquire the lock,
- we need cache coherence as well as we spin on the same memory 
4b.
- small critical sections, low contention, lack of fairness (fixed delay times could lead to some cpus being starved)
4c.
- large critical sections, high contention, cache-friendly behavior (L == locked) leverage cache coherence benefits spin LOCALLy
5.
- we have a priority order, so fair share is guaranteed, dynamic waits calculated going to be more optimal, lower contention possiblities with the fetch and inc once and then spoins on now_serving which is mostly read only, scalability is better when increasing 
5b.
- bandwidth saturation, update messages are larger, so we will have more latency and increased cache coherence traffic
5c.
- traffic is less compared to update, but downsides are memory access latecny for spinninig processors, invalidates are smaller, better scalaability in terms of network bandwitch but increasing wait time for processors due to memory accesses