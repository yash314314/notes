---
title: Journaling File Systems and Crash Consistency
subject: Operating Systems
module: File Systems
difficulty: Advanced
prerequisites:
  - "[[Operating System]]"
  - "[[Kernel]]"
  - "[[File Concept and File Attributes]]"
  - "[[File Descriptors and File Tables]]"
  - "[[Inodes and File System Metadata]]"
  - "[[Virtual File System - VFS]]"
  - "[[Page Cache and Buffer Cache]]"
related:
  - "[[ext4 Architecture Overview]]"
  - "[[XFS and ZFS Overview]]"
aliases:
  - Journaling File Systems and Crash Consistency
  - Journaling File Systems
  - Crash Consistency
  - Write-Ahead Logging
  - JBD2
  - Journal Modes
  - data=ordered
  - fsck
tags:
  - os
  - storage
  - filesystem
  - reliability
  - linux-kernel
  - database
status: complete
---

# Journaling File Systems and Crash Consistency

> [!abstract] Mental Model
> Journaling is **double-entry bookkeeping before committing a real estate transfer**:
> - Appending data to a file requires updating **three separate disk sectors**: the **Data Block ($D$)**, the **Inode ($I$)**, and the **Allocation Bitmap ($B$)**.
> - If power cuts out when only 2 of the 3 sectors are written, the disk is left in a corrupt, torn state (**The Crash Consistency Problem**).
> - A **Journaling File System** writes a pre-flight transaction receipt to a contiguous log (**The Journal / Write-Ahead Log**) *before* modifying the filesystem. If power dies, rebooting replays the journal in **$< 1\text{ second}$**, completely eliminating hours of destructive `fsck` scans.

---

## The Crash Consistency Problem: The 3-Write Dilemma

Appending a $4\text{ KB}$ block requires three distinct disk updates:
1. **$D$ (Data Block)**: The actual file payload data.
2. **$I$ (Inode)**: Updating `i_size` and adding the block pointer.
3. **$B$ (Data Bitmap)**: Marking the newly allocated block as `1` (used).

```mermaid
flowchart TD
    subgraph ThreeWrites ["The Three Disjoint Disk Writes"]
        D["1. Data Block (D)"]
        I["2. Inode Metadata (I)"]
        B["3. Allocation Bitmap (B)"]
    end

    subgraph FailureScenarios ["Crash Failure Scenarios (Power Cut Midway)"]
        F1["Crash after writing ONLY D<br/>• Result: Data is orphaned. Minor space leak, zero corruption."]
        F2["Crash after writing ONLY I<br/>• Result: CRITICAL CORRUPTION! Inode points to unwritten garbage disk sectors."]
        F3["Crash after writing ONLY B<br/>• Result: Bitmap shows block allocated, but no file owns it. Permanent leak."]
        F4["Crash after writing I and B, but NOT D<br/>• Result: File points to stale/uninitialized data (Severe privacy leak!)."]
    end

    D -.-> F1
    I -.-> F2
    B -.-> F3
    ThreeWrites -.-> F4
```

---

## Write-Ahead Logging (WAL) & The 5-Stage Journal Lifecycle

Linux filesystems (ext4 using the **JBD2 - Journaling Block Device 2** engine) solve this via a 5-step state machine:

```mermaid
sequenceDiagram
    autonumber
    participant RAM as Linux Page Cache
    participant Journal as On-Disk Journal (JBD2 Log)
    participant Disk as Home Storage Locations

    Note over RAM: Application appends 4 KB to file
    RAM->>Journal: 1. Journal Write (Descriptor + Inode + Bitmap changes)
    RAM->>Journal: 2. Journal Commit (Commit Record written with Barrier)
    Note over Journal: TRANSACTION IS NOW IMMUTABLE & DURABLE!
    RAM->>Disk: 3. Checkpointing (Write Inode & Bitmaps to Home locations)
    RAM->>Journal: 4. Transaction Release (Free space in Journal ring buffer)
```

---

### Crash Recovery Dynamics:
- **Crash before Step 2 (Commit)**: The transaction is incomplete. JBD2 discards the pending log on boot. Zero corruption.
- **Crash after Step 2 (Commit)**: The transaction is committed. JBD2 **replays the journal log**, writing the committed metadata directly to home storage blocks in $< 500\text{ ms}$ (**Journal Replay**).

---

## The Three Ext4 Journaling Modes

Controlled via mount option `mount -o data=<mode>`:

```mermaid
flowchart TD
    subgraph Modes ["ext4 Journaling Modes"]
        JournalMode["1. data=journal<br/>• Both User Data (D) and Metadata (I, B) are written to Journal first.<br/>• Pros: Absolute highest data integrity.<br/>• Cons: Slowest (2x Write Amplification - doubles SSD write wear)."]
        
        OrderedMode["2. data=ordered (LINUX DEFAULT)<br/>• Only Metadata is journaled.<br/>• HARD INVARIANT: User Data (D) is written to disk BEFORE Metadata commits!<br/>• Pros: Optimal performance; prevents files from reading stale garbage."]
        
        WritebackMode["3. data=writeback<br/>• Only Metadata is journaled.<br/>• No write ordering between User Data and Metadata.<br/>• Pros: Maximum raw throughput.<br/>• Cons: Power cut can leave files containing deleted previous files!"]
    end
```

---

## Journaling Modes Comparison

| Journal Mode | Metadata Journaled? | Data Journaled? | Crash Guarantee | Performance |
| :--- | :--- | :--- | :--- | :--- |
| **`data=journal`** | **Yes** | **Yes** | No data or metadata loss. | **Slowest** ($2\times$ disk I/O) |
| **`data=ordered`** | **Yes** | **No** (Direct to home) | Consistent metadata; no stale data leaks. | **Optimal (Default)** |
| **`data=writeback`** | **Yes** | **No** (Unordered) | Consistent metadata; possible stale garbage. | **Fastest** |

---

## Production Focus: `fsck` vs Journal Replay

Before journaling existed (ext2 / classic UFS), any ungraceful shutdown forced a full **`fsck` (File System Consistency Check)** scan:

```mermaid
flowchart LR
    subgraph FSCK_Legacy ["Legacy ext2 fsck Scan"]
        Scan["Scan EVERY Inode and Block on 16 TB Disk (Takes 4-12 HOURS!)"]
    end

    subgraph Journal_Replay ["Modern ext4 Journal Replay"]
        Replay["Scan 128 MB Journal Log Only (Takes < 1 SECOND!)"]
    end
```

---

## Production Diagnostics & Storage Commands

```bash
# 1. Verify filesystem has an active journal enabled
sudo tune2fs -l /dev/nvme0n1p1 | grep -i "features"
# Filesystem features: has_journal ext_attr resize_inode dir_index filetype extent 64bit

# 2. Inspect active journaling mount mode
mount | grep ext4
# /dev/nvme0n1p1 on / type ext4 (rw,relatime,data=ordered)

# 3. Force non-destructive read-only filesystem integrity check
sudo e2fsck -fn /dev/nvme0n1p1
```

---

## Active Recall & Interview Perspective

### Reasoning Prompts
1. *Why does `data=ordered` mode enforce that user data blocks must be written to disk BEFORE the metadata transaction commits to the journal?*
   - **Answer**: If metadata (the Inode containing the updated file size and block pointers) were committed to the journal *before* the user data payload was physically written to disk, a power outage occurring immediately after the commit would leave the file pointing to uninitialized storage sectors. Upon reboot and journal replay, the file would successfully report its new size but its contents would expose **stale garbage data or previously deleted files from other users** (a catastrophic data privacy vulnerability). `data=ordered` guarantees that pointers only link to blocks that are already persisted on disk.
2. *What is the difference between Journal Checkpointing and Journal Commit?*
   - **Answer**: **Journal Commit** occurs when the transaction descriptor and dirty metadata blocks are written to the sequential on-disk *journal log* and sealed with a commit record and hardware barrier (`blkdev_issue_flush`), rendering the transaction durable. **Journal Checkpointing** is the subsequent background step where the committed metadata changes are copied from the journal into their permanent on-disk *home storage locations* (the actual Inode Table and Allocation Bitmaps). Once checkpointing completes, the transaction is evicted from the circular journal buffer.
3. *Why does Write-Ahead Logging (WAL) in databases operate nearly identically to filesystem journaling?*
   - **Answer**: Both address the fundamental problem of atomic multi-block disk updates in the presence of power crashes. Both convert random multi-location disk writes into a single, high-speed sequential write to an append-only log. In both systems, a transaction is considered officially committed the moment the sequential log record hits durable media, allowing expensive in-place data updates to be deferred to asynchronous background checkpointing workers.

---

## Key Takeaways
- Appending data requires updating **Data ($D$)**, **Inode ($I$)**, and **Bitmap ($B$)**; failure midway creates **Crash Inconsistency**.
- **Write-Ahead Logging (JBD2)** commits metadata transactions to a sequential log before modifying on-disk home blocks.
- **`data=ordered`** is the production sweet spot: journaling metadata while ordering data writes to prevent stale data leaks.

---

## Related Notes
- [[Operating System]] — Storage subsystem architecture.
- [[File Concept and File Attributes]] — File metadata.
- [[Inodes and File System Metadata]] — Inode structures and bitmaps.
- [[Virtual File System - VFS]] — VFS I/O dispatch.
- [[Page Cache and Buffer Cache]] — Dirty page writeback.
- [[ext4 Architecture Overview]] — Deep dive into ext4 JBD2 mechanics.
- [[XFS and ZFS Overview]] — Copy-on-Write alternative to journaling.
- [[Operating Systems MOC]] — Master table of contents for Operating Systems.
