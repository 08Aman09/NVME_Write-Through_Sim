# Software Design

## 1. Design Goals

The software is designed with the following goals:

1. Simulate a write-through storage system.
2. Compare a baseline write-through implementation with an optimized implementation.
3. Demonstrate Linux system programming concepts.
4. Demonstrate C++17 programming and object-oriented design.
5. Model NVMe queue parallelism in a simple user-space environment.
6. Measure latency, throughput, IOPS, queue utilization, and physical storage operations.

---

## 2. Design Approach

The project uses a modular C++ architecture.

The major modules are:

- Workload generation
- Write-through cache
- NVMe device model
- Performance metrics
- Command-line interface

Each module has a specific responsibility and communicates with the other modules through well-defined C++ classes and functions.

The design keeps the NVMe hardware behavior simplified because the project is a software simulator rather than a physical NVMe driver.

---

## 3. Module Structure

The project is organized as follows:

```text
nvme_wt_sim/
│
├── include/
│   ├── metrics.hpp
│   ├── nvme_device.hpp
│   ├── workload.hpp
│   └── write_through_cache.hpp
│
├── src/
│   ├── main.cpp
│   ├── metrics.cpp
│   ├── nvme_device.cpp
│   ├── workload.cpp
│   └── write_through_cache.cpp
│
├── scripts/
│   └── plot_results.py
│
├── results/
│   ├── CSV files
│   ├── benchmark logs
│   └── generated charts
│
├── Makefile
└── README.md

## 4. Module Responsibilities

### 4.1 Main Module

The `main.cpp` module is responsible for:

- Reading command-line arguments.
- Configuring the simulation.
- Creating the required components.
- Running the baseline benchmark.
- Running the optimized benchmark.
- Comparing the results.
- Printing the final performance report.
- Writing comparison results to CSV files.

---

### 4.2 Workload Module

The workload module generates logical write requests.

It supports:

- Uniform address distribution.
- Zipfian address distribution.
- Configurable number of requests.
- Configurable address space.
- Configurable random seed.
- Configurable Zipf skew.

The same workload configuration is used for both implementations to maintain a fair comparison.

---

### 4.3 NVMe Device Module

The NVMe device module provides the simulated storage device.

Its responsibilities include:

- Managing simulated NVMe queue pairs.
- Submitting write operations.
- Processing storage requests.
- Tracking queue depth.
- Tracking queue utilization.
- Recording physical storage operations.
- Performing the actual Linux file I/O operation.

The module provides a simplified representation of NVMe device behavior rather than implementing a real hardware controller.

---

### 4.4 Write-Through Cache Module

The write-through cache module contains the two main cache implementations.

#### Baseline Write-Through Cache

The baseline implementation performs synchronous writes.

The basic flow is:

```text
Application Request
        |
        v
Baseline Cache
        |
        v
Queue 0
        |
        v
NVMe Device Model
        |
        v
Linux pwrite()
        |
        v
Completion
        |
        v
Application Acknowledgement

## 5. Baseline Algorithm

The baseline implementation follows a simple synchronous approach.

```text
Start
  |
  v
Receive logical write
  |
  v
Select Queue 0
  |
  v
Submit write operation
  |
  v
Wait for completion
  |
  v
Record latency and metrics
  |
  v
Acknowledge request
  |
  v
Next request


## 6. Optimized Algorithm

The optimized implementation collects pending writes before sending them to the simulated device.

```text
Start
  |
  v
Receive logical write
  |
  v
Add request to pending set
  |
  v
Check for duplicate block
  |
  +---- Yes ----> Replace with latest write
  |
  +---- No -----> Add new pending write
  |
  v
Check flush condition
  |
  +---- Timer reached
  |
  +---- Batch threshold reached
  |
  v
Create batch
  |
  v
Distribute operations across queues
  |
  v
Submit to NVMe device model
  |
  v
Wait for completion
  |
  v
Complete waiting requests
  |
  v
Record metrics
  |
  v
End

## 7. Write Coalescing

Write coalescing reduces redundant physical storage operations.

For example, consider the following logical requests:

```text
Request 1 -> Block 100
Request 2 -> Block 200
Request 3 -> Block 100

## 8. Queue Management

The optimized implementation can use multiple simulated NVMe queue pairs.

A simple round-robin distribution can be used:

```text
Request 0 -> Queue 0
Request 1 -> Queue 1
Request 2 -> Queue 2
Request 3 -> Queue 3
Request 4 -> Queue 0
Request 5 -> Queue 1
...


## 9. Batching

The optimized cache groups pending writes into batches.

A batch can be triggered by:

- Flush timer expiration.
- Number of pending requests reaching the configured threshold.

Example:

```bash
./nvme_wt_sim --flush-interval-us 200 --batch-trigger 64


## 10. Concurrency and Synchronization

The project uses standard C++ concurrency features.

### Threads

`std::thread` is used for concurrent execution.

### Mutex

`std::mutex` protects shared data structures from simultaneous access.

### Condition Variable

`std::condition_variable` is used to coordinate background processing and pending requests.

### Atomic Variables

`std::atomic` is used for counters and statistics that can be safely updated by multiple threads.

### Futures and Promises

`std::future` and `std::promise` provide a mechanism for communicating completion between asynchronous operations and requesting threads.


## 11. Linux System Programming

The project uses Linux system-programming interfaces for the storage path.

Important concepts include:

- File descriptors
- `open()`
- `pwrite()`
- `close()`
- `O_DIRECT`
- POSIX-aligned memory
- Linux file-system interaction
- Multithreading

The simulator combines a software storage architecture with real Linux I/O operations.


## 12. Memory Management

The project uses aligned memory for operations performed using `O_DIRECT`.

The design also uses C++ RAII principles to manage resources.

Examples include:

- Automatic cleanup of allocated buffers
- Automatic closing of device resources
- Automatic destruction of C++ objects

This reduces the possibility of memory leaks and resource-management errors.


## 13. Performance Metrics

The simulator collects performance information from both implementations.

The main metrics are:

| Metric | Description |
|---|---|
| Average Latency | Average time required to complete a request |
| P50 Latency | Median request latency |
| P95 Latency | 95th percentile latency |
| P99 Latency | 99th percentile latency |
| Maximum Latency | Highest observed request latency |
| Application IOPS | Logical operations completed per second |
| Physical Device IOPS | Physical storage operations per second |
| Application Throughput | Logical data throughput |
| Physical Throughput | Physical data written per second |
| Queue Utilization | Utilization of simulated queues |
| Physical Storage Operations | Number of physical writes |
| Coalescing Reduction | Reduction in physical operations |

---

## 14. Error Handling

The application checks important operations for failures.

Examples include:

- File opening errors
- Storage write errors
- Invalid command-line arguments
- Memory allocation failures
- Invalid simulation configuration

Errors are reported to the user instead of silently continuing with invalid data.

---

## 15. Configuration

The simulator provides command-line options for controlling the experiment.

Examples include:

```text
--requests
--address-space
--queues
--distribution
--zipf-skew
--flush-interval-us
--batch-trigger
--app-threads
--seed
--sim-latency-us
--backing-dir
--results-dir

## 16. Build Design

The project uses GNU Make for compilation.

The source files are compiled using:

```text
C++17
-O2
-Wall
-Wextra
-pthread


## 17. Testing Approach

Testing is performed at multiple levels.

### 17.1 Build Testing

The project is compiled using:

```bash
make clean
make

### 17.2 Functional Testing

The executable is tested using different configurations.

Example:

```bash
./nvme_wt_sim --requests 100 --address-space 1000 --queues 2 --app-threads 2 --sim-latency-us 60

### 17.3 Workload Testing

Different workload distributions are tested to evaluate the behavior of the simulator under different access patterns.

The project supports:

- Uniform distribution
- Zipfian distribution

Uniform distribution provides a relatively even access pattern across the address space.

Zipfian distribution creates a skewed access pattern where some blocks are accessed more frequently than others.

Example:

```bash
./nvme_wt_sim --distribution uniform

### 17.4 Timing Testing

The simulator supports both modeled device latency and real `O_DIRECT` timing.

Modeled timing:

```bash
./nvme_wt_sim --sim-latency-us 60

### 17.5 Result Validation

The generated results are checked to verify that the simulator produces valid output.

The following files are inspected:

- Comparison CSV files
- Execution log files
- Performance charts

The results are checked for:

- Logical write request count
- Physical storage operation count
- Latency measurements
- Throughput
- IOPS
- Queue utilization
- Coalescing reduction

The results from different workloads and configurations are compared to evaluate the behavior of the simulator.


## 18. Design Trade-offs

The optimized implementation does not necessarily improve every metric for every workload.

Increasing concurrency can increase queueing and therefore request latency while improving overall throughput.

Write coalescing is more effective when multiple requests target the same blocks.

Therefore, the simulator evaluates both:

- Uniform workloads
- Zipfian workloads

to demonstrate how workload characteristics affect optimization results.

---

## 19. Design Summary

The project uses a simple modular design consisting of:

```text
Workload
   |
   v
Write-Through Cache
   |
   +-------------------+
   |                   |
   v                   v
Baseline           Optimized
                       |
                       v
                Coalescing
                       |
                       v
                   Batching
                       |
                       v
                Multiple Queues
                       |
                       v
                NVMe Device Model
                       |
                       v
                  Linux I/O
                       |
                       v
                 Backing File
