---
title: Solid State Drives - Flash Memory, Wear Leveling, TRIM
subject: Operating Systems
module: Storage & I/O Subsystems
difficulty: Advanced
prerequisites:
  - "[[Operating System]]"
  - "[[Kernel]]"
  - "[[File Concept and File Attributes]]"
  - "[[Inodes and File System Metadata]]"
  - "[[IO Hardware - Port-Mapped vs Memory-Mapped IO]]"
  - "[[Direct Memory Access - DMA]]"
related:
  - "[[Disk Scheduling Algorithms - SSTF, SCAN, C-SCAN]]"
  - "[[RAID Levels and Reliability]]"
  - "[[ext4 Architecture Overview]]"
aliases:
  - Solid State Drives - Flash Memory, Wear Leveling, TRIM
  - Solid State Drives
  - SSD Architecture
  - NAND Flash
  - Flash Translation Layer
  - FTL
  - Wear Leveling
  - Write Amplification Factor
  - WAF
  - TRIM Command
  - NVMe Deallocate
tags:
  - os
  - storage
  - hardware
  - ssd
  - nvme
  - linux-kernel
  - performance
status: complete
---

# Solid State Drives: Flash Memory, Wear Leveling, TRIM

> [!abstract] Mental Model
> NAND Flash memory is **writing in permanent ink on an Etch-a-Sketch**:
> - You can write words on individual lines (**$4\text{ KB}$ Page Programming**).
> - But you **cannot erase a single word**—you can only shake the entire Etch-a-Sketch clean (**$4\text{ MB}$ Block Erase**).
> - Because flash memory cannot overwrite in-place, SSDs run an internal embedded operating system—the **Flash Translation Layer (FTL)**—which handles out-of-place writes, **Garbage Collection**, **Wear Leveling**, and relies on the **TRIM command** to avoid self-destructive write amplification.

---

## The Fundamental Asymmetry of NAND Flash

```mermaid
flowchart TD
    subgraph PhysicalHierarchy ["NAND Flash Physical Structure"]
        Block["Erase Block (e.g. 4 MB = 1024 Pages)"]
        Page0["Page 0 (4 KB) [Read / Write Unit]"]
        Page1["Page 1 (4 KB) [Read / Write Unit]"]
        PageN["Page 1023 (4 KB) [Read / Write Unit]"]

        Block --> Page0 & Page1 & PageN
    end

    subgraph Operations ["Flash Physical Operational Constraints"]
        ReadOp["1. Read: 4 KB Page Granularity (~10-25 μs)"]
        ProgOp["2. Program (Write): 4 KB Page Granularity (~100 μs)<br/>• Can only flip bits from 1 -> 0!"]
        EraseOp["3. Erase: MUST ERASE ENTIRE 4 MB BLOCK (~2-5 ms)!<br/>• High reverse voltage resets all bits to 1."]
    end
```

> **The In-Place Write Prohibition**: To update 1 byte in Page 0, the controller **cannot rewrite Page 0 directly**. It must write the new data to an unused empty Page, update its internal mapping table, and mark old Page 0 as **Invalid (Dead Data)**.

---

## NAND Flash Cell Taxonomy & Endurance

```mermaid
flowchart LR
    SLC["SLC (1 bit/cell)<br/>2 Voltage States<br/>100,000 P/E Cycles<br/>Ultra-high endurance"]
    --> MLC["MLC (2 bits/cell)<br/>4 Voltage States<br/>3,000 - 10,000 P/E<br/>Industrial grade"]
    --> TLC["TLC (3 bits/cell)<br/>8 Voltage States<br/>1,000 - 3,000 P/E<br/>Enterprise NVMe Standard"]
    --> QLC["QLC (4 bits/cell)<br/>16 Voltage States<br/>100 - 1,000 P/E<br/>Cold read storage"]
```

---

## The Flash Translation Layer (FTL) & Out-of-Place Writes

The **FTL** is an embedded controller running firmware inside the SSD that presents a virtual flat disk interface (LBA) to the host OS:

```mermaid
flowchart TD
    OS_Write["OS issues write(LBA 500, newData)"] --> FTL["Flash Translation Layer (FTL)"]
    
    subgraph FTL_Engine ["FTL In-RAM Mapping Table"]
        MapOld["LBA 500 -> Physical Block 10, Page 3 [MARK INVALID]"]
        MapNew["LBA 500 -> Physical Block 45, Page 0 [WRITE NEW DATA]"]
    end

    FTL --> FTL_Engine
```

---

## Garbage Collection & Write Amplification Factor (WAF)

When free empty blocks run out, the FTL must perform **Garbage Collection (GC)**:
1. Select a block filled with mixed valid and invalid pages.
2. Read the valid pages into SSD internal RAM.
3. Write (re-program) valid pages into a new clean block.
4. Issue a high-voltage erase to the entire original block.

```mermaid
flowchart TD
    subgraph GarbageCollection ["In-Drive Garbage Collection Cycle"]
        OldBlock["Original Block 10<br/>• 200 Valid Pages<br/>• 824 Invalid Pages"]
        Copy["Read 200 Valid Pages to Controller RAM"]
        NewBlock["Write 200 Valid Pages into New Block 50"]
        Erase["Erase Block 10 -> Now 1024 Clean Pages!"]

        OldBlock --> Copy --> NewBlock --> Erase
    end
```

---

### Write Amplification Factor (WAF):
$$\mathbf{\text{WAF} = \frac{\text{Total Bytes Written to NAND Flash}}{\text{Total Bytes Written by Host OS}}}$$

- **Ideal WAF = 1.0**: For every 1 GB written by OS, 1 GB hits NAND.
- **Pathological WAF = 5.0 to 10.0**: Under random-write database loads on a nearly full SSD, moving valid pages during GC multiplies write volume by $10\times$, destroying SSD IOPS and burning through silicon endurance in months!

---

## Wear Leveling: Dynamic vs Static

Because each NAND flash block degrades physically after a fixed number of **Program/Erase (P/E) cycles** (due to oxide layer breakdown), the FTL distributes writes uniformly:

```mermaid
flowchart TD
    subgraph DynamicWL ["1. Dynamic Wear Leveling"]
        D1["Directs new incoming writes to blocks with the LOWEST erase counts."]
        D2["Flaw: Blocks holding read-only static files (OS binaries) NEVER get erased or cycled!"]
        D1 --- D2
    end

    subgraph StaticWL ["2. Static Wear Leveling (Comprehensive)"]
        S1["Monitors cold, undisturbed blocks."]
        S2["Actively moves cold static data OUT of low-wear blocks into worn blocks."]
        S3["Frees low-wear blocks to absorb hot, incoming write traffic."]
        S1 --> S2 --> S3
    end
```

---

## The TRIM / NVMe Deallocate Command

### Why Standard File Deletion Destroys SSDs:
When a user deletes a $10\text{ GB}$ file (`rm video.mp4`), the OS filesystem only clears the Inode bitmap in OS RAM. The OS **never issues an I/O write to the storage controller** to say the data is dead. The SSD still thinks those LBAs contain critical data, and continues endlessly copying dead data during Garbage Collection!

### The TRIM Solution:

```mermaid
sequenceDiagram
    autonumber
    participant OS as Linux Filesystem (ext4/XFS)
    participant FTL as SSD Controller (FTL)
    participant NAND as Physical Flash Blocks

    OS->>OS: User runs: rm large_file.bin
    OS->>FTL: Issues ATA TRIM / NVMe Deallocate command (LBAs 1000..50000)
    Note over FTL: FTL marks physical pages for LBAs 1000..50000 as INVALID!
    Note over FTL: Garbage Collection can now instant-erase those blocks WITHOUT copying!
    FTL-->>NAND: Lowers WAF to near 1.0 & restores full drive write speed!
```

---

## Production Diagnostics & Flash Monitoring

```bash
# 1. Inspect NVMe Health, Wear Level, and Spare Capacity
sudo nvme smart-log /dev/nvme0n1

# Output metrics:
# critical_warning                    : 0
# temperature                         : 38 C
# available_spare                     : 100%
# percentage_used                     : 3% (Drive has consumed 3% of total rated lifespan)
# data_units_written                  : 48,192,014 (approx 24.6 TB written)

# 2. Trigger On-Demand Online TRIM across all mounted filesystems:
sudo fstrim -av

# Output:
# /: 42.1 GiB (45210291200 bytes) trimmed on /dev/nvme0n1p1
# /data: 110.8 GiB (118921004032 bytes) trimmed on /dev/nvme1n1p1

# 3. Check SSD Over-Provisioning and SMART attributes on SATA SSD:
sudo smartctl -A /dev/sda | grep -E "Wear_Range_Delta|Used_Rsvd_Blk_Cnt_Tot"
```

---

## Active Recall & Interview Perspective

### Reasoning Prompts
1. *What is the Flash Translation Layer (FTL) and why does an SSD require it while a mechanical hard drive does not?*
   - **Answer**: Mechanical hard drives allow random overwrites directly in-place at sector granularity ($512\text{B} / 4096\text{B}$). NAND flash memory physically *cannot overwrite data in-place*—it can only write at the Page level ($4\text{ KB}$) and erase at the Block level ($4\text{ MB}$). The **Flash Translation Layer (FTL)** is an embedded microcontroller subsystem that translates logical OS block addresses (LBA) to dynamic physical flash pages (PPA), performing out-of-place writes, tracking invalid pages, executing background garbage collection, and balancing cell wear leveling transparently to the host operating system.
2. *Why is Static Wear Leveling superior to Dynamic Wear Leveling?*
   - **Answer**: **Dynamic Wear Leveling** only selects blocks with low erase counts when writing *new incoming data*. If a portion of the drive stores static, read-only data (such as OS system files or game assets), those blocks remain untouched with near-zero erase counts, while the remainder of the drive absorbs all write traffic and wears out prematurely. **Static Wear Leveling** detects blocks containing cold, unwritten data, relocates that static data into heavily-worn blocks, and frees up the pristine, low-wear blocks to participate in active write cycles, ensuring $100\%$ of flash cells degrade at an identical rate.
3. *What is the TRIM / NVMe Deallocate command and what happens to SSD performance if it is disabled?*
   - **Answer**: When an OS deletes a file, it only updates filesystem metadata; it sends zero write requests to the underlying storage sectors. Without **TRIM**, the SSD controller cannot distinguish between active user files and deleted file sectors. During background Garbage Collection, the SSD dutifully copies all dead, deleted pages into new blocks before erasing old blocks, driving the **Write Amplification Factor (WAF)** to extreme levels ($5\times \dots 10\times$). This causes write performance to collapse to single-digit megabytes per second and exhausts the drive's P/E endurance cycles prematurely. TRIM notifies the FTL that deleted LBA ranges are dead, allowing GC to skip copying them and erase blocks instantly.

---

## Key Takeaways
- Flash memory has an intrinsic asymmetry: **Read/Program at $4\text{ KB}$ Page size**, but **Erase at $4\text{ MB}$ Block size**.
- The **FTL** handles out-of-place writes, mapping tables, and **Static Wear Leveling**.
- **TRIM / NVMe Deallocate** informs the controller of deleted blocks, minimizing **Write Amplification (WAF)** and preserving SSD lifespan.

---

## Related Notes
- [[Operating System]] — Storage subsystem architecture.
- [[IO Hardware - Port-Mapped vs Memory-Mapped IO]] — NVMe PCIe BARs.
- [[Direct Memory Access - DMA]] — High-throughput NVMe DMA queues.
- [[Disk Scheduling Algorithms - SSTF, SCAN, C-SCAN]] — `blk-mq` `none` scheduler.
- [[RAID Levels and Reliability]] — SSD array wear and rebuilds.
- [[ext4 Architecture Overview]] — Ext4 extent allocation on flash.
- [[Operating Systems MOC]] — Master table of contents for Operating Systems.
