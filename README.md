# How Much Randomness Does ASLR Really Provide?

## 1. Introduction: When Addresses Stop Being Reliable

When I solved the split challenge, I was able to calculate addresses using fixed offsets. However, Address Space Layout Randomization (ASLR) and Position Independent Executables (PIE) randomize memory layouts, making such calculations unreliable.

Knowing this, I wanted to observe how much randomness these mechanisms actually introduce.

## 2. What ASLR Actually Randomizes

First, I focused on ASLR alone. When ASLR is available, it randomly assigns an address for the major components for program such as stacks, shared libraries (libc), main program and heap. 

## 3. Experiment Setup

I prepared a simple C program (aslr_demo) based on a Qiita article (https://qiita.com/taharma/items/4b65d1776b0d3b55164f), compiled it without PIE, and collected address data over multiple executions.

## 4. Observations: What Changed and What Didn’t
### 4.1 Fixed Addresses
