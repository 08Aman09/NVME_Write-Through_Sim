# Project Requirements

## 1. Project Overview

The NVMe Write-Through Cache Accelerator Simulator is a Linux-based C++17
software simulator designed to study write-through cache performance in an
NVMe-inspired storage architecture.

The project compares a conventional synchronous write-through implementation
with an optimized implementation using batching, duplicate-write coalescing,
multiple queue pairs, and concurrent request processing.

The simulator also performs real Linux file I/O using `O_DIRECT` and
`pwrite()` against a backing file while optionally using a modeled device
service time for architecture-level performance evaluation.

## 2. Functional Requirements

### FR-01: Workload Generation

The system shall generate configurable logical write workloads.

Supported workload distributions:

- Uniform
- Zipfian

The workload shall support configurable:

- Number of requests
- Address-space size
- Random seed
- Zipf skew

### FR-02: Baseline Write-Through Cache

The system shall provide a baseline synchronous write-through cache.

For every logical write:

1. The request shall be submitted to the storage device model.
2. The storage operation shall complete.
3. The request shall be acknowledged.

The baseline implementation shall use a single queue with queue depth one.

### FR-03: Optimized Write-Through Cache

The system shall provide an optimized write-through cache using:

- Multiple queue pairs
- Concurrent request processing
- Background processing
- Request batching
- Duplicate-LBA write coalescing

### FR-04: Write-Through Semantics

The system shall not acknowledge a write before its corresponding data has
been processed by the storage device model.

### FR-05: NVMe Queue Simulation

The system shall model multiple NVMe queue pairs.

Each queue shall track relevant request and utilization information.

### FR-06: Linux Storage I/O

The system shall perform real Linux file I/O using appropriate Linux/POSIX
interfaces.

The current implementation uses:

- `open()`
- `pwrite()`
- `O_DIRECT`
- `close()`
- aligned memory allocation

### FR-07: Performance Metrics

The system shall measure and report:

- Average latency
- P50 latency
- P95 latency
- P99 latency
- Maximum latency
- Application throughput
- Physical throughput
- Application IOPS
- Physical IOPS
- Queue utilization
- Physical storage operations
- Coalescing reduction

### FR-08: Configurable Execution

The application shall provide command-line configuration for:

- Request count
- Address space
- Queue count
- Workload distribution
- Zipf skew
- Application threads
- Flush interval
- Batch trigger
- Device service latency
- Backing directory
- Results directory

### FR-09: Result Generation

The system shall generate machine-readable comparison results in CSV format.

### FR-10: Reproducibility

The same random seed and configuration shall produce reproducible workload
generation behavior.

## 3. Non-Functional Requirements

### NFR-01: Platform

The project shall run on Linux.

The primary development environment is Ubuntu 24.04 running under WSL2.

### NFR-02: Programming Language

The implementation shall use C++17.

Python shall not be used for the core simulator implementation.

### NFR-03: Build System

The project shall be buildable using GNU Make.

### NFR-04: Compiler

The project shall support a modern GNU C++ compiler.

### NFR-05: Code Quality

The project shall be compiled with warnings enabled:

- `-Wall`
- `-Wextra`

### NFR-06: Concurrency

Concurrent components shall use appropriate C++ synchronization primitives.

### NFR-07: Maintainability

The implementation shall separate interfaces and implementations using header
and source files.

### NFR-08: Reproducibility

Benchmark configurations and important results shall be documented.

### NFR-09: Simplicity

The project shall remain a software simulator rather than attempting to
implement a complete physical NVMe controller or production Linux NVMe
kernel driver.

## 4. System Constraints

The project models NVMe device behavior in software.

The simulator does not attempt to implement:

- PCIe hardware communication
- NVMe controller firmware
- A complete production NVMe kernel driver
- Physical SSD controller hardware

Linux system-programming concepts are demonstrated through the real Linux I/O
path and concurrency mechanisms used by the simulator.

## 5. Deliverables

The final repository shall contain:

- C++ source code
- Header files
- Makefile
- README
- Requirements documentation
- Architecture documentation
- Design documentation
- Test cases
- Performance results
- Architecture diagrams
- Git history
- Execution instructions
