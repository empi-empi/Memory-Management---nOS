# Memory Management Simulator

An interactive, single-file HTML simulator demonstrating core memory management
concepts in a conceptual operating system: **paging, demand paging, the TLB,
copy-on-write (COW), and memory fragmentation**.

No build step or dependencies — open `memory_management_simulator.html` in any
modern browser to run it.

---

## Overview

The simulator models a physical memory space of **64 frames × 4KB = 256KB**,
using **fixed-size paging** as its allocation strategy. It is organized into
three tabs:

| Tab | Concepts demonstrated |
|---|---|
| **Paging** | Fixed-size page allocation, 3-level page tables, fork/copy-on-write |
| **Virtual Memory** | Virtual address translation, TLB caching, demand paging, page replacement (LRU) |
| **Fragmentation** | External fragmentation, compaction, internal fragmentation |

A live stats bar at the top tracks **free physical memory**, **demand paging
faults**, and **TLB hit rate** across all tabs.

---

## Tab 1: Paging

### Allocate a Process
- Enter a process name and a page count (1–16), then click **Allocate**.
- Pages are assigned to the next available free frames — no fit algorithm is
  used, since fixed-size paging means every block is identical and external
  fragmentation cannot occur at the frame level.
- The **Physical Memory** grid colors each frame by owning process; click any
  frame to log its owner and status to the System Log.

### Page Table
- Select a process from the dropdown to view its page table.
- Each row shows the **L1 / L2 / L3 indices** (from a simulated 3-level page
  table), **VPN → PFN** mapping, and the **Valid**, **COW**, **Dirty**, and
  **Referenced** bits found in real page table entries.

### Fork & Copy-on-Write (COW)
- **Fork Selected Process** creates `<name>_child`, sharing all of the
  parent's physical frames. Shared frames are marked `COW` in the memory grid.
- **Simulate Write Access** (enter a VPN, then click) breaks COW for that
  page: a new frame is allocated for the writer, its `cow` flag clears, and
  `dirty` is set. A second write to an already-broken page just sets the dirty
  bit — no further copy is needed.
- **Free** a process from the **Running Processes** list to release its
  frames. Shared (COW) frames are only fully released once *all* owning
  processes have freed them.

---

## Tab 2: Virtual Memory

### Address Translation
- Select a process and enter a virtual address (hex or decimal), then click
  **Translate**.
- The breakdown panel shows the **L1 / L2 / L3 page table indices** and
  **12-bit page offset** (4KB pages), plus the resulting physical address.
- Each translation reports a **TLB hit or miss**:
  - First access to an address → **miss** (added to TLB).
  - Repeat access → **hit**.
  - Access to an unmapped VPN → **page fault**.
- If the resolved page is COW, the result notes that a write will trigger a
  new-frame allocation.

### TLB (Translation Lookaside Buffer)
- An 8-entry cache of recent VPN → PFN mappings.
- **Flush TLB** simulates a context switch, clearing all cached entries.
- **Simulate Page Reference** generates random VPN accesses against the TLB,
  using **LRU** eviction once the TLB is full.
- **Change Page Mapping** remaps the selected process's current VPN to a new
  frame (e.g. simulating a post-compaction remap), resets its dirty/referenced
  bits, invalidates the stale TLB entry, and flushes the TLB.

### Demand Paging & Page Replacement
- Enter a page number (0–15) and click **Reference Page**.
- With only **4 simulated frames**, each new page reference is a fault that
  loads the page from "disk."
- Once all 4 frames are full, further faults trigger **LRU eviction** — the
  least-recently-used page is evicted to make room.
- **Clear** resets the demand-paging simulation.

---

## Tab 3: Fragmentation

### External Fragmentation
- **Simulate Step** randomly allocates/frees 8KB blocks across a 32-block
  region, mimicking processes starting and exiting over time.
- The status line reports whether free space is contiguous or
  **⚠ FRAGMENTED** (scattered).
- **Compact** consolidates all used blocks together, merging free space into
  one contiguous region — shown in a before/after **Compaction Visualizer**.
- **Reset** restores the initial seeded layout.

### Internal Fragmentation
- Enter a process size in KB and click **Calculate**.
- With 4KB pages, the simulator shows how many pages are allocated and how
  much of the final page is wasted (e.g. a 13KB process needs 4 pages = 16KB
  allocated, wasting 3KB; a 16KB process wastes 0KB).

---

## Key Takeaways

- **Paging** eliminates external fragmentation by allocating in fixed-size
  units, at the cost of **internal fragmentation** in the last page of each
  process.
- **Copy-on-write** makes process creation (`fork()`) cheap by sharing pages
  until a write forces a copy.
- The **TLB** avoids repeated page-table walks, but must be flushed or
  selectively invalidated on context switches and remappings.
- **Demand paging** loads pages only when referenced, requiring a
  **page replacement policy** (LRU here) when physical memory is full.
- **Compaction** fixes external fragmentation but requires relocating and
  re-mapping all affected processes.

---

## Technical Notes

- Physical memory: 64 frames × 4KB = 256KB total.
- Page size: 4KB (12-bit offset).
- Page table: simulated 3-level (L1/L2/L3) structure.
- TLB: 8 entries, LRU replacement.
- Page replacement (demand paging panel): 4 frames, LRU replacement.
- All state is held in memory (JavaScript variables) and resets on page
  reload, or via the **Reset** button.

