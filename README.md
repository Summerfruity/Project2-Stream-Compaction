CUDA Stream Compaction
======================

**University of Pennsylvania, CIS 565: GPU Programming and Architecture, Project 2**

* Name: Summerfruity
* Tested on: Windows 11, AMD Ryzen 7 6800H, NVIDIA GeForce RTX 3060 Laptop GPU

### Project Overview

This project implements several versions of prefix-sum scan and stream compaction in CUDA and C++.

Implemented features:

* CPU exclusive scan
* CPU stream compaction without scan
* CPU stream compaction with scan
* GPU naive scan
* GPU work-efficient scan
* GPU stream compaction using work-efficient scan
* Thrust scan baseline

### Implementation Notes

The project compares multiple scan implementations:

* A serial CPU scan used as a correctness reference
* A naive GPU scan
* A work-efficient GPU scan based on the Blelloch up-sweep / down-sweep approach
* A Thrust-based scan as a library baseline

For stream compaction, the project also includes:

* A direct CPU compaction implementation without scan
* A CPU compaction implementation built on scan
* A GPU compaction implementation built on the work-efficient scan

### Performance Analysis

To reduce timing noise, large-scale testing was performed with `SIZE = 1 << 20`. For scan benchmarking, the first run was treated as a warm-up run and excluded from the final result. The reported benchmark values are the average of 20 runs.

Observed average scan timings:

#### Power-of-two input (`n = 1048576`)

* CPU scan: `0.446695 ms`
* Naive GPU scan: `0.914424 ms`
* Work-efficient GPU scan: `0.687051 ms`
* Thrust scan: `0.578616 ms`

#### Non-power-of-two input (`n = 1048573`)

* CPU scan: `0.434840 ms`
* Naive GPU scan: `0.864520 ms`
* Work-efficient GPU scan: `0.496755 ms`
* Thrust scan: `0.516326 ms`

### Discussion

The work-efficient scan is clearly faster than the naive scan after removing first-run effects and averaging repeated runs.

For the power-of-two case, the work-efficient implementation reduces scan time from `0.914424 ms` to `0.687051 ms`, which is about a `1.33x` speedup over the naive version.

For the non-power-of-two case, the work-efficient implementation reduces scan time from `0.864520 ms` to `0.496755 ms`, which is about a `1.74x` speedup over the naive version.

This shows that the work-efficient algorithm successfully improves over the naive GPU scan in practice.

However, on this machine and at this problem size, the serial CPU scan is still faster than all tested GPU scan implementations. This is reasonable because scan requires multiple passes and synchronization, so kernel launch overhead and scan structure overhead remain significant at this scale.

Among the GPU scan implementations, Thrust performs very well and is close to or slightly better than the custom work-efficient scan depending on the input size. This suggests that the custom implementation is competitive, but there is still room for further optimization such as reducing kernel overhead, improving occupancy, or using shared memory optimizations.

For stream compaction, the GPU work-efficient compaction performs better than the CPU compaction with scan in the tested run, which indicates that the scan-based GPU pipeline is effective for compaction workloads.

### Test Output

```text
****************
** SCAN TESTS **
****************
    [  11  27  10  16   2   6  36  10  17   2   9  23   4 ...  23   0 ]
==== cpu scan, power-of-two ====
   elapsed time: 0.5194ms    (std::chrono Measured)
    [   0  11  38  48  64  66  72 108 118 135 137 146 169 ... 25714479 25714502 ]
==== cpu scan, non-power-of-two ====
   elapsed time: 0.5938ms    (std::chrono Measured)
    [   0  11  38  48  64  66  72 108 118 135 137 146 169 ... 25714425 25714452 ]
    passed
==== naive scan, power-of-two ====
   elapsed time: 0.938624ms    (CUDA Measured)
    passed
==== naive scan, non-power-of-two ====
   elapsed time: 0.838848ms    (CUDA Measured)
    passed
==== work-efficient scan, power-of-two ====
   elapsed time: 0.813792ms    (CUDA Measured)
    passed
==== work-efficient scan, non-power-of-two ====
   elapsed time: 0.723968ms    (CUDA Measured)
    passed
==== thrust scan, power-of-two ====
   elapsed time: 0.326656ms    (CUDA Measured)
    passed
==== thrust scan, non-power-of-two ====
   elapsed time: 0.701312ms    (CUDA Measured)
    passed

*****************************
** STREAM COMPACTION TESTS **
*****************************
    [   3   3   0   0   0   2   0   0   1   0   1   3   0 ...   3   0 ]
==== cpu compact without scan, power-of-two ====
   elapsed time: 1.7001ms    (std::chrono Measured)
    [   3   3   2   1   1   3   3   3   2   1   1   1   1 ...   1   3 ]
    passed
==== cpu compact without scan, non-power-of-two ====
   elapsed time: 1.6847ms    (std::chrono Measured)
    [   3   3   2   1   1   3   3   3   2   1   1   1   1 ...   3   2 ]
    passed
==== cpu compact with scan ====
   elapsed time: 4.5824ms    (std::chrono Measured)
    [   3   3   2   1   1   3   3   3   2   1   1   1   1 ...   1   3 ]
    passed
==== work-efficient compact, power-of-two ====
   elapsed time: 0.717824ms    (CUDA Measured)
    passed
==== work-efficient compact, non-power-of-two ====
   elapsed time: 0.728064ms    (CUDA Measured)
    passed

**********************************************
** SCAN BENCHMARK (drop warmup, average)   **
**********************************************
    cpu scan (power-of-two)              n=1048576 avg(20 runs) = 0.446695ms
    naive scan (power-of-two)            n=1048576 avg(20 runs) = 0.914424ms
    work-efficient scan (power-of-two)   n=1048576 avg(20 runs) = 0.687051ms
    thrust scan (power-of-two)           n=1048576 avg(20 runs) = 0.578616ms
    cpu scan (non-power-of-two)          n=1048573 avg(20 runs) = 0.434840ms
    naive scan (non-power-of-two)        n=1048573 avg(20 runs) = 0.864520ms
    work-efficient scan (non-power-of-two) n=1048573 avg(20 runs) = 0.496755ms
    thrust scan (non-power-of-two)       n=1048573 avg(20 runs) = 0.516326ms
```

