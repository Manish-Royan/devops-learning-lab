# 🧠 Memory — Kernel Internals

> **Goal:** Understand *how the kernel actually finds things in memory* — the 4-level page table walk, the TLB cache, page faults, VMAs, `struct page` / `mem_map`, sparse vs flat memory models, compound pages / folios, and the virtual address space layout on x86-64.

---

## 📍 Going Deeper — How the Kernel Finds Things in Memory

The previous note covered the user-facing model: RAM, cache, swap, OOM killer. Now we go one layer deeper — into the kernel itself.

The question we're answering: **when a process reads virtual address `0x100`, how does the kernel actually find the right physical memory?**

All values in this note are for **x86-64**. Values change per architecture — that's the nature of memory management.

---

## 1️⃣ Virtual Memory — Two Key Ideas

Virtual memory means a lot of things, but the two main ideas to highlight:

1. **A virtual address may not be the same as the physical address.** If a process tries to read virtual address `100`, it may map to physical address `2000`.

2. **Each process has its own virtual memory address space.** Each process has its own little "garden of memory" it can do what it wants with. If Process A reads address `100` and Process B reads address `100`, they may be reading **completely different physical memory** — or maybe the memory isn't even allocated yet.

```
   PROCESS A reads vaddr 0x100  ──┐
                                   ├──▶ may point to completely
   PROCESS B reads vaddr 0x100  ──┘    different physical frames
                                   (or no frame at all yet)

   Each process = independent virtual address space
   "Their own little garden of memory"
```

Under this system, processes have **completely independent memory address spaces**. This is the isolation that makes modern multi-tasking possible.

---

## 2️⃣ Pages and Page Frames — The Building Blocks

When the kernel boots, the first thing it does with memory is split it into small **4 KB chunks** and create a **page** assigned to each chunk.

### 📦 Two Terms You Must Distinguish

| Term | What it refers to |
|---|---|
| **Page frame** | The actual **physical 4 KB chunk of memory** on the RAM module |
| **Page** (`struct page`) | The **data/metadata about that chunk** — what's going on with it |

```
   PHYSICAL RAM (the memory module):           KERNEL'S TRACKING:
   ┌──────────┬──────────┬──────────┬───┐     ┌──────────┬──────────┬──────────┬───┐
   │ page     │ page     │ page     │   │     │  page    │  page    │  page    │   │
   │ frame 0  │ frame 1  │ frame 2  │...│     │ metadata │ metadata │ metadata │...│
   │ (4 KB)   │ (4 KB)   │ (4 KB)   │   │     │  for f0  │  for f1  │  for f2  │   │
   └──────────┴──────────┴──────────┴───┘     └──────────┴──────────┴──────────┴───┘
         │                                         │
         └───────── linked by the page ────────────┘
                    metadata structure
```

### What's Inside a `struct page`?

The page metadata contains:
- **Flags** — what's going on with this memory (is it cache? kernel purpose? specific limitations?)
- **Reference count** — how many processes (or kernel subsystems) are using it

> ⚠️ **Crucially: there is NO address inside `struct page`.** The page metadata doesn't store the physical address directly — we'll see how the kernel derives it later (via `mem_map`).

---

## 3️⃣ The MMU and the TLB — The CPU's Translation Cache

Now we need a way to link virtual addresses to physical ones. For that, we look at the CPU.

The CPU has a **Memory Management Unit (MMU)**. The MMU's job is to take a **virtual address** and spit out a **physical address** — *as fast as possible*. So fast, in fact, that there's no loss in performance when it's done correctly.

### ⚡ The TLB — Translation Lookaside Buffer

The MMU doesn't walk the page table every time — that would be way too slow. Instead, it has a cache called the **TLB (Translation Lookaside Buffer)**.

```
   ┌──────────────────────────────────────────────────────────┐
   │                      CPU CORE                            │
   │                                                          │
   │   1. Process requests vaddr 0x100                        │
   │              │                                           │
   │              ▼                                           │
   │   ┌─────────────────────┐                                │
   │   │  TLB (cache)        │  ← 16-64 entries per core      │
   │   │  "do I know 0x100?" │    (scarce resource!)          │
   │   └──────────┬──────────┘                                │
   │              │                                           │
   │       HIT?   │   MISS?                                   │
   │       │      │                                           │
   │       ▼      ▼                                           │
   │   ⚡ fast   🐢 page fault → ask kernel to fix it         │
   │   (return   (walk the page table, fill TLB)              │
   │    physical                                              │
   │    address)                                              │
   └──────────────────────────────────────────────────────────┘
```

**The TLB lookup flow:**

1. CPU needs to read/write a virtual address
2. First, it asks the **TLB**: *"Do you know this address?"*
3. **If HIT (TLB has it):** Fast — return the physical address immediately
4. **If MISS (TLB doesn't have it):** The CPU calls on the kernel to fix it — this is a **page fault**

> ⚠️ The TLB is a **scarce resource**. There can be 16, 32, or 64 entries per core. That's it. This is why **huge pages** help — they let one TLB entry cover 2 MB or 1 GB instead of just 4 KB, dramatically increasing the effective TLB coverage.

---

## 4️⃣ The Page Table Walk — 4 Levels on x86-64

When the TLB misses, the kernel's **page fault handler** must walk the **page table** to find the physical address. On x86-64, this is a **4-level walk**.

### 🗺️ The 4-Level Page Table Structure

```
   VIRTUAL ADDRESS (48 bits used): 
   ┌─────────┬─────────┬─────────┬─────────┬────────────┐
   │  9 bits │  9 bits │  9 bits │  9 bits │  12 bits   │
   │  (PGD)  │  (PUD)  │  (PMD)  │  (PTE)  │  (offset)  │
   │  39-47  │  30-38  │  21-29  │  12-20  │   0-11     │
   └─────────┴─────────┴─────────┴─────────┴────────────┘
       │         │         │         │          │
       │         │         │         │          └──▶ byte offset within the 4 KB page frame
       │         │         │         │
       │         │         │         └──▶ index into Page Table Entry array
       │         │         └──▶ index into Page Middle Directory
       │         └──▶ index into Page Upper Directory
       └──▶ index into Page Global Directory
```

### 🚶 The Walk, Step by Step

```
   ┌───────────────────────────────────────────────────────────────┐
   │                    THE PAGE TABLE WALK                        │
   │                                                               │
   │   CR3 register                                                │
   │   (points to PGD)                                             │
   │       │                                                       │
   │       ▼                                                       │
   │   ┌──────────────────┐                                        │
   │   │  Page Global     │  ◀── bits 39-47 of vaddr select entry │
   │   │  Directory (PGD) │       (a 4 KB page frame of entries)   │
   │   └────────┬─────────┘                                        │
   │            │ entry points to...                               │
   │            ▼                                                  │
   │   ┌──────────────────┐                                        │
   │   │  Page Upper      │  ◀── bits 30-38 of vaddr select entry │
   │   │  Directory (PUD) │       (same structure, one level down) │
   │   └────────┬─────────┘                                        │
   │            │ entry points to...                               │
   │            ▼                                                  │
   │   ┌──────────────────┐                                        │
   │   │  Page Middle     │  ◀── bits 21-29 of vaddr select entry │
   │   │  Directory (PMD) │       (rinse and repeat)               │
   │   └────────┬─────────┘                                        │
   │            │ entry points to...                               │
   │            ▼                                                  │
   │   ┌──────────────────┐                                        │
   │   │  Page Table      │  ◀── bits 12-20 of vaddr select entry │
   │   │  Entry (PTE)     │       → points to actual 4 KB frame!   │
   │   └────────┬─────────┘                                        │
   │            │                                                  │
   │            ▼                                                  │
   │   ┌──────────────────┐                                        │
   │   │  4 KB page frame │  ◀── bits 0-11 (offset) select byte   │
   │   │  (physical)      │       within this frame                │
   │   └──────────────────┘                                        │
   │                                                               │
   │   🎯 Physical address found! Load it into the TLB.            │
   └───────────────────────────────────────────────────────────────┘
```

Let's walk through it:

1. **CR3 register** — Inside each CPU core, the `CR3` register holds the address of the current process's **Page Global Directory (PGD)**. When a process is scheduled onto a core, its PGD address is loaded into `CR3`.

2. **PGD (Page Global Directory)** — A full 4 KB page frame containing a bunch of entries. We use **bits 39-47** (9 bits) of the virtual address to select which entry to follow.

3. **PUD (Page Upper Directory)** — The PGD entry points to another 4 KB page frame. We use **bits 30-38** (9 bits) to select an entry here. Same structure, one level down.

4. **PMD (Page Middle Directory)** — The PUD entry points to another page frame. We use **bits 21-29** (9 bits) to select an entry. Rinse and repeat.

5. **PTE (Page Table Entry)** — The PMD entry points to a page table. We use **bits 12-20** (9 bits) to select the final entry — which points to the **actual 4 KB page frame**.

6. **Offset** — Finally, we use **bits 0-11** (12 bits) of the virtual address as the byte offset within that 4 KB page frame.

**Physical address found!** The kernel loads this translation into the TLB so future accesses are fast.

### 📊 Why 9 Bits Per Level?

Each level uses 9 bits → 2⁹ = 512 entries per level. Each entry is 8 bytes on x86-64 → 512 × 8 = 4096 bytes = exactly **one 4 KB page frame**. The math works out perfectly.

### 🔢 What's Inside a Page Table Entry?

A PTE contains more than just the physical address:

```
   ┌─────────────────────────────────────────────────────────┐
   │              PAGE TABLE ENTRY (PTE)                     │
   ├─────────────────────────────────────────────────────────┤
   │  [physical address bits] [permissions/flags]            │
   │                                                         │
   │  Flags include:                                         │
   │   • Present bit      — is this page actually in RAM?    │
   │   • No-execute (NX)  — can the CPU execute this?        │
   │   • User/Kernel      — user mode or kernel mode?        │
   │   • Read/Write       — write privileges?                │
   │   • Accessed         — has it been read recently?       │
   │   • Dirty            — has it been written to?          │
   └─────────────────────────────────────────────────────────┘
```

---

## 5️⃣ Huge Pages — Cutting Steps from the Walk

Now you can understand *why* huge pages help. The page table walk is 4 levels deep — that's 4 memory lookups per translation (when the TLB misses). Huge pages **cut the walk short**.

### 📏 How Huge Pages Work in the Page Table

| Page size | Where the walk stops | Which bits become the offset |
|---|---|---|
| **4 KB** (standard) | Goes all the way to PTE | Last 12 bits (offset within 4 KB) |
| **2 MB** huge page | Stops at **PMD** level | Last 9 + 12 = **21 bits** (offset within 2 MB) |
| **1 GB** huge page | Stops at **PUD** level | Last 9 + 9 + 12 = **30 bits** (offset within 1 GB) |

```
   4 KB page (full 4-level walk):
   PGD → PUD → PMD → PTE → [4 KB frame]
                              ↑ stop here, 12-bit offset

   2 MB huge page (3-level walk):
   PGD → PUD → PMD → [2 MB frame]
                       ↑ stop here, 21-bit offset

   1 GB huge page (2-level walk):
   PGD → PUD → [1 GB frame]
                 ↑ stop here, 30-bit offset
```

Instead of going all the way down to the PTE before pointing to a memory chunk, huge pages point to the chunk **at the PMD or PUD level** — covering a much larger region with a single entry.

> 🧭 **DevOps angle:** This is *why* databases love huge pages. A database with a 64 GB buffer pool using 4 KB pages needs 16 million page table entries. With 2 MB huge pages, it needs only 32,000 — 512× fewer entries, 512× better TLB coverage.

---

## 6️⃣ The Present Bit and Swap Tracking

Here's something cool the page table enables: **swapping pages in and out**.

At the end of each page table entry, there's a **present bit**. When it's set to `0`, it means **this entry is not currently in memory** (it's been swapped out).

### 🔄 The Swap-Out Process

```
   BEFORE swap-out:                    AFTER swap-out:

   ┌──────────────────┐                ┌──────────────────┐
   │  PTE:            │                │  PTE:            │
   │  [physical addr] │                │  present=0       │
   │  present=1       │   ───────▶    │  [swap index]   │
   │  (in RAM)        │                │  (in swap_info)  │
   └──────────────────┘                └──────────────────┘
         │                                      │
         ▼                                      ▼
   ┌──────────────────┐                ┌──────────────────┐
   │  page frame      │                │  swap file/      │
   │  (in RAM)        │                │  partition (disk)│
   │  [actual data]   │                │  [actual data]   │
   └──────────────────┘                └──────────────────┘
                                              ▲
                                              │
   Data moved to disk ────────────────────────┘
   PTE's address field replaced with index into swap_info/swap_map
   (kernel structures that are always in memory)
```

1. Kernel takes the memory from the page frame
2. Stores it in the swap partition/file
3. Records *where* to find it in `swap_info` and `swap_map` (kernel structures always in memory)
4. Sets the PTE's **present bit to 0**
5. Puts an **index into swap_info/swap_map** where the address used to be

When the process tries to access that address again → TLB miss → page fault → kernel sees present=0 → reads from swap → loads back into RAM → sets present=1 → returns.

> 💡 This is why swap is slow: every swapped-out access requires a page fault (CPU→kernel) and a disk read.

---

## 7️⃣ VMAs — Virtual Memory Areas

The page table is cool, but it doesn't contain everything that's in your memory. There's another structure: the **VMA (Virtual Memory Area)**.

### 📄 What VMAs Do

VMAs allow the kernel to:
1. **Map a file to memory** (without loading the whole file)
2. **Represent anonymous memory** (malloc'd memory not backed by a file)

### 🗺️ File Mapping with VMAs

When you map a file to memory, you **don't actually load the file into memory**. You just tell the system: *"This chunk of virtual memory is now linked to that file. If someone tries to read it, go to the disk, get the data, and put it in memory."*

```
   PROCESS'S VIRTUAL ADDRESS SPACE:
   ┌──────────────────────────────────────────┐
   │  VMA 1: [0x1000 - 0x5000] → file: a.txt  │  ◀── mapped file
   │  VMA 2: [0x6000 - 0x8000] → NULL (anon)  │  ◀── anonymous (malloc)
   │  VMA 3: [0x9000 - 0xF000] → file: b.txt  │  ◀── mapped file
   └──────────────────────────────────────────┘

   Each VMA contains:
   • start address (virtual)
   • end address (virtual)
   • how to get the underlying data (file address, or NULL)
   • permissions (read/write/execute)
```

### ⚡ Why VMAs Make Allocation Fast

Here's the clever part. Say a process calls `malloc()` for **10 GB of memory**. Is the kernel going to create page table entries for all 10 GB right now? **Of course not.**

```
   malloc(10 GB):
   ┌──────────────────────────────────────────────────────┐
   │  VMA created: [vaddr_start - vaddr_end] → NULL       │
   │  (no page table entries created!)                    │
   │                                                      │
   │  Page table entries are created ON DEMAND:           │
   │  • process writes to vaddr X                         │
   │  • page fault → kernel checks VMAs                   │
   │  • finds the VMA covering vaddr X                    │
   │  • creates a page table entry for just that page     │
   │  • loads it into the TLB                             │
   │  • job done                                          │
   └──────────────────────────────────────────────────────┘
```

Instead of creating page table entries for every page, the kernel just creates a **VMA** and sets its backing to `NULL` (anonymous memory). The actual page table entries are created **lazily, on demand**, only for the pages the process actually touches.

This makes large allocations **way faster** — you don't pay for memory you don't use.

### 🔁 The Complete Fault Handling Flow

Putting it all together — when a process accesses a virtual address:

```
   1. CPU asks the TLB
      │
      ├─ HIT (and permissions OK) ──▶ ⚡ return physical address, done
      │
      └─ MISS (or permission mismatch) ──▶ page fault
                                              │
                                              ▼
   2. Kernel's page fault handler runs
      │
      ├─ Walk the page table (CR3 → PGD → PUD → PMD → PTE)
      │
      ├─ PTE found and present=1?
      │   └─ YES ──▶ load into TLB, done
      │
      └─ PTE not found or present=0?
          │
          ▼
   3. Search VMAs for one covering this address
      │
      ├─ Found a VMA?
      │   └─ YES ──▶ create page table entry, load data (from file or swap), load into TLB, done
      │
      └─ No VMA covers this address?
          │
          ▼
   4. 💥 SEGFAULT — the process tried to access memory it doesn't own
      kernel throws an error (SIGSEGV)
```

> 🧭 **DevOps angle:** This is exactly what a **segfault** is — the process accessed an address with no VMA covering it. When you see "Segmentation fault" in your logs, the kernel walked the page table, found no entry, searched the VMAs, found nothing, and sent `SIGSEGV`.

---

## 8️⃣ `struct page` and `mem_map` — Finding Physical Addresses

Earlier we noted that `struct page` contains **no address**. So how does the kernel get from a `struct page` to a physical address?

### 🗺️ The `mem_map` Array

When the computer boots, the kernel initializes all the `struct page` structures for all the physical memory chunks — and stores them in a **big array called `mem_map`**.

```
   mem_map array (one entry per physical page frame):
   ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬───┐
   │ p0  │ p1  │ p2  │ p3  │ p4  │ p5  │ p6  │...│
   └─────┴─────┴─────┴─────┴─────┴─────┴─────┴───┘
     │     │     │
     │     │     └──▶ corresponds to page frame 2 (physical address 8192)
     │     └────────▶ corresponds to page frame 1 (physical address 4096)
     └──────────────▶ corresponds to page frame 0 (physical address 0)
```

### 🧮 The Math

To find the physical address from a `struct page`:

1. Take the **pointer to the `struct page`**
2. Subtract the **start of `mem_map`** → gives you the **index** (which page frame this is)
3. **Shift left by 12 bits** (equivalent to multiplying by 4096, since each page is 4 KB)
4. Result: **the physical address**

```
   physical_address = (page_pointer - mem_map_start) << 12
                    = (index) × 4096
```

This is a smart way to find the physical address **without having to store it inside every `struct page`** — saving memory.

### 📊 The Cost of `struct page`

Here's a quick fact that shows how important these structures are:

- Each `struct page` is about **64 bytes**
- They're loaded **at all times, 24/7** — whenever the computer is on
- This means roughly **1.5% of your total memory** is used *just for `struct page` metadata*

> 💡 That's how important the flags and reference counts are — they're worth 1.5% of your memory, always. For a 64 GB server, that's ~1 GB just for page metadata.

---

## 9️⃣ Flat Memory vs Sparse Memory — Physical Memory Models

What I just described (`mem_map` as one big array) is one of two physical memory models:

### 📐 Flat Memory (`flatmem`)

- One big `mem_map` array for all physical memory
- Used on **x86-32** and older systems
- Simple, but doesn't scale to huge memory or NUMA

### 🧩 Sparse Memory (`sparsemem`)

- Memory divided into **sections** (`mem_section`)
- Each section has its own `section_mem_map` (acts like `mem_map` for that section)
- Used on **x86-64** and **NUMA-compatible** systems

```
   FLATMEM (one big array):              SPARSEMEM (sectioned):
   ┌──────────────────────────┐          ┌──────────┬──────────┬──────────┬───┐
   │      mem_map (array)     │          │ section 0│ section 1│ section 2│...│
   │  [p0][p1][p2]...[pN]     │          │  mem_map │  mem_map │  mem_map │   │
   └──────────────────────────┘          └──────────┴──────────┴──────────┴───┘
   All memory in one array.              Memory split into sections.
   Simple but doesn't scale.             Scales to huge/NUMA memory.
```

### 🎛️ Why Sparse Memory Enables Cool Features

Splitting memory into sections allows:

1. **Deferred loading** — On massive machines (thousands of cores, terabytes of RAM), just initializing all the `struct page` structures takes time. Sparse memory lets the machine **boot first, then load more memory sections later**.

2. **Memory hot-plug** — You can **add or remove memory sections while the machine is running**. ("I want that section gone" or "add a new section.") This is impossible with flatmem.

3. **NUMA support** — Necessary for Non-Uniform Memory Access architectures (where different CPUs have different physical RAM access speeds). NUMA is a whole topic on its own.

---

## 🔟 Compound Pages and Folios — Managing Groups of Pages

Sometimes we want a bunch of pages to be managed as one unit. This becomes critical for **huge pages** — if you have a 1 GB huge page, you don't want to manage all 262,144 underlying 4 KB pages individually.

### 🔗 Compound Pages

The kernel sets up the group like this:

```
   ┌──────────────────────────────────────────────────────────┐
   │  COMPOUND PAGE (e.g., a 2 MB huge page = 512 × 4 KB)     │
   │                                                          │
   │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐    ┌────┐  │
   │  │ HEAD │ │ tail │ │ tail │ │ tail │ │ tail │... │tail│  │
   │  │ page │ │ page │ │ page │ │ page │ │ page │    │page│  │
   │  └──┬───┘ └──┬───┘ └───┬──┘ └──┬───┘ └──┬───┘    └─┬──┘  │
   │     │        │         │       │        │          │     │
   │     │        └─────────┴───────┴────────┴──────────┘     │
   │     │                       │                            │
   │     │                       └── all tails point to HEAD  │
   │     │                           (get info from head)     │
   │     │                                                    │
   │     └── HEAD stores ALL info: flags, ref count, etc.     │
   │         (editing HEAD = editing the whole group)         │
   └──────────────────────────────────────────────────────────┘
```

- **First page** = **page head** — stores all the information, attributes, and flags for the entire group
- **Other pages** = **page tails** — each contains a **pointer to the head**

This way:
- You only look at the **head** to get information about the whole group
- You only edit the **head** to change the whole group
- Huge performance win for managing large contiguous regions

### 🛡️ Folios — The Safe Way to Use Compound Pages

There's a problem with compound pages: **what if you accidentally edit a tail page directly?**

That could cause corruption — you'd be editing something not meant to be edited, and you'd miss information that should go to the head.

**Folio**s solve this. A **folio** is just **the safe way to manage compound pages**:

| | Compound Pages (raw) | Folios |
|---|---|---|
| **What it is** | The underlying mechanism | A safe API wrapper |
| **Editing tails** | ⚠️ Possible to corrupt | ✅ Forces you to go through the head |
| **Universality** | Limited use | Used throughout the kernel |

When you hear "we've added folios to X" in a kernel release, it really means **"we've added compound pages to X in a safe way."** Folios make compound pages safer and more universal throughout the kernel.

> 🧭 **DevOps angle:** If you read kernel changelogs (which you will, as a senior DevOps engineer), "folios" comes up constantly. Now you know: it just means "safe compound pages."

---

## 1️⃣1️⃣ Virtual Memory Address Space — The Big Picture

Let's end with some quick facts about the x86-64 virtual address space.

### 🔢 The Address Space Numbers

```
   ┌───────────────────────────────────────────────────────────────┐
   │           x86-64 VIRTUAL ADDRESS SPACE LAYOUT                 │
   │                                                               │
   │   Full 64-bit theoretical:  16 exabyte  (ridiculous, unused)  │
   │   Actually used:            48 bits                           │
   │   Addressable:              256 terabyte                      │
   │                                                               │
   │   ┌─────────────────────────────────────────┐                 │
   │   │  USER SPACE (process addressable)       │  128 TB         │
   │   │  0x0000000000000000 - 0x00007FFFFFFFFFFF│                 │
   │  ─┤                                         ├─                │
   │   │  KERNEL SPACE (kernel only)             │  128 TB         │
   │   │  0xFFFF800000000000 - 0xFFFFFFFFFFFFFFFF│                 │
   │   │   ┌───────────────────────────────┐     │                 │
   │   │   │  Direct memory map: 64 TB     │     │  ◀── kernel can│
   │   │   │  (phys addr + 128 TB offset)  │     │     access any  │
   │   │   └───────────────────────────────┘     │     phys addr   │
   │   └─────────────────────────────────────────┘                 │
   └───────────────────────────────────────────────────────────────┘
```

| Fact | Value |
|---|---|
| Full 64-bit theoretical | 16 exabyte (completely ridiculous — nobody needs this) |
| Actually used by the kernel | **48 bits** |
| Total addressable | **256 terabyte** |
| **Per-process user space limit** | **128 terabyte** (half of 256) |
| Kernel space | 128 terabyte (the other half) |
| Direct memory map in kernel space | **64 TB** (half of kernel space) |

### 🎯 The Direct Memory Map

Here's the cool part: **half of the kernel's address space (64 TB) is reserved for direct memory mapping.**

If the kernel wants to access physical address `100`, it can go to address `100 + 128 TB` (the offset where the direct map starts) and it will be at that physical address directly.

```
   KERNEL ADDRESS              PHYSICAL ADDRESS
   0xFFFF800000000000  +  0   =  physical address 0
   0xFFFF800000000000  +  100 =  physical address 100
   0xFFFF800000000000  +  X   =  physical address X
            ↑
       (128 TB offset,
        start of direct map)
```

This lets the kernel access any physical memory location directly — but it's **only available to the kernel**, never to user processes. (That would be a big no-no — processes can't see physical addresses directly, only virtual ones.)

> 💡 The remaining 16 bits of the 64-bit address (bits 48-63) are unused on standard x86-64. That's why there was a "16 bit that we were not using" in the page table — only 48 bits are actually used for addressing.

---

## 🧾 Summary

| Concept | One-line takeaway |
|---|---|
| **Virtual memory** | A virtual address may map to a different physical address; each process has its own independent virtual address space. |
| **Page frame vs page** | Page frame = the physical 4 KB chunk of RAM. Page (`struct page`) = the kernel's metadata about it. |
| **`struct page` contents** | Flags (what's going on) + reference count (how many are using it). **No address stored.** |
| **MMU** | Memory Management Unit — CPU component that translates virtual addresses to physical ones. |
| **TLB** | Translation Lookaside Buffer — small CPU cache (16-64 entries/core) of recent virtual→physical translations. |
| **TLB hit vs miss** | Hit = fast return. Miss = page fault → kernel walks the page table. |
| **4-level page table** | PGD → PUD → PMD → PTE → page frame. 9 bits per level + 12-bit offset = 48-bit virtual address. |
| **CR3 register** | CPU register holding the current process's PGD address. Loaded on context switch. |
| **9 bits per level** | 2⁹ = 512 entries per level. 512 × 8 bytes = 4096 bytes = one 4 KB page frame. Math is elegant. |
| **PTE flags** | Present bit, no-execute (NX), user/kernel, read/write, accessed, dirty. |
| **Huge pages** | 2 MB (stop at PMD) or 1 GB (stop at PUD) — cuts the page table walk short, fewer TLB entries needed. |
| **Present bit = 0** | Page is swapped out. PTE's address field becomes an index into `swap_info`/`swap_map`. |
| **VMA (Virtual Memory Area)** | Maps a range of virtual addresses to a file (or NULL for anonymous memory). Enables `mmap` and lazy allocation. |
| **VMA + lazy allocation** | `malloc(10 GB)` creates a VMA, not page table entries. PTEs created on demand when pages are touched. |
| **Segfault** | Process accessed an address with no VMA covering it → `SIGSEGV`. |
| **`mem_map`** | Big array of all `struct page` structures, one per physical page frame. Index × 4096 = physical address. |
| **`struct page` cost** | ~64 bytes each, always in memory = ~1.5% of total RAM. Worth it. |
| **Flat memory** | One big `mem_map` array. Used on x86-32. Doesn't scale. |
| **Sparse memory** | Memory split into sections (`mem_section`), each with its own `section_mem_map`. Used on x86-64/NUMA. Enables hot-plug and deferred loading. |
| **Compound pages** | Group of pages managed as one (for huge pages). HEAD stores all info; tails point to head. |
| **Folios** | The safe API for compound pages — forces you to go through the head. What "adding folios" in kernel changelogs means. |
| **48-bit addressing** | x86-64 uses 48 of 64 bits → 256 TB addressable. Bits 48-63 unused. |
| **User space limit** | 128 TB per process (half of 256). |
| **Kernel space** | 128 TB (the other half). Includes a 64 TB direct memory map. |
| **Direct memory map** | Kernel can access physical address X by going to X + 128 TB offset. Kernel-only — processes can't use it. |

---

← Back to [Linux Memory](./README.md) | Previous: [User-Facing Model](./1-memory-user-facing.md) | Next: [Monitoring Commands](./3-monitoring-commands.md) →
