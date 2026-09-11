---
title: "OS/161 Demand Paging"
date: 2024-09-13
summary: "An OS/161 kernel extension implementing demand paging, swap space management, per-process page tables, dynamic ELF loading, and TLB handling."
tags: ["C", "Systems Programming"]
featured: true
---

<!-- Enable math formatting -->
{{< katex >}}

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://github.com/LienoPC/OS161-DemandPaging" target="_blank" >}}
    {{< icon "github" >}} View Source Code
  {{< /button >}}
</div>

This project implements a demand-paging virtual memory subsystem within the OS/161 operating system kernel. It replaces the default static memory allocator, enabling the execution of user programs whose memory footprint exceeds available physical RAM.

**Project Overview:**
*   **Role:** Kernel Developer (Team of 2).
*   **Context:** Systems Programming — OS/161 Architecture.
*   **Responsibilities:** Swap space storage, page replacement logic, TLB management, on-demand ELF loading.

## Contribution Overview

To support demand paging, the following core components were implemented:

*   **Swap Space Management:** a disk-backed storage manager utilizing a custom self-describing framing protocol and in-place frame swapping to prevent unbounded file growth.
*   **Page Replacement:** a doubly-linked FIFO tracking data structure designed to manage page residency and victim selection in the absence of hardware reference bits.
*   **TLB Management:** a translation lookaside buffer manager providing round-robin slot replacement and software-enforced write protection.
*   **On-Demand ELF Loading:** a loading sequence that eliminates eager binary loading by lazily streaming executable segments directly from disk into memory.

## Engineering Highlights

### Swap Space and In-Place Replacement
To serialize evicted pages to disk without maintaining complex external block-allocation tables, a self-describing framing format was designed. Every page written to the swap file is structured as a 4100-byte record: a 4-byte header storing the virtual address followed by a 4096-byte data payload. This structure allows the swap manager to perform linear disk lookups and assert data integrity directly against the on-disk record header. To prevent continuous file expansion during sustained paging, which would quickly exhaust disk capacity, an in-place swap replacement mechanism was implemented. When retrieving a swapped page, the data is read into a temporary stack buffer, the exact disk offset is overwritten with the data of the newly evicted victim page, and the buffered page is finally copied into RAM. This ensures disk usage remains neutral during active page replacement.

### FIFO Page Replacement
Because the MIPS R3000 CPU lacks hardware reference or access bits, implementing a strict Least Recently Used (LRU) algorithm without severe software overhead is impractical. Instead, a FIFO replacement policy was developed. The tracking structure is maintained on a per-address-space basis as a doubly-linked list, encapsulating page table indices. Maintaining both head and tail pointers enables \(\mathcal{O}(1)\) push operations at the tail when a physical frame is mapped, and \(\mathcal{O}(1)\) pop operations at the head to instantly identify eviction candidates. The queue also supports arbitrary node extraction, allowing the kernel to gracefully remove entries if a page is unmapped or migrated independently of the standard replacement sequence.

### TLB and Memory Protection
On the MIPS R3000 processor, virtual-to-physical address translation is managed through a software-managed Translation Lookaside Buffer containing 64 hardware slots. When a TLB miss occurs, the trap handler searches for a free slot or, if the buffer is full, delegates to a rolling modulo counter for deterministic round-robin victim selection. Furthermore, the MIPS architecture does not provide a dedicated "write-enable" control bit, relying instead on a "dirty" bit. This behavior was leveraged to enforce segment protection. By leaving the dirty bit cleared for addresses falling within the compiled program's text segment, any illegal write attempt safely triggers a `VM_FAULT_READONLY` hardware trap, which the dispatcher intercepts to terminate the rogue process. To ensure atomic programming, the entire TLB update sequence is strictly wrapped between interrupt masking calls.

### On-Demand ELF Loading
The baseline OS/161 kernel eagerly copies all ELF program segments into contiguous memory during `load_elf`, wasting physical RAM on unexecuted code and failing completely on large executables. To enable demand loading, a segment metadata structure was designed to store executable vnode references, program headers, and virtual base addresses without allocating physical frames upfront. The kernel loader was modified to bypass segment copying entirely, initializing process address spaces with empty page tables and metadata descriptors, thereby consuming negligible RAM at startup. Because pages must be read dynamically during execution, the filesystem lifecycle was updated. The internal reference count of the binary vnode is explicitly incremented during process startup, preserving the active file handle long after the initial launch and ensuring the executable remains accessible for lazy loading.
