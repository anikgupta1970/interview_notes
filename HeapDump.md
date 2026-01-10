Scenario: High Memory Usage / Frequent GC – Heap Dump Analysis

An issue was reported in production where the application was experiencing high memory usage and frequent garbage collection, leading to slow response times and occasional service instability. In some cases, the application was close to an OutOfMemoryError.

Application logs did not clearly indicate the source of the problem, so I decided to perform a heap dump analysis to identify possible memory leaks or excessive object retention.

Action Taken

Identified the application PID

jps


Captured a heap dump

jmap -dump:live,format=b,file=heapDump.hprof <PID>


live → captures only reachable (live) objects

format=b → binary format

heapDump.hprof → heap dump file

Transferred the heap dump securely from the production server to a local machine for analysis.

Analysis

I analyzed the heap dump using tools such as:

Eclipse MAT (Memory Analyzer Tool)

(Alternatively) VisualVM

During analysis, I focused on:

Top memory-consuming objects

Retained heap size

Dominator tree

GC roots and object references

Findings

A large number of objects of a specific type were retained in memory

These objects were referenced from a static cache / singleton

The cache had no eviction policy and kept growing over time

This prevented garbage collection, causing heap pressure and frequent GC cycles

Root Cause

The root cause was a memory leak due to unbounded in-memory caching, where objects were strongly referenced and never released.

Fix Implemented

Introduced cache size limits and eviction (LRU-based)

Replaced strong references with weak references where applicable

Added monitoring and alerts for heap usage and GC activity

Reviewed object lifecycle to ensure proper cleanup

Outcome

Heap usage stabilized

GC frequency reduced significantly

Application performance improved

No further OutOfMemoryErrors observed in production
