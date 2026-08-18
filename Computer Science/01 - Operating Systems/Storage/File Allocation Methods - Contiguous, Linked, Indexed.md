---
title: File Allocation Methods - Contiguous, Linked, Indexed
subject: Operating Systems
module: File Systems
difficulty: Advanced
prerequisites:
  - "[[Operating System]]"
  - "[[Kernel]]"
  - "[[File Concept and File Attributes]]"
  - "[[File Descriptors and File Tables]]"
  - "[[Directory Structures and Path Resolution]]"
related:
  - "[[Inodes and File System Metadata]]"
  - "[[Virtual File System - VFS]]"
  - "[[Page Cache and Buffer Cache]]"
  - "[[ext4 Architecture Overview]]"
aliases:
  - File Allocation Methods - Contiguous, Linked, Indexed
  - File Allocation Methods
  - Contiguous Allocation
  - Linked Allocation
  - Indexed Allocation
  - FAT File System
  - Extents
  - Extent Trees
tags:
  - os
  - storage
  - filesystem
  - algorithms
  - linux-kernel
  - architecture
status: complete
---

# File Allocation Methods: Contiguous, Linked, and Indexed

> [!abstract] Mental Model
> Physical block allocation is **urban real estate development**:
> - **Contiguous Allocation**: Buying an unbroken city block ($50$ consecutive plots). Perfect for high-speed drag racing (**Sequential Read/Write Speed**), but if a building needs an expansion later, you hit immovable neighboring buildings (**External Fragmentation**).
> - **Linked Allocation (FAT)**: A scavenger hunt across scattered vacant city lots. No land is wasted, but finding the 40th room requires walking through rooms $1 \to 39$ (**$O(N)$ Random Access Seek Penalty**).
> - **Indexed Allocation & Extents (UNIX / ext4)**: A central blueprint office (**Inode / Extent Tree**) holding direct GPS coordinates to all plots. Delivers instant $O(1)$ access anywhere.

---

## The Allocation Architecture Taxonomy

```mermaid
flowchart TD
    subgraph AllocationMethods ["Storage Block Allocation Architectures"]
        Contig["1. Contiguous Allocation<br/>• File occupies contiguous set of linear LBA disk sectors.<br/>• Direct: O(1) via (Start + Offset). High sequential throughput.<br/>• Flaw: Severe external fragmentation; dynamic growth impossible."]
        
        Linked["2. Linked Allocation & FAT<br/>• Each block stores pointer to next block (or central FAT table).<br/>• Zero external fragmentation.<br/>• Flaw: Catastrophic random access seek latency O(N)."]
        
        Indexed["3. Multi-Level Indexed Allocation (UNIX Inode)<br/>• Central index block holds array of direct/indirect block pointers.<br/>• Fast random access; scales to massive files via indirect trees."]
        
        Extents["4. Extent-Based Allocation (ext4 / XFS / Btrfs)<br/>• Maps contiguous block ranges: (Logical Start, Physical Start, Length).<br/>• 1 descriptor maps up to 128 MB contiguous data in O(1)!"]
    end
```

---

## 1. Contiguous Allocation

The directory entry stores only two fields: **Start Block** and **Length (in blocks)**:

```mermaid
flowchart LR
    DirEntry["Directory Entry: (Start: 1000, Length: 5)"]
    
    DirEntry --> B0["Block 1000"]
    B0 --> B1["Block 1001"]
    B1 --> B2["Block 1002"]
    B2 --> B3["Block 1003"]
    B3 --> B4["Block 1004"]
```

- **Advantages**:
  - Unrivaled sequential I/O performance (HDD read heads don't seek; Flash controllers stream full NAND pages).
  - Direct Random Access is trivial: Address of block $k = \text{Start} + k$ in $O(1)$ time.
- **Fatal Production Flaws**:
  - Suffers from **External Fragmentation** (requires periodic disk compaction).
  - Cannot expand a file without reallocating the entire file to a larger contiguous hole.
- **Modern Deployments**: ISO 9660 (CD-ROMs/DVDs), high-throughput video recording swap buffers.

---

## 2. Linked Allocation & FAT (File Allocation Table)

### Pure Linked Allocation:
Every data block reserves its final 4 bytes to store the integer address of the next disk block:

```mermaid
flowchart LR
    Dir["Dir: Start = 28"] --> B28["Block 28 (Data + Next: 412)"]
    B28 --> B412["Block 412 (Data + Next: 89)"]
    B412 --> B89["Block 89 (Data + Next: -1 EOF)"]
```
- *Flaw*: Seeking to byte $500\text{ MB}$ requires $128,000$ sequential disk reads! A single corrupted pointer loses all downstream data.

---

### The Evolution: FAT (File Allocation Table - MS-DOS / FAT32)
Moves all block linkage pointers into a **single centralized table cached in RAM**:

```mermaid
flowchart TD
    subgraph FAT_Table ["File Allocation Table in RAM"]
        F28["Entry 28 -> 412"]
        F412["Entry 412 -> 89"]
        F89["Entry 89 -> EOF (-1)"]
    end

    DirFAT["Directory: (Start: 28)"] --> F28
```
- **Benefit**: Random access seeks occur purely in RAM by walking table indexes without issuing physical disk seeks.
- **Bottleneck**: Table size scales with disk capacity (A $2\text{ TB}$ disk requires large FAT tables permanently occupying RAM).

---

## 3. Indexed Allocation (UNIX Multi-Level Inodes)

Each file owns an **Inode** containing an array of **Direct, Single Indirect, Double Indirect, and Triple Indirect pointers**:

```mermaid
flowchart TD
    subgraph InodePointers ["Classical UNIX Inode Pointer Hierarchy (4 KB Blocks)"]
        Direct["12 Direct Pointers -> 12 x 4 KB = 48 KB (Fast Path)"]
        SingInd["1 Single Indirect Pointer -> 1 Index Block (1024 Pointers) = 4 MB"]
        DoubInd["1 Double Indirect Pointer -> 1024 Single Indirects = 4 GB"]
        TripInd["1 Triple Indirect Pointer -> 1024 Double Indirects = 4 TB"]
    end
```

---

## 4. Modern Industry Standard: Extent-Based Allocation (`ext4`, `XFS`)

In modern file systems, single block pointers create excessive metadata bloat. A $100\text{ GB}$ file would require $26.2\text{ million}$ individual 4-byte block pointers!

**Extents** solve this by describing contiguous block ranges using a **12-byte Extent Descriptor**:

```mermaid
flowchart LR
    subgraph ExtentStruct ["struct ext4_extent (12 Bytes)"]
        LogBlock["ee_block: Logical File Block Offset (32-bit)"]
        Len["ee_len: Number of Contiguous Blocks (up to 32,768 = 128 MB)"]
        PhysBlock["ee_start: Starting Physical Disk Block (48-bit)"]
    end
```

```mermaid
flowchart TD
    subgraph ExtentBenefit ["Metadata Efficiency: 100 GB Contiguous Video File"]
        Traditional["Classical Indirect Block Tree: ~100,000 Metadata Index Blocks (~400 MB RAM/Disk)"]
        ModernExt4["ext4 Extent Tree: ~800 Extent Descriptors (< 10 KB Metadata!)"]
        
        Traditional --- ModernExt4
    end
```

---

## Comprehensive Allocation Methods Comparison

| Feature | Contiguous | Pure Linked | FAT32 | UNIX Multi-Level Inode | Modern Extents (ext4/XFS) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Sequential Read** | **Maximum** | Poor | Good | Excellent | **Maximum** |
| **Random Access** | $O(1)$ | $O(N)$ Disk Reads | $O(N)$ RAM Hops | $O(1)$ Tree Walk | **$O(1)$ B+ Tree Search** |
| **External Fragmentation** | **Severe** | None | None | None | Low (Buddy Allocator) |
| **File Growth** | Difficult | Trivial | Trivial | Trivial | Trivial |
| **Metadata Overhead** | **Zero** | 4B per block | Large table in RAM | Moderate | **Extremely Low** |

---

## Active Recall & Interview Perspective

### Reasoning Prompts
1. *Why does FAT32 exhibit dramatically better random-access read performance than pure linked allocation?*
   - **Answer**: In pure linked allocation, the pointer to the next block is physically embedded inside the payload of each on-disk data block. Traversing to block $N$ requires reading $N-1$ physical disk blocks across the storage bus. In FAT32, all pointer links are decoupled from storage blocks and aggregated into a single centralized **File Allocation Table (FAT)** that is cached in RAM. Finding block $N$ requires hopping through memory integers in nanoseconds, issuing only a single physical I/O read for the final destination block.
2. *How does ext4 Extent-based allocation drastically reduce metadata overhead for large sequential files compared to ext3 multi-level indirect blocks?*
   - **Answer**: ext3 used indirect block pointers where every single $4\text{ KB}$ data block required an individual $4$-byte pointer in an index block. A $100\text{ GB}$ file required millions of pointer entries taking up hundreds of megabytes of metadata disk blocks. ext4 replaces individual pointers with **Extents** ($\langle \text{logical\_block, length, physical\_block} \rangle$). A single extent descriptor can map up to $32,768$ contiguous blocks ($128\text{ MB}$) in just $12\text{ bytes}$, shrinking metadata requirements by over $99.9\%$.
3. *Why is Contiguous Allocation still utilized in optical media (ISO 9660) and specialized video streaming storage?*
   - **Answer**: Contiguous allocation provides the highest possible sustained read throughput and minimal latency because data sectors are physically adjacent, completely eliminating disk head seek times on HDDs and enabling maximal prefetching/burst transfers on flash NAND. Because optical media (CD/DVD) and pre-recorded video files are write-once and immutable, they do not suffer from dynamic file resizing or runtime external fragmentation, making contiguous allocation ideal.

---

## Key Takeaways
- **Contiguous Allocation** maximizes sequential speed but suffers from external fragmentation.
- **Linked / FAT Allocation** eliminates fragmentation but introduces random seek bottlenecks.
- **Modern filesystems (ext4, XFS, Btrfs)** use **Extent Trees** to combine the blazing sequential speed of contiguous blocks with dynamic $O(1)$ B-Tree scaling.

---

## Related Notes
- [[Operating System]] — Storage subsystem architecture.
- [[File Concept and File Attributes]] — Logical file properties.
- [[File Descriptors and File Tables]] — File handle operations.
- [[Directory Structures and Path Resolution]] — Directory parsing.
- [[Inodes and File System Metadata]] — Inode pointer structures.
- [[ext4 Architecture Overview]] — Extent tree implementation in ext4.
- [[Operating Systems MOC]] — Master table of contents for Operating Systems.
