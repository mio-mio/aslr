# How Random Is ASLR in Practice? An Experimental Analysis

## 1. Introduction: When Addresses Stop Being Reliable

When I solved the split challenge, I was able to calculate addresses using fixed offsets. However, Address Space Layout Randomization (ASLR) and Position Independent Executables (PIE) randomize memory layouts, making such calculations unreliable.

Knowing this, I wanted to observe how much randomness these mechanisms actually introduce.


## 2. What ASLR Actually Randomizes

First, I focused on ASLR alone.


ASLR is often illustrated as fully randomizing the memory layout.  

At first glance, it looks as if memory addresses are completely shuffled across executions.


In practice, ASLR randomizes the locations of major program components such as the stack, heap, and shared libraries (e.g., libc).  

However, as we will see, this randomization is not entirely unrestricted.


## 3. Experiment Setup

I prepared a simple C program (aslr_demo) based on a Qiita article (https://qiita.com/taharma/items/4b65d1776b0d3b55164f), compiled it without PIE.

I then used a Python script to run the program multiple times, collect address data, and visualize the results using Matplotlib.


## 4. Observations: What Changed and What Didn’t

![output](Changedynon_Screenshot2026-04-27215714.png)

I observed that the addresses for the main program and global variables remained constant, while the addresses for the stack, heap, and libc changed on every execution.

The five address ranges are shown together in the graph below.

The main program is not clearly visible in the graph because it is very close to the global variables and overlaps with them. However, like the global variables, it remains at a low address range and appears as a single line.

The heap shows a distinctive spread, covering a wide range of addresses across executions.

In contrast, the stack and libc appear as relatively tight, horizontal bands at higher address ranges.

![memory layout overview (log scale)](Figure_A.png)

### 4.1 Fixed Addresses (The Main Program & Global Variables)

![Fixed Addresses](NotChanged_Screenshot2026-04-27214214.png)

The main program and global variables remained at fixed addresses across all executions because the program was compiled without PIE.

If PIE were enabled, these addresses would also be randomized, similar to other memory regions.

### 4.2 Heap Behavior

Even in the combined graph, the heap already appears highly variable.  

However, when we examine the addresses as offsets, the pattern becomes even clearer, as shown below.

![Heap Graph](Figure_3.png)

### 4.3 Stack Behavior

In the memory layout overview (log scale), the stack overlaps with libc and is not easily distinguishable. However, its address is not fixed.

The stack addresses consistently fall within the 0x7ff... range, suggesting a narrower range compared to the heap. Despite this, the stack is still randomized across executions, as shown below.

![Stack Graph](Figure_2.png)

### 4.4 Libc Behavior

Similar to the stack, libc addresses are not fixed.

Libc addresses consistently fall within the 0x7... range, but show a wider distribution compared to the stack.

![Libc Graph](Figure_4.png)

### 4.5 Quantifying Randomness

As observed, all addresses are randomized except for the main program and global variables.

To better understand this behavior, we can quantify the randomness of these address distributions.

One simple way to do this is by measuring entropy, which captures how spread out the values are.

![Entropy results](Entropy_Screenshot2026-04-28130638.png)

From the memory layout overview, the heap initially appeared to be the most randomized.

However, in terms of entropy, the values are nearly identical across the heap, stack, and libc.

This suggests that while the heap is more widely distributed, the stack and libc are still sufficiently varied within their narrower ranges.

In other words, the difference lies in the range of distribution, not in the level of randomness itself.

This leads to an important insight about how ASLR behaves in practice.


## 5. Key Insight: Random, but Constrained

The entropy and graph results show that ASLR randomizes heap, stack, and libc addresses, but the ranges of these addresses are constrained.

This constraint exists due to factors such as system safety, memory management, and performance considerations in the operating system.

Additionally, when examining the lower digits of the addresses, we can observe consistent patterns.

For example, stack addresses often end with "4", heap addresses with "310", and libc addresses with "1c0" across executions.

![Change Addresses](Changed_Screenshot2026-04-27214231.png)

This behavior is caused by memory alignment, which restricts addresses to specific boundaries.

Another important factor is that not all bits of an address are randomized.

In practice, only a portion of the address space is subject to randomization, while other parts remain fixed due to alignment and system constraints.

This limits the effective randomness, even though the addresses appear to change across executions.

As a result, the number of possible address values is significantly smaller than the full address space, which reduces the uncertainty for an attacker.

## 6. Conclusion: Random, but Not Fully Unpredictable

ASLR is often illustrated as fully randomizing memory layout, but in practice the randomization is constrained.

In this experiment, I observed that while addresses change across executions, they tend to fall within specific ranges depending on the memory region.

These constraints reduce the overall uncertainty, which can make certain attacks more feasible.

In the next step, I plan to explore how PIE affects this behavior by introducing additional randomization, such as randomizing the base address of the program.


