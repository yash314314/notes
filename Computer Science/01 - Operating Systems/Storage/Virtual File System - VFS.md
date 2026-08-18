---
title: Virtual File System - VFS
subject: Operating Systems
module: File Systems
difficulty: Advanced
prerequisites:
  - "[[Operating System]]"
  - "[[Kernel]]"
  - "[[System Calls]]"
  - "[[File Concept and File Attributes]]"
  - "[[File Descriptors and File Tables]]"
  - "[[Directory Structures and Path Resolution]]"
  - "[[Inodes and File System Metadata]]"
related:
  - "[[Page Cache and Buffer Cache]]"
  - "[[Journaling File Systems and Crash Consistency]]"
  - "[[ext4 Architecture Overview]]"
  - "[[XFS and ZFS Overview]]"
aliases:
  - Virtual File System - VFS
  - Virtual File System
  - VFS
  - file_operations
  - inode_operations
  - super_operations
  - dentry_operations
  - procfs
  - sysfs
tags:
  - os
  - storage
  - filesystem
  - linux-kernel
  - architecture
  - posix
status: complete
---

# Virtual File System (VFS)

> [!abstract] Mental Model
> The VFS is the **Universal Polymorphic Plug-and-Play Adapter of the Operating System**:
> - An application calls `read(fd, buf, 1024)`. It does not know—and should not care—whether `fd` points to an **ext4 NVMe drive**, an **XFS storage array**, an **NFS network share in Germany**, a **FAT32 USB stick**, or a virtual kernel telemetry file (**`/proc/cpuinfo`**).
> - The **Virtual File System (VFS)** is the kernel's object-oriented abstraction layer that translates uniform POSIX system calls into filesystem-specific drivers via **C function pointer dispatch tables**.

---

## VFS Architectural Placement

```mermaid
flowchart TD
    UserApp["User Applications (C / Go / Python / Java)"]
    
    subgraph PosixAPI ["Standard POSIX System Calls"]
        SysCalls["open() | read() | write() | lseek() | stat() | fsync()"]
    end

    subgraph VFS_Layer ["Linux Virtual File System (VFS Layer)"]
        VFS_Core["VFS Common Model (dcache, inode cache, mount tree)"]
    end

    subgraph FileSystemDrivers ["Concrete Filesystem Drivers (Polymorphic Implementations)"]
        ext4["ext4 Driver"]
        XFS["XFS Driver"]
        NFS["NFS Network Driver"]
        ProcFS["procfs / sysfs (RAM Pseudo-FS)"]
        tmpfs["tmpfs (In-RAM Page Cache)"]
    end

    subgraph HardwareLayer ["Underlying Targets"]
        NVMe["NVMe SSD Disk"]
        SAN["Fibre Channel SAN"]
        Network["Remote AWS Server"]
        KernelRAM["Kernel Memory Data Structures"]
    end

    UserApp --> PosixAPI
    PosixAPI --> VFS_Layer
    VFS_Layer --> ext4 --> NVMe
    VFS_Layer --> XFS --> SAN
    VFS_Layer --> NFS --> Network
    VFS_Layer --> ProcFS --> KernelRAM
    VFS_Layer --> tmpfs --> KernelRAM
```

---

## The Four Primary VFS Object Types

The VFS implements an Object-Oriented design pattern in pure C (`include/linux/fs.h`) structured around four core objects:

```mermaid
flowchart LR
    SB["1. Superblock Object (struct super_block)<br/>• Represents an entire mounted filesystem.<br/>• Operations: struct super_operations *s_op"]
    
    IN["2. Inode Object (struct inode)<br/>• Represents a specific file metadata entity.<br/>• Operations: struct inode_operations *i_op"]
    
    DE["3. Dentry Object (struct dentry)<br/>• Represents a directory entry / path link component.<br/>• Operations: struct dentry_operations *d_op"]
    
    FO["4. File Object (struct file)<br/>• Represents an active open file descriptor in memory.<br/>• Operations: struct file_operations *f_op"]

    SB --- IN --- DE --- FO
```

---

## Polymorphic Dynamic Dispatching: `struct file_operations`

When an application calls `write(fd, buffer, count)`, the VFS resolves the open descriptor to its `struct file` and dispatches through its function table:

```c
// include/linux/fs.h
struct file_operations {
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    int (*mmap) (struct file *, struct vm_area_struct *);
    int (*open) (struct inode *, struct file *);
    int (*fsync) (struct file *, loff_t, loff_t, int datasync);
    long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
};
```

```mermaid
sequenceDiagram
    autonumber
    participant App as User Application
    participant Syscall as sys_write() [Kernel Syscall Entry]
    participant VFS as VFS Dispatcher (vfs_write)
    participant Driver as Concrete Driver (ext4_file_write_iter)
    participant HW as NVMe Controller Driver

    App->>Syscall: write(fd, buf, len)
    Syscall->>VFS: vfs_write(file, buf, len, &pos)
    VFS->>Driver: file->f_op->write_iter(iocb, from) [DYNAMIC DISPATCH]
    Driver->>HW: submit_bio() [Block I/O Request]
    HW-->>App: Return bytes written
```

---

## Pseudo / Virtual Filesystems in VFS

Because VFS abstracts all I/O as files, the kernel exposes internal subsystem state through **Virtual Filesystems** that exist purely in RAM:

```mermaid
flowchart TD
    subgraph PseudoFS ["Linux In-Memory Virtual Filesystems"]
        proc["1. procfs (/proc)<br/>• Live process telemetry & kernel parameters.<br/>• Examples: /proc/cpuinfo, /proc/meminfo, /proc/<PID>/status"]
        
        sys["2. sysfs (/sys)<br/>• Hardware device hierarchy & driver kernel objects (kobjects).<br/>• Examples: /sys/block/sda, /sys/class/net/eth0"]
        
        tmp["3. tmpfs (/tmp, /dev/shm)<br/>• Ultra-fast temporary volatile storage allocated in Linux Page Cache.<br/>• Can swap to disk if physical RAM is pressurized."]
        
        dev["4. devtmpfs (/dev)<br/>• Kernel-managed device nodes populated automatically by udev."]
    end
```

---

## Filesystem Registration and Mounting

1. **Registration**: When a kernel module (e.g. `ext4.ko`) loads, it calls:
   ```c
   register_filesystem(&ext4_fs_type);
   ```
2. **Mounting**: When `mount -t ext4 /dev/sdb1 /data` is executed:
   - VFS locates `ext4_fs_type` in the registered filesystem linked list.
   - Invokes `ext4_mount()` to read the on-disk superblock and instantiate the in-memory `struct super_block`.
   - Creates a `struct vfsmount` linking the target directory dentry (`/data`) to the child filesystem root dentry.

---

## Production Diagnostics & VFS Inspection

```bash
# 1. View all filesystem types currently supported and registered in kernel
cat /proc/filesystems

# Output:
# nodev sysfs
# nodev rootfs
# nodev ramfs
# nodev proc
# nodev tmpfs
#       ext4
#       xfs
#       btrfs

# 2. Inspect filesystem type and mount options for any file or directory
findmnt -T /var/log/syslog
# TARGET SOURCE    FSTYPE OPTIONS
# /      /dev/sda1 ext4   rw,relatime,errors=remount-ro

# 3. View global VFS open file allocation count
cat /proc/sys/fs/file-nr
```

---

## Active Recall & Interview Perspective

### Reasoning Prompts
1. *How does the VFS implement Object-Oriented polymorphism in C without class inheritance?*
   - **Answer**: The VFS uses structures containing **tables of function pointers** (such as `struct file_operations`, `struct inode_operations`, and `struct super_operations`). Each generic VFS object holds a pointer to its concrete filesystem operation table. When a system call like `read()` or `mkdir()` is executed, the VFS simply invokes `file->f_op->read(...)` or `dir->i_op->mkdir(...)`. Each filesystem driver (ext4, XFS, NFS) populates these function pointers with its own custom functions at initialization time, achieving classic runtime polymorphism.
2. *What is the difference between `struct inode_operations` and `struct file_operations`?*
   - **Answer**: **`struct inode_operations`** manages operations that manipulate the *metadata and directory namespace* of a file (e.g., `lookup()`, `create()`, `link()`, `unlink()`, `mkdir()`, `rename()`, `permission()`). **`struct file_operations`** manages operations performed on the *data stream and open instances* of a file (e.g., `read()`, `write()`, `llseek()`, `mmap()`, `fsync()`, `unlocked_ioctl()`).
3. *Why do files in `/proc` (e.g. `/proc/meminfo`) report a file size of 0 bytes in `ls -l`, yet return kilobytes of data when read with `cat`?*
   - **Answer**: `/proc` is a virtual in-memory filesystem (procfs) that has no physical disk backing. The files do not contain stored data blocks, so `struct inode->i_size` is initialized to `0`. When an application calls `read()` on `/proc/meminfo`, the VFS invokes procfs's custom `read()` handler, which dynamically interrogates kernel memory telemetry data structures in real-time, formats the output as ASCII text, and copies it directly into the user application buffer.

---

## Key Takeaways
- The **Virtual File System (VFS)** provides a uniform POSIX abstraction over heterogeneous local and network filesystems.
- The 4 core VFS objects are **Superblock**, **Inode**, **Dentry**, and **File**, each driven by C function pointer operation tables.
- **procfs**, **sysfs**, and **tmpfs** leverage VFS to expose kernel runtime telemetry as standard files.

---

## Related Notes
- [[Operating System]] — Storage subsystem architecture.
- [[System Calls]] — Standard I/O syscalls.
- [[File Concept and File Attributes]] — File abstraction.
- [[File Descriptors and File Tables]] — Open file objects (`struct file`).
- [[Directory Structures and Path Resolution]] — Dentry caching (`dcache`).
- [[Inodes and File System Metadata]] — Inode operations.
- [[Page Cache and Buffer Cache]] — In-memory caching engine.
- [[Operating Systems MOC]] — Master table of contents for Operating Systems.
