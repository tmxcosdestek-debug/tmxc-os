# TMXC OS - Microkernel Architecture Specification

**Document Version:** 1.0  
**Date:** 28 Eylül 2026  
**Author:** Systems Software Architect  
**Status:** Draft for Review  

---

## 1. Executive Summary

TMXC OS is a microkernel operating system designed for ARM64 (AArch64) architecture. The architecture follows strict microkernel principles with minimal kernel space and maximum service isolation in userspace.

**Design Philosophy:**
- **Minimal kernel:** Only essential services in kernel space
- **Maximum isolation:** All non-essential services in userspace
- **Clear separation:** Well-defined boundaries between layers
- **IPC-based communication:** All inter-process communication via kernel-mediated IPC
- **Security-first:** Memory isolation, capability-based security

**Target Hardware:**
- ARM64 (AArch64) processors
- EL1 (Kernel) and EL0 (Userspace) exception levels
- MMU with 4KB page tables
- Generic Timer, GIC interrupt controller

---

## 2. Architecture Overview

### 2.1 System Layers

```
┌─────────────────────────────────────────────────────────────┐
│                     Userspace Layer (EL0)                    │
├─────────────────────────────────────────────────────────────┤
│  Applications  │  Filesystem  │  Network  │  Graphics  │  UI  │
├─────────────────────────────────────────────────────────────┤
│                   System Call Interface (SVC)               │
├─────────────────────────────────────────────────────────────┤
│                     Kernel Layer (EL1)                       │
├─────────────────────────────────────────────────────────────┤
│  Process Mgmt  │  Memory Mgmt  │  IPC  │  Drivers  │  HAL  │
├─────────────────────────────────────────────────────────────┤
│                   Hardware Abstraction Layer                  │
├─────────────────────────────────────────────────────────────┤
│                     Hardware Layer                           │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Kernel Space Components

**Essential Kernel Services (≤ 12 files):**
1. **Boot Sequence** (`boot.S`) - ARM64 boot entry
2. **Kernel Main** (`kernel_main.c`) - Main initialization
3. **MMU** (`mmu.c/h`) - Virtual memory management
4. **Memory Manager** (`memory.c/h`) - Physical memory allocation
5. **Process Manager** (`process.c/h`) - Process creation/scheduling
6. **Exception Handler** (`exceptions.c/h`) - Interrupt/exception handling
7. **System Call Handler** (`syscall.c/h`) - Userspace interface
8. **IPC** (`ipc.c/h`) - Inter-process communication
9. **UART Driver** (`uart.c/h`) - Serial I/O
10. **Timer Driver** (`timer.c/h`) - System timer
11. **GIC Driver** (`gic.c/h`) - Interrupt controller
12. **HAL** (`hal.c/h`) - Hardware abstraction

**Total Kernel Files:** 12 (target)

### 2.3 Userspace Services

**Userspace Daemons (Future):**
1. **Filesystem Daemon** - VFS and filesystem implementations
2. **Network Daemon** - TCP/IP stack
3. **Graphics Daemon** - Display server
4. **Audio Daemon** - Audio server
5. **Security Daemon** - Authentication/authorization

---

## 3. Memory Architecture

### 3.1 Physical Memory Layout

```
0x0000_0000 ┌─────────────────────────────────────────┐
            │         Reserved / Firmware            │
0x8000_0000 ├─────────────────────────────────────────┤
            │         Kernel Image (4MB)              │
            │  - Boot code (64KB)                     │
            │  - Kernel text (2MB)                    │
            │  - Kernel data (1MB)                    │
            │  - Kernel BSS (512KB)                   │
            │  - Kernel stack (512KB)                 │
0x8040_0000 ├─────────────────────────────────────────┤
            │         Page Tables (4MB)               │
0x8080_0000 ├─────────────────────────────────────────┤
            │         Physical Memory Map             │
            │  - Device memory (MMIO)                 │
            │  - Reserved regions                     │
0x8100_0000 ├─────────────────────────────────────────┤
            │         Free Physical Memory             │
            │  - Userspace allocations                │
            │  - Kernel allocations                  │
            └─────────────────────────────────────────┘
```

### 3.2 Virtual Memory Layout

```
0xFFFF_0000_0000_0000 ┌─────────────────────────────────────────┐
                       │         Kernel Space (High Memory)       │
                       │  - Kernel code (0xFFFF_0000_8000_0000)   │
                       │  - Kernel data (0xFFFF_0000_8040_0000)   │
                       │  - Kernel stack (0xFFFF_0000_8080_0000)   │
0xFFFF_0000_8000_0000 ├─────────────────────────────────────────┤
                       │         Device Memory (MMIO)             │
                       │  - UART registers                       │
                       │  - Timer registers                      │
                       │  - GIC registers                        │
0xFFFF_0000_9000_0000 ├─────────────────────────────────────────┤
                       │         Userspace Space (Low Memory)     │
                       │  - Process 0 (init)                      │
                       │  - Process 1 (shell)                     │
                       │  - Process 2 (fs daemon)                 │
                       │  - Process 3 (network daemon)            │
0x0000_0000_4000_0000 ├─────────────────────────────────────────┤
                       │         Reserved                         │
0x0000_0000_0000_0000 └─────────────────────────────────────────┘
```

### 3.3 Page Table Structure

**ARM64 4-Level Page Tables:**
- **L0:** Page Global Directory (PGD) - 512 entries, 512GB each
- **L1:** Page Upper Directory (PUD) - 512 entries, 1GB each
- **L2:** Page Directory (PD) - 512 entries, 2MB each
- **L3:** Page Table (PT) - 512 entries, 4KB each

**Page Attributes:**
- **Normal memory:** Cacheable, shareable
- **Device memory:** Non-cacheable, device-nGnRnE
- **Kernel pages:** Read-only text, Read-write data
- **Userspace pages:** Read-only text, Read-write data, NX on data

---

## 4. Process Architecture

### 4.1 Process Structure

```c
typedef struct {
    uint64_t pid;                    /* Process ID */
    uint64_t ppid;                   /* Parent Process ID */
    uint64_t state;                  /* Process state */
    uint64_t priority;               /* Scheduling priority */
    
    /* Virtual memory */
    uint64_t pgd;                    /* Page Global Directory */
    uint64_t code_start;             /* Code segment start */
    uint64_t code_end;               /* Code segment end */
    uint64_t data_start;             /* Data segment start */
    uint64_t data_end;               /* Data segment end */
    uint64_t heap_start;             /* Heap start */
    uint64_t heap_end;               /* Heap end */
    uint64_t stack_top;              /* Stack top */
    
    /* Registers */
    uint64_t x0_x30[31];             /* General purpose registers */
    uint64_t sp;                     /* Stack pointer */
    uint64_t pc;                     /* Program counter */
    uint64_t cpsr;                   /* Current program status */
    
    /* Scheduling */
    uint64_t time_slice;             /* Time slice */
    uint64_t cpu_time;               /* CPU time used */
    
    /* IPC */
    uint64_t message_queue;          /* Message queue */
    uint64_t signal_mask;            /* Signal mask */
    
    /* Security */
    uint64_t capabilities;           /* Capability mask */
    uint64_t uid;                    /* User ID */
    uint64_t gid;                    /* Group ID */
} tmxc_process_t;
```

### 4.2 Process States

```
┌─────────────┐
│   CREATED   │
└──────┬──────┘
       │ fork()
       ▼
┌─────────────┐
│   READY     │ ◄─────┐
└──────┬──────┘       │
       │ schedule()   │
       ▼              │ time_slice
┌─────────────┐       │ exhausted
│  RUNNING    │───────┘
└──────┬──────┘
       │ yield() / preemption
       ▼
┌─────────────┐
│  BLOCKED    │
└──────┬──────┘
       │ I/O complete / signal
       ▼
┌─────────────┐
│   READY     │
└─────────────┘

┌─────────────┐
│  TERMINATED │
└─────────────┘
```

### 4.3 Scheduling Algorithm

**Round-Robin with Priority:**
- **Priority levels:** 0 (highest) to 7 (lowest)
- **Time quantum:** 10ms per priority level
- **Preemption:** Timer interrupt at quantum expiration
- **Aging:** Increase priority of starving processes

---

## 5. IPC Architecture

### 5.1 IPC Mechanism

**Message Passing:**
```c
typedef struct {
    uint64_t sender_pid;            /* Sender process ID */
    uint64_t receiver_pid;          /* Receiver process ID */
    uint64_t message_type;          /* Message type */
    uint64_t message_size;          /* Message size */
    uint64_t message_data[16];      /* Message data (128 bytes) */
} tmxc_message_t;
```

**IPC Operations:**
- `send(pid, message)` - Send message to process
- `receive(pid, message)` - Receive message from process
- `call(pid, message)` - Synchronous RPC
- `reply(pid, message)` - Reply to RPC

### 5.2 IPC Security

**Capability-Based Security:**
- Each process has capability mask
- IPC only allowed if capability present
- Kernel validates all IPC operations

**Capability Types:**
- `CAP_IPC_SEND` - Send messages
- `CAP_IPC_RECEIVE` - Receive messages
- `CAP_IPC_CALL` - Make RPC calls
- `CAP_IPC_REPLY` - Reply to RPC

---

## 6. System Call Interface

### 6.1 System Call Numbers

```c
#define TMXC_SYSCALL_EXIT        0
#define TMXC_SYSCALL_FORK       1
#define TMXC_SYSCALL_EXEC       2
#define TMXC_SYSCALL_WAIT       3
#define TMXC_SYSCALL_YIELD      4
#define TMXC_SYSCALL_SEND       5
#define TMXC_SYSCALL_RECEIVE    6
#define TMXC_SYSCALL_CALL       7
#define TMXC_SYSCALL_REPLY      8
#define TMXC_SYSCALL_MMAP       9
#define TMXC_SYSCALL_MUNMAP     10
#define TMXC_SYSCALL_BRK        11
#define TMXC_SYSCALL_OPEN       12
#define TMXC_SYSCALL_CLOSE      13
#define TMXC_SYSCALL_READ       14
#define TMXC_SYSCALL_WRITE      15
#define TMXC_SYSCALL_IOCTL      16
```

### 6.2 System Call Convention

**ARM64 System Call (SVC):**
- **Syscall number:** x8
- **Arguments:** x0-x5
- **Return value:** x0
- **Error indication:** x0 = -errno

**Example:**
```assembly
mov x8, #TMXC_SYSCALL_WRITE
mov x0, #fd          /* file descriptor */
mov x1, #buffer      /* buffer address */
mov x2, #count       /* byte count */
svc #0               /* invoke system call */
/* result in x0 */
```

---

## 7. Boot Sequence

### 7.1 Boot Flow

```
┌─────────────────────────────────────────────────────────────┐
│  1. Bootloader Entry (EL2 or EL3)                          │
│     - Check exception level                                 │
│     - Configure system registers                           │
│     - Drop to EL1                                          │
├─────────────────────────────────────────────────────────────┤
│  2. Kernel Entry (EL1)                                     │
│     - Set up stack pointer                                 │
│     - Clear BSS                                            │
│     - Enable MMU                                           │
├─────────────────────────────────────────────────────────────┤
│  3. Kernel Initialization                                  │
│     - Initialize UART (for debug output)                    │
│     - Initialize MMU (page tables)                          │
│     - Initialize exceptions (vector table)                  │
│     - Initialize timer (system timer)                      │
│     - Initialize GIC (interrupt controller)                 │
│     - Initialize memory manager (physical allocator)        │
│     - Initialize process manager (scheduler)                │
├─────────────────────────────────────────────────────────────┤
│  4. Userspace Initialization                               │
│     - Create init process (PID 0)                          │
│     - Load init binary                                     │
│     - Jump to userspace                                    │
├─────────────────────────────────────────────────────────────┤
│  5. System Running                                         │
│     - Scheduler running                                    │
│     - Interrupts enabled                                   │
│     - Userspace processes executing                        │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Boot Timing

**Target Boot Times:**
- Bootloader to kernel: < 50ms
- Kernel initialization: < 100ms
- Userspace init: < 50ms
- **Total boot time:** < 200ms

---

## 8. Security Model

### 8.1 Memory Isolation

**MMU Enforcement:**
- EL0 cannot access EL1 memory
- Userspace processes cannot access each other's memory
- Kernel memory is read-only for userspace
- NX (No-Execute) on data pages

### 8.2 Capability-Based Security

**Capability System:**
- Each process has capability mask
- System calls check capabilities
- IPC operations check capabilities
- File operations check capabilities

**Capability Types:**
- `CAP_PROCESS` - Process management
- `CAP_MEMORY` - Memory management
- `CAP_IPC` - IPC operations
- `CAP_FILE` - File operations
- `CAP_DEVICE` - Device access

### 8.3 Stack Protection

**Stack Canaries:**
- Random canary at function prologue
- Canary check at function epilogue
- Panic on canary mismatch

**Guard Pages:**
- Guard page below stack
- Guard page above heap
- Guard page between processes

---

## 9. Driver Architecture

### 9.1 Minimal Driver Set

**Essential Drivers (Kernel Space):**
1. **UART Driver** - Serial I/O (debug)
2. **Timer Driver** - System timer
3. **GIC Driver** - Interrupt controller
4. **GPIO Driver** - GPIO access
5. **I2C Driver** - I2C bus
6. **SPI Driver** - SPI bus

**Optional Drivers (Userspace):**
1. **Display Driver** - Graphics
2. **Audio Driver** - Audio
3. **Network Driver** - Ethernet/WiFi
4. **Storage Driver** - SD/eMMC
5. **Sensor Drivers** - Accelerometer, gyroscope, etc.

### 9.2 Driver Interface

**Standard Driver Interface:**
```c
typedef struct {
    const char* name;
    int (*init)(void);
    int (*read)(void* buffer, size_t size);
    int (*write)(const void* buffer, size_t size);
    int (*ioctl)(int cmd, void* arg);
    void (*cleanup)(void);
} tmxc_driver_t;
```

---

## 10. Build System

### 10.1 Directory Structure

```
tmxc_os/
├── boot/
│   └── boot.S
├── kernel/
│   ├── kernel_main.c
│   ├── kernel_main.h
│   ├── boot.S
│   ├── mmu.c
│   ├── mmu.h
│   ├── memory.c
│   ├── memory.h
│   ├── process.c
│   ├── process.h
│   ├── exceptions.c
│   ├── exceptions.h
│   ├── syscall.c
│   ├── syscall.h
│   ├── ipc.c
│   ├── ipc.h
│   ├── uart.c
│   ├── uart.h
│   ├── timer.c
│   ├── timer.h
│   ├── gic.c
│   ├── gic.h
│   ├── hal.c
│   └── hal.h
├── lib/
│   ├── string.c
│   ├── string.h
│   ├── stdio.c
│   └── stdio.h
├── Makefile
└── linker.ld
```

### 10.2 Build Targets

**Makefile Targets:**
- `all` - Build kernel binary
- `clean` - Clean build artifacts
- `run` - Run in QEMU
- `debug` - Run in QEMU with GDB
- `info` - Show kernel information

---

## 11. Testing Strategy

### 11.1 Unit Testing

**Test Coverage:**
- Boot sequence
- MMU operations
- Memory allocation
- Process creation
- System calls
- IPC operations

### 11.2 Integration Testing

**Test Scenarios:**
- Multi-process scheduling
- Inter-process communication
- Memory isolation
- Interrupt handling
- Driver operations

### 11.3 Performance Testing

**Performance Metrics:**
- Boot time (< 200ms)
- Context switch time (< 10μs)
- IPC latency (< 5μs)
- Memory allocation time (< 1μs)

---

## 12. Documentation Requirements

### 12.1 Technical Documentation

**Required Documents:**
1. **Boot Sequence Documentation** - Detailed boot flow
2. **Memory Layout Documentation** - Physical/virtual memory
3. **API Documentation** - System calls and kernel APIs
4. **Architecture Documentation** - Overall architecture
5. **Build System Documentation** - Build instructions

### 12.2 Code Documentation

**Documentation Standards:**
- English comments only
- Doxygen-style documentation
- Function headers with purpose, parameters, return values
- Inline comments for complex logic

---

## 13. Success Criteria

### 13.1 Code Quality

**Metrics:**
- Kernel files: ≤ 12
- Total LOC (kernel): ≤ 5,000
- Code coverage: ≥ 80%
- Static analysis: 0 warnings

### 13.2 Build Quality

**Metrics:**
- Compilation: Error-free
- Linking: Error-free
- Boot: Successful
- Tests: All passing

### 13.3 Architecture Quality

**Metrics:**
- Microkernel compliance: 100%
- Layer separation: Clear
- IPC-based: All communication
- Security: Memory isolation verified

---

## 14. Implementation Timeline

### Phase 1: Core Infrastructure (Week 1-2)
- Boot sequence
- UART driver
- MMU setup
- Exception handling

### Phase 2: Memory & Process (Week 3-4)
- Memory manager
- Process manager
- Scheduler
- System calls

### Phase 3: IPC & Drivers (Week 5-6)
- IPC mechanism
- Timer driver
- GIC driver
- HAL

### Phase 4: Testing & Documentation (Week 7-8)
- Unit testing
- Integration testing
- Performance testing
- Documentation

---

## 15. Conclusion

This architecture specification defines a clean, enterprise-grade microkernel for TMXC OS. The architecture follows strict microkernel principles with minimal kernel space and maximum service isolation.

**Key Principles:**
- Minimal kernel (≤ 12 files)
- Clear layer separation
- IPC-based communication
- Security-first design
- Professional documentation

**Next Steps:**
1. Review and approve architecture specification
2. Archive existing codebase
3. Implement core infrastructure
4. Test thoroughly
5. Document comprehensively

**Status:** Ready for implementation upon approval.
