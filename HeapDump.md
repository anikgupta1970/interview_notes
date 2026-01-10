Scenario: High Memory Usage / Frequent GC – Heap Dump Analysis

An issue was reported in production where the application was experiencing high memory usage and frequent garbage collection, leading to slow response times and occasional service instability. In some cases, the application was close to an OutOfMemoryError.

Since logs did not clearly indicate the problem, I decided to perform a heap dump analysis.

Action Taken

Identified the application PID

jps


Captured a heap dump

jmap -dump:live,format=b,file=heapDump.hprof <PID>


Safely transferred the heap dump file from the production server for offline analysis.

Analysis

I analyzed the heap dump using Eclipse MAT (Memory Analyzer Tool).

I followed a structured approach:

1. Histogram Analysis

Started with the Histogram view to identify:

Classes with the highest number of instances

Classes consuming the most heap memory

This quickly highlighted unusually large object counts for a specific domain object.

2. Dominator Tree Analysis

Used the Dominator Tree to:

Identify objects with the largest retained heap

Understand which objects were preventing others from being garbage collected

This helped trace memory retention back to a single root object.

3. GC Roots & Reference Chain

Analyzed the GC root paths to see why objects were still reachable

Found that objects were strongly referenced from a static cache / singleton

Findings

Large numbers of domain objects were retained in memory

Histogram showed continuous growth in object count

Dominator Tree revealed that a static in-memory cache was dominating a large portion of the heap

The cache did not have an eviction or size limit

Root Cause

The root cause was a memory leak caused by an unbounded static cache, which retained strong references and prevented garbage collection.

Fix Implemented

Introduced cache size limits and eviction (LRU-based)

Replaced strong references with WeakReference where appropriate

Reviewed object lifecycle and ensured proper cleanup

Added heap and GC monitoring to catch similar issues early

Outcome

Heap usage stabilized

GC frequency reduced significantly

Application performance improved

No further memory-related incidents in production
