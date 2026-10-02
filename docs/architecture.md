# System Architecture

## 1. Overview

The NVMe Write-Through Cache Accelerator Simulator models the behavior of a
write-through storage system using C++17 on Linux.

The system compares two write-through approaches:

1. Baseline synchronous write-through
2. Optimized batched, coalesced, multi-queue write-through

Both implementations use the same workload and NVMe device model so that
their performance can be compared fairly.

## 2. High-Level Architecture

```text
                    Application
                         |
                         v
                +------------------+
                | Workload Manager |
                +--------+---------+
                         |
                         v
                +------------------+
                | Write-Through    |
                | Cache Layer       |
                +--------+---------+
                         |
              +----------+----------+
              |                     |
              v                     v
       +-------------+       +---------------+
       |  Baseline   |       |  Optimized    |
       | Write Path  |       |  Write Path   |
       +------+------+       +-------+-------+
              |                      |
              |                      |
              v                      v
       +--------------------------------------+
       |          NVMe Queue Model            |
       |                                      |
       | Queue 0 | Queue 1 | ... | Queue N    |
       +------------------+-------------------+
                          |
                          v
                 +----------------+
                 | NVMe Device    |
                 | Model          |
                 +-------+--------+
                         |
                         v
                 +----------------+
                 | Linux I/O      |
                 | O_DIRECT       |
                 | pwrite()       |
                 +-------+--------+
                         |
                         v
                 +----------------+
                 | Backing File   |
                 +----------------+


## 3. Major Components

### 3.1 Workload Manager

The workload manager generates logical write requests.

It supports:

- Uniform address distribution
- Zipfian address distribution
- Configurable request count
- Configurable address space
- Configurable random seed
- Configurable Zipf skew

The generated workload is shared between the baseline and optimized implementations to maintain a fair comparison.

---

### 3.2 Baseline Write-Through Cache

The baseline implementation represents a simple synchronous write-through path.

Characteristics:

- Single queue
- Queue depth of one
- One storage operation per logical write
- Caller waits for completion before the request is acknowledged

This provides the reference implementation for performance comparison.

---

### 3.3 Optimized Write-Through Cache

The optimized implementation introduces several techniques:

- Multiple queue pairs
- Concurrent application requests
- Background flushing
- Batching
- Duplicate write coalescing

Multiple writes targeting the same logical block within a flush window can be combined so that only the latest value needs to be physically written.

---

### 3.4 NVMe Queue Model

The NVMe device model contains multiple queue pairs.

Each queue represents an independent simulated submission/completion path.

The model tracks:

- Queue depth
- Outstanding operations
- Maximum queue depth
- Queue utilization
- Completed operations

Worker threads represent independent device-side execution paths.

---

### 3.5 Linux I/O Layer

The simulator performs actual Linux file operations.

The storage backing file is accessed using:

- File descriptors
- `O_DIRECT`
- `pwrite()`
- Aligned memory
- `close()`

This allows the simulator to exercise real Linux system-programming interfaces while the NVMe service time can independently be modeled.

---

### 3.6 Metrics System

The metrics component records performance information from both cache implementations.

Measured metrics include:

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

---

## 4. Data Flow

A logical write follows this general path:

```text
Logical Write
     |
     v
Workload Generator
     |
     v
Write-Through Cache
     |
     +----------------------+
     |                      |
     v                      v
Baseline                Optimized
     |                      |
     |                 Pending Writes
     |                      |
     |                 Coalescing
     |                      |
     |                  Batch Flush
     |                      |
     +----------+-----------+
                |
                v
          NVMe Queue Pair
                |
                v
           Device Model
                |
                v
          Linux pwrite()
                |
                v
           Backing File
                |
                v
           Completion
                |
                v
         Request Ack


## 5. Concurrency Model

The baseline path intentionally provides limited concurrency.

The optimized path allows multiple application threads to submit requests while a background processing mechanism collects and dispatches pending operations.

The NVMe model uses worker threads associated with queue pairs.

Synchronization is implemented using C++ facilities such as:

- `std::thread`
- `std::mutex`
- `std::condition_variable`
- `std::atomic`
- `std::future`
- `std::promise`

---

## 6. Timing Model

The simulator supports two device timing modes.

### 6.1 Modeled Timing

A configurable device service time is used to represent an NVMe-like storage device.

This allows queueing and architectural behavior to be studied independently of the storage performance of the development machine.

### 6.2 Real Linux I/O Timing

When modeled latency is disabled, the simulator uses the actual timing of Linux `O_DIRECT` storage operations.

This provides a real system-level I/O execution path while retaining the software NVMe architecture model.

---

## 7. System Architecture Principles

The simulator separates the logical storage architecture from the physical storage implementation.

This makes it possible to study:

- Queue parallelism
- Batching
- Write coalescing
- Concurrency
- Latency
- Throughput
- IOPS

without requiring physical NVMe hardware.

---

## 8. Linux and System Programming Concepts

The project demonstrates Linux system-programming concepts through its storage and concurrency implementation.

### 8.1 File I/O

The simulator uses Linux/POSIX interfaces such as:

- `open()`
- `pwrite()`
- `close()`

### 8.2 Direct I/O

`O_DIRECT` is used to perform direct file I/O while avoiding the normal page-cache path.

### 8.3 Memory Alignment

Aligned memory is used for buffers required by direct I/O.

### 8.4 Multithreading

C++ threads are used to model concurrent application and device-side execution.

### 8.5 Synchronization

Mutexes, condition variables, futures, promises, and atomic variables are used to coordinate concurrent operations.

---

## 9. Software Architecture

The project follows a modular architecture:

```text
include/
├── metrics.hpp
├── nvme_device.hpp
├── workload.hpp
└── write_through_cache.hpp

src/
├── main.cpp
├── metrics.cpp
├── nvme_device.cpp
├── workload.cpp
└── write_through_cache.cpp

scripts/
└── plot_results.py

results/
└── Benchmark results and generated charts


## 10. Performance Evaluation Architecture

The simulator evaluates both implementations using the same workload configuration.

The main performance metrics are:

| Metric | Purpose |
|---|---|
| Average Latency | Measures overall request latency |
| P50 Latency | Represents typical request latency |
| P95 Latency | Measures high-percentile latency |
| P99 Latency | Measures tail latency |
| Maximum Latency | Shows worst observed request latency |
| Application IOPS | Measures logical operations processed per second |
| Physical Device IOPS | Measures actual storage operations per second |
| Application Throughput | Measures logical data throughput |
| Physical Throughput | Measures physical storage throughput |
| Queue Utilization | Measures use of simulated NVMe queues |
| Physical Storage Operations | Measures actual physical write operations |
| Coalescing Reduction | Measures redundant writes eliminated |

---

## 11. Timing and Benchmark Modes

The simulator provides two execution modes for evaluating storage behavior.

### 11.1 Mode A — Modeled NVMe Timing

The simulator uses a configurable service-time model to represent the latency of an NVMe-like storage device.

Example:

```bash
./nvme_wt_sim --sim-latency-us 60

### 11.2 Mode B — Real Linux O_DIRECT Timing

The simulator can disable the modeled latency by setting the simulation latency to zero.

Example:

```bash
./nvme_wt_sim --sim-latency-us 0


## 12. Limitations

The project is an educational software simulator.

It does **not** implement:

- A physical NVMe controller
- PCIe hardware communication
- NVMe controller firmware
- Physical SSD controller logic
- A production Linux NVMe kernel driver

The NVMe queue and device behavior are modeled in user-space C++, while the backing storage path uses real Linux system calls.

Therefore, the measured results should be interpreted as simulator results rather than direct measurements of a physical NVMe SSD.

The purpose of the project is to demonstrate:

- Storage architecture
- Write-through caching
- Queue management
- Concurrency
- Linux system programming
- Performance measurement
- Performance analysis

in a controlled software environment.

---

## 13. Expected Outcome

The completed simulator provides a practical environment for comparing different write-through storage strategies.

The project demonstrates how:

- Multiple queues can provide greater concurrency.
- Batching can reduce request-processing overhead.
- Write coalescing can reduce redundant physical operations.
- Workload characteristics can affect optimization benefits.
- Queue utilization can influence system performance.
- Latency and throughput can have different trade-offs.

The simulator produces performance information through:

- Console reports
- CSV comparison files
- Benchmark logs
- Performance charts

These results can be used to compare the baseline and optimized write-through implementations.
