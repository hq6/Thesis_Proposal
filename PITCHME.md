# Arachne
## Towards Core-Aware Scheduling
#### Henry Qin Thesis Proposal

#HSLIDE

### Outline
 - Problem Description & Background
 - Hypothesis
 - Initial Results
 - Additional Work
 - Related Work
 - Request for Feedback

#HSLIDE

## The Problem

It is difficult to simultaneously achieve low latency, high throughput and high
core utilization for datacenter applications.

How have various applications attempted to deal with this problem?

#HSLIDE

##  RAMCloud
#### Low latency by polling
 - In the RAMCloud system, we achieve low latency by using kernel bypass and
   then polling for responses to RPC's in userspace.  Technically, we have high
   core utilization, although one might argue that the cores are not being used
   for useful work.
 - Unfortunately, the poor use of cores limits thoughput.

#HSLIDE

## Redis
#### Low latency and high throughput with an event loop
 - Redis assumes that the network is the primary bottleneck and uses epoll in
   an event loop to perform all operations.
 - Unfortunately, this limits Redis to a single core without
   application-managed sharding.

#HSLIDE

## Nginx
#### Low latency and high throughput with multiple non-blocking event loops on multiple processes.
 - Nginx is statically configurable, but typically uses a fixed number of
   processes equal to the number of cores, with each process doing non-blocking
   polls for new connections and new requests on existing connections.
 - Unfortunately, having a fixed number of kernel threads effectively polling
   implies that cores are wasted when load is not sufficiently high.

#HSLIDE

## Thread scheduling is not a solved problem.

The Linux Scheduler: a Decade of Wasted Cores
 - Cores stay idle for seconds while ready threads wait to run.

#HSLIDE

## Hypothesis
### We can achieve high throughput, low latency, and efficient core utilization by allocating cores in the kernel and scheduling in userspace.

#HSLIDE

## Hypothesis

 - Scheduling preemptively in the kernel is inefficient because the kernel is
   not aware of application state.
 - Scheduling cooperatively in userspace is inefficient because userspace
   schedulers sit on top of preemptable kernel threads.
    - This is true independently of whether userspace scheduling presents an
      event-based model or thread-based model.

#HSLIDE

## Current Status
 - Highly efficient scheduler implemented in userspace
    - github.com/PlatformLab/Arachne
 - Kernel module for core isolation in progress

#HSLIDE

## Initial Results
### RAMCloud machines (Nahalem Lynnfield, Xeon X3470 @ 2.93 Ghz, 4-Core with Hyperthreading Disabled)
 - 60 ns yield
 - 160 ns cross-core thread creation
 - 180 ns cross-core condition variable wakeup

### CloudLab c220g1 (2 Haswell, Intel E5-2630 v3 @ 2.40 GHz, 8-core CPUs)
 - 120 ns yield
 - 470 ns cross-core thread creation
 - 355 ns cross-core condition variable wakeup

#HSLIDE

## Additional Research Before Thesis
 - Can we efficiently handle blocking system calls using our dual user/kernel
   approach?
 - Can we devise policies for effectively relating parallelism with detected
   application load?
 - Can we efficiently mediate between multiple applications with changing
   workloads?

#HSLIDE

## Related Work
 - Scheduler Activations
 - Capriccio
 - Linux Cgroups
 - Golang

#HSLIDE

## 25 Years Ago: Scheduler Activations
 - A hybrid threading model in which the kernel and userspace threading library
   share information about application requirements and kernel events.
 - The kernel migrates "activations" and preempts cores, and then tells the
   application afterwards using an upcall.

#HSLIDE

## 14 Years Ago: Capriccio

 - A user-level N:1 threading package from Berkeley which feature the same
   performance as events-based models, by using static analysis to make
   resource utilization more efficient.
 - Handle blocking IO's by changing all IO's into nonblocking IO's.

#HSLIDE

## 9 Years Ago: Linux Cgroups

 - Linux kernel feature for resource (compute, memory, IO) isolation of one or
   more processes.
 - Could we leverage this for core allocation?

#HSLIDE

## Linux Cgroups for core allocation

 - We can and will use this for initial testing, but is not a permanent solution.
   - Does not solve the blocking IO problem
   - Does not allow us to move in-kernel threads off of our cores.

#HSLIDE

## 4 Years Ago: Golang Goroutines

 - Partially preemptive threading package embedded in language runtime
 - Extremely lightweight threads with initially small, growable stacks
 - Transparently spins up more kernel threads when current user threads block
   on system calls

## Request for Feedback
