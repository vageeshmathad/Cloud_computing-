# Performance Analysis of Type-1 and Type-2 Hypervisors

---

## Executive Summary

This repository contains the complete experimental setup, empirical benchmark data, performance visualization, and technical report comparing the CPU performance of a **Type-1 Bare-Metal Hypervisor (Proxmox VE)** and a **Type-2 Hosted Hypervisor (VMware Workstation)**.

Both hypervisors were deployed with identically configured **Ubuntu Virtual Machines** (2 vCPU, 2 GB RAM, 20 GB Disk, Ubuntu 22.04.5 LTS). The standard `sysbench` CPU prime-number calculation benchmark (`--cpu-max-prime=20000`) was executed on both virtual machines under identical workload conditions.

### Key Finding

> **Proxmox VE (Type-1 Hypervisor) achieved 1,587.47 Events/sec compared to VMware Workstation's 1,440.80 Events/sec — demonstrating a +10.18% throughput advantage, an 8.70% reduction in average latency, and a 27.78% reduction in 95th percentile latency.**

---

## Table of Contents

1. [Project Objectives](#1-project-objectives)
2. [Hypervisor Architectural Comparison](#2-hypervisor-architectural-comparison)
3. [Virtual Machine Specifications](#3-virtual-machine-specifications)
4. [Experimental Procedure](#4-experimental-procedure)
5. [Empirical Results & Screenshots](#5-empirical-results--screenshots)
6. [Performance Comparison Table](#6-performance-comparison-table)
7. [Metric Explanations & Visualizations](#7-metric-explanations--visualizations)
8. [Technical Analysis & Discussion](#8-technical-analysis--discussion)
9. [Conclusion & Engineering Takeaways](#9-conclusion--engineering-takeaways)
10. [Repository Structure & Reproduction](#10-repository-structure--reproduction)

---

## 1. Project Objectives

The primary objectives of this Cloud Computing laboratory experiment are:

1. **Deployment**: Provision two identical Ubuntu Virtual Machines across different hypervisor architectures:
   - **Type-1 (Bare-Metal)**: Proxmox VE (Kernel-based Virtual Machine / KVM)
   - **Type-2 (Hosted)**: VMware Workstation on a Windows Host OS
2. **Standardization**: Enforce uniform hardware resource allocations (2 vCPU, 2048 MB RAM, 20 GB Virtual Storage) to ensure direct comparability.
3. **Benchmarking**: Execute the `sysbench` CPU computational benchmark using 20,000 prime numbers to stress test CPU virtualization efficiency.
4. **Metric Collection**: Capture execution time, total events processed, throughput (events/sec), and latency statistics (min, avg, max, 95th percentile).
5. **Architectural Evaluation**: Quantify the performance overhead introduced by host operating system abstraction layers in Type-2 hypervisors versus bare-metal hypervisor execution.

---

## 2. Hypervisor Architectural Comparison

### Type-1 Hypervisor — Proxmox VE (Bare-Metal Architecture)

Proxmox VE runs directly on the bare-metal physical host hardware. The Linux kernel integrated with KVM (Kernel-based Virtual Machine) acts as the hypervisor. Guest operating system instructions execute directly on hardware CPU VT-x/AMD-V extensions without passing through an intermediate desktop operating system.

```mermaid
graph TD
    subgraph Physical_Hardware["Physical Hardware (CPU, Memory, Storage, NIC)"]
    end
    
    subgraph Type1_Layer["Proxmox VE Hypervisor (Bare-Metal OS & KVM Kernel)"]
    end
    
    subgraph Guest_VM1["Ubuntu 22.04.5 LTS Virtual Machine (vm01)"]
        Sysbench1["Sysbench CPU Benchmark"]
    end
    
    Physical_Hardware --> Type1_Layer
    Type1_Layer --> Guest_VM1
```

```
+-------------------------------------------------------------------+
|               Ubuntu Virtual Machine (Type-1 Guest)               |
+-------------------------------------------------------------------+
|               Proxmox VE Hypervisor (Linux Kernel / KVM)          |
+-------------------------------------------------------------------+
|                 Physical Server Hardware (Bare Metal)             |
+-------------------------------------------------------------------+
```

---

### Type-2 Hypervisor — VMware Workstation (Hosted Architecture)

VMware Workstation runs as an application process on top of a host operating system (Windows 11/10). CPU requests from the guest VM must navigate through the VMware VMM engine, translate through host OS system calls, and be scheduled by the Windows NT kernel scheduler before reaching physical hardware.

```mermaid
graph TD
    subgraph Physical_Hardware2["Physical Hardware (CPU, Memory, Storage, NIC)"]
    end

    subgraph Host_OS["Host Operating System (Windows 11 / Windows NT Kernel)"]
    end
    
    subgraph Type2_Layer["VMware Workstation (Type-2 Hypervisor Application)"]
    end
    
    subgraph Guest_VM2["Ubuntu Virtual Machine (nupur-virtual-machine)"]
        Sysbench2["Sysbench CPU Benchmark"]
    end
    
    Physical_Hardware2 --> Host_OS
    Host_OS --> Type2_Layer
    Type2_Layer --> Guest_VM2
```

```
+-------------------------------------------------------------------+
|               Ubuntu Virtual Machine (Type-2 Guest)               |
+-------------------------------------------------------------------+
|               VMware Workstation (Virtual Machine Monitor)        |
+-------------------------------------------------------------------+
|               Host Operating System (Windows 11 / 10)             |
+-------------------------------------------------------------------+
|                        Physical PC Hardware                       |
+-------------------------------------------------------------------+
```

---

## 3. Virtual Machine Specifications

To guarantee scientific accuracy and eliminate resource skewing, identical configurations were assigned to both VMs:

| Resource Parameter | Proxmox VE (Type-1) | VMware Workstation (Type-2) | Status |
| :--- | :--- | :--- | :--- |
| **Virtual Machine Name** | `vm01-Standard-PC-i440FX-PIIX-1996` | `nupur-virtual-machine` | Standardized |
| **Guest Operating System** | Ubuntu 22.04.5 LTS (x86_64) | Ubuntu 22.04.5 LTS (x86_64) | Identical Baseline |
| **CPU Allocation** | 2 vCPU (1 Socket, 2 Cores) | 2 vCPU (1 Processor, 2 Cores) | Identical |
| **CPU Type / Model** | `QEMU Virtual CPU version 2.5+` | `13th Gen Intel Core i5-13450HX` | Host Passthrough |
| **RAM Allocation** | 2048 MiB (2.0 GB) | 2048 MB (2.0 GB) | Identical |
| **Virtual Disk Capacity** | 20.0 GB | 20.0 GB | Identical |
| **Virtual Network Adapter**| VirtIO Bridge (`vmbr0`) | NAT | Standardized |
| **Virtualization Engine** | KVM (Full Virtualization) | VMware (Full Virtualization) | Standardized |
| **Benchmark Tool** | `sysbench 1.0.20` | `sysbench 1.0.20` | Identical |

---

## 4. Experimental Procedure

### Step 1: Virtual Machine Creation & Setup

1. **Proxmox VE (Type-1)**:
   - Accessed Proxmox VE web management console.
   - Initialized `Create VM` wizard.
   - Attached Ubuntu 22.04.5 LTS ISO, configured 2 Cores, 2048 MiB RAM, 20 GB VirtIO disk, and `vmbr0` bridge adapter.
   - Completed standard Ubuntu installation and configured the initial environment.

2. **VMware Workstation (Type-2)**:
   - Launched VMware Workstation application on Windows host.
   - Selected `Typical Configuration` wizard.
   - Mounted Ubuntu 22.04.5 LTS ISO.
   - Allocated a 20 GB virtual disk, configured 1 Processor with 2 Cores (2 vCPU total), 2 GB RAM, and NAT adapter.
   - Completed standard Ubuntu installation.

### Step 2: System Configuration Verification

On both guest OS terminals, system specifications were verified prior to running benchmarks:

```bash
# 1. Verify Hostname & System Architecture
hostnamectl

# 2. Verify CPU Topology & Core Allocation
lscpu

# 3. Verify Memory Allocation
free -h

# 4. Verify Disk Partition Allocation
df -h

# 5. Monitor Real-time Process & System Load
top
```

### Step 3: Sysbench Benchmark Installation & Execution

```bash
# Package Index Update & Sysbench Installation
sudo apt update && sudo apt install sysbench -y

# Verify Version
sysbench --version

# Execute CPU Benchmark (Prime Calculation up to 20,000)
sysbench cpu --cpu-max-prime=20000 run
```

---

## 5. Empirical Results & Screenshots

### Type-1 Hypervisor Screenshot (Proxmox VE)

Below is the verified screenshot captured directly from the Proxmox VE guest terminal:

![Proxmox VE Type-1 Sysbench Result](images/1.png)

*Figure 1: Proxmox VE (Type-1 Hypervisor) Sysbench Benchmark Console Output.*

---

### Type-2 Hypervisor Screenshot (VMware Workstation)

Below is the verified screenshot captured directly from the VMware Workstation guest terminal:

![VMware Workstation Type-2 Sysbench Result](images/2.png)

*Figure 2: VMware Workstation (Type-2 Hypervisor) Sysbench Benchmark Terminal Output.*

---

## 6. Performance Comparison Table

The following table summarizes the exact values recorded from the experimental benchmark runs:

| Performance Metric | Proxmox VE (Type-1) | VMware Workstation (Type-2) | Performance Delta | Winner / Advantage |
| :--- | :---: | :---: | :---: | :---: |
| **Hypervisor Type** | Bare-Metal | Hosted | Architectural | Type-1 Direct Control |
| **Guest OS** | Ubuntu 22.04.5 LTS | Ubuntu 22.04.5 LTS | Matched | Identical Baseline |
| **vCPU Allocation** | 2 vCPU | 2 vCPU | Matched | Identical Compute |
| **RAM Allocation** | 2 GB | 2 GB | Matched | Identical Memory |
| **Disk Capacity** | 20 GB | 20 GB | Matched | Identical Storage |
| **Benchmark Limit** | 20,000 Primes | 20,000 Primes | Matched | Identical Stress Test |
| **Total Execution Time** | **10.0005 s** | **10.0004 s** | ~0.001% difference | Fixed 10s Window |
| **Total Events Processed** | **15,877** | **14,410** | **+1,467 events (+10.18%)** | **Proxmox VE (Type-1)** |
| **Events per Second (EPS)** | **1,587.47** | **1,440.80** | **+146.67 eps (+10.18%)** | **Proxmox VE (Type-1)** |
| **Minimum Latency** | **0.59 ms** | **0.65 ms** | **-0.06 ms (-9.23%)** | **Proxmox VE (Faster)** |
| **Average Latency** | **0.63 ms** | **0.69 ms** | **-0.06 ms (-8.70%)** | **Proxmox VE (Lower)** |
| **95th Percentile Latency**| **0.65 ms** | **0.90 ms** | **-0.25 ms (-27.78%)** | **Proxmox VE (More Consistent)**|
| **Maximum Latency** | **1.34 ms** | **1.89 ms** | **-0.55 ms (-29.10%)** | **Proxmox VE (Fewer Spikes)** |

---

## 7. Metric Explanations & Visualizations

### Performance Metric Definitions

1. **Total Execution Time (seconds)**: The wall-clock duration taken to execute the Sysbench workload. Standardized to ~10 seconds.
2. **Events per Second (Throughput / EPS)**: The number of prime number calculation iterations completed per second. **Higher is better.**
3. **Total Events**: Total number of prime verification cycles executed during the test duration. **Higher is better.**
4. **Latency (milliseconds)**: Time elapsed per event execution:
   - **Minimum Latency**: The fastest event execution time.
   - **Average Latency**: Arithmetic mean of all event processing times.
   - **95th Percentile Latency**: The latency threshold below which 95% of all events fell. Critical for evaluating response consistency.
   - **Maximum Latency**: The worst-case event delay, highlighting thread scheduling latency spikes.

---

### Chart 1: CPU Throughput Comparison (Events / Sec)

![CPU Throughput Comparison](images/events_per_second_comparison.png)

*Figure 3: CPU Throughput comparison showing Proxmox VE (+10.18% faster).*

---

### Chart 2: CPU Latency Metrics Comparison

![Latency Comparison](images/latency_comparison.png)

*Figure 4: Latency comparison (Min, Avg, 95th Percentile, Max) across both hypervisors.*

---

### Chart 3: Total Events Processed

![Total Events Comparison](images/total_events_comparison.png)

*Figure 5: Total Events completed in 10 seconds (15,877 vs 14,410).*

---

### Chart 4: Comprehensive Performance Dashboard

![Overall Performance Dashboard](images/overall_performance_dashboard.png)

*Figure 6: Multi-panel performance evaluation dashboard.*

---

## 8. Technical Analysis & Discussion

The empirical data demonstrates a clear performance superiority of **Proxmox VE (Type-1)** over **VMware Workstation (Type-2)** in CPU-bound computational workloads.

### 1. Architectural Overhead & Trap-and-Emulate Delays
- **Proxmox VE (Type-1)** utilizes Linux KVM, which interfaces directly with hardware Intel VT-x / AMD-V virtualization extensions. CPU instructions generated inside the VM execute directly in VMX root mode with minimal hypervisor interception.
- **VMware Workstation (Type-2)** operates on top of Windows NT OS. Privileged guest CPU operations undergo double translation: first through VMware's VMM virtualization engine, and second through Windows kernel user-to-kernel mode context transitions (`NtSystemService`).

### 2. CPU Scheduling & Context Switching
- In Proxmox VE, guest vCPUs map directly to host Linux kernel POSIX threads scheduled by the **Completely Fair Scheduler (CFS)** operating at Ring 0.
- In VMware Workstation, guest CPU execution competes with Windows host background services (e.g., Windows Defender, System Updates, Desktop Window Manager). The host OS scheduler introduces thread preemptions, leading to higher latency spikes (Max Latency: 1.89 ms on VMware vs 1.34 ms on Proxmox; 95th Percentile: 0.90 ms vs 0.65 ms).

### 3. Memory & Virtual Cache Access
- Proxmox VE benefits from direct Extended Page Tables (EPT / NPT) hardware translation.
- Type-2 hypervisors incur memory address translation penalties when mapping Guest Physical Address (GPA) $\rightarrow$ Host Virtual Address (HVA) $\rightarrow$ Host Physical Address (HPA).

---

## 9. Conclusion & Engineering Takeaways

1. **Bare-metal dominance**: Proxmox VE (Type-1) delivers **+10.18% higher CPU throughput** (1,587.47 vs 1,440.80 EPS) and **8.70% lower average latency** compared to VMware Workstation (Type-2).
2. **Predictable Latency**: Proxmox VE exhibits a **27.78% lower 95th percentile latency** (0.65 ms vs 0.90 ms) and a **29.10% lower maximum latency** (1.34 ms vs 1.89 ms), demonstrating superior determinism and minimal jitter.
3. **Use-Case Recommendation**:
   - **Type-1 (Proxmox VE / KVM / ESXi)**: Recommended for Cloud Data Centers, Production Enterprise Infrastructure, Database Servers, and High-Performance Computing (HPC).
   - **Type-2 (VMware Workstation / VirtualBox)**: Recommended for Local Software Development, Testing, Desktop Sandbox Environments, and Educational Labs.

---

## 10. Repository Structure & Reproduction

### Folder Layout

```
Cloud_computing/
│
├── README.md                                  # Main Project & Benchmark Report
├── LAB_REPORT.md                              # Formal Academic Lab Report Submission
├── CClab1.docx                                # Original Part 1 Lab Document (Proxmox VE)
├── Type-2.docx                                # Original Part 2 Lab Document (VMware Workstation)
│
├── images/                                    # Screenshots & Generated Charts
│   ├── 1.png                                  # Proxmox VE Sysbench Result Screenshot
│   ├── 2.png                                  # VMware Workstation Sysbench Result Screenshot
│   ├── events_per_second_comparison.png       # Throughput Comparison Graph
│   ├── latency_comparison.png                 # Latency Metrics Graph
│   ├── total_events_comparison.png            # Total Events Graph
│   └── overall_performance_dashboard.png      # Multi-panel Dashboard
│
└── scripts/                                   # Automation & Plotting Scripts
    ├── benchmark.sh                           # Sysbench Automation Script
    ├── generate_plots.py                      # Matplotlib Visualization Generator
    └── parse_sysbench.py                      # Results Parser & Ratio Calculator
```

### How to Reproduce

1. **Run Benchmark Script on VM**:
   ```bash
   chmod +x scripts/benchmark.sh
   ./scripts/benchmark.sh
   ```

2. **Generate Plots**:
   ```bash
   python scripts/generate_plots.py
   ```

3. **Parse & Compare Results**:
   ```bash
   python scripts/parse_sysbench.py
   ```

---
*Laboratory Experiment conducted for Cloud Computing / Computer Networks Course.*
