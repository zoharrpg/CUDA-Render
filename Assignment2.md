# Assignment2
Junshang Jia--junshanj, Kaiyuan Liu--kaiyuan3

## Question 1:

Compare and explain the difference between the results provided by two sets of timers (the timer you added and the timer that was already in the provided starter code). Are the bandwidth values observed roughly consistent with the reported bandwidths available to the different components of the machine?
```bash
---------------------------------------------------------
Found 1 CUDA devices
Device 0: NVIDIA GeForce RTX 2080
   SMs:        46
   Global mem: 7982 MB
   CUDA Cap:   7.5
---------------------------------------------------------
Overall: 32.794 ms              [6.816 GB/s]
Kernel: 0.685 ms                [326.154 GB/s]
Overall: 34.109 ms              [6.553 GB/s]
Kernel: 0.682 ms                [327.952 GB/s]
Overall: 33.907 ms              [6.592 GB/s]
Kernel: 0.685 ms                [326.293 GB/s]

```
The RTX 2080 GPU has 448 GB/s memory bandwidthand Kernel speed is really fast, about 326GB/s, and consistent with this high memory bandwidth.
The Overall timer includes the data transfer, about 6.5GB/s. The PCIe-x16 bus has maximum transfer speed of 16 GB/s, which is much lower than the kernel speed, so this speed is consistent with PCIe-x16 maximum transfer speed. GPU has really fast speed to process data internal, but the data transfer speed is limited by the bandwidth of PCIe-x16 bus. Therefore, the overall time is much longer due to the difference between the GPU fast process data bandwidth and low transfer data bandwidth between CPU and GPU.


## Questions 2
Machine: ghc56.ghc.andrew.cmu.edu
```bash
------------
Score table:
------------
-------------------------------------------------------------------------
| Scene Name      | Target Time     | Your Time       | Score           |
-------------------------------------------------------------------------
| rgb             | 0.1855          | 0.1789          | 12              |
| rand10k         | 1.9560          | 2.0302          | 12              |
| rand100k        | 17.8552         | 19.3228         | 12              |
| pattern         | 0.2713          | 0.2522          | 12              |
| snowsingle      | 7.7305          | 6.9439          | 12              |
| biglittle       | 14.2676         | 16.9361         | 12              |
-------------------------------------------------------------------------
|                                   | Total score:    | 72/72           |
-------------------------------------------------------------------------
```
3. We decomposed the problem into box of piexel and process each pixel in the box. The maximum number of threads in a block for RTX 2080 is 1024. Therefore, we decide to use 32x32 threads to process box of piexels in parallel.
4. Basically, we use __syncthreads() to as the barrier for all threads. First, we wait all threads to compute their circle index, and process the box function. Second, we use __syncthreads() after sharedMemExclusiveScan to wait prefix sum completeion. At end the this inner loop, wait all threads to complete this iteration and move to next iteration together.
``` cpp
  cIndex = i + linearThreadIndex;
        __syncthreads();
        if (cIndex < numberOfCircles) {
            float3 position = *(float3*)(&cuConstRendererParams.position[3 * cIndex]);
            float radius = cuConstRendererParams.radius[cIndex];
            threadCount[linearThreadIndex] = circleInBoxConservative(position.x, position.y, radius, boxL, boxR, boxT, boxB) &&
                                              circleInBox(position.x, position.y, radius, boxL, boxR, boxT, boxB);
        } else {
            threadCount[linearThreadIndex] = 0;
        }

        sharedMemExclusiveScan(linearThreadIndex, threadCount, blockCount, prefixSumScratch, SCAN_BLOCK_DIM);
        __syncthreads();

        uint total = blockCount[SCAN_BLOCK_DIM - 1] + threadCount[SCAN_BLOCK_DIM - 1];
        if (linearThreadIndex < SCAN_BLOCK_DIM - 1 && blockCount[linearThreadIndex + 1] > blockCount[linearThreadIndex]) {
            prefixSumScratch[blockCount[linearThreadIndex]] = cIndex;
        }

        if (linearThreadIndex == SCAN_BLOCK_DIM - 1 && (total > blockCount[linearThreadIndex])) {
            prefixSumScratch[blockCount[linearThreadIndex]] = cIndex;
        }
        __syncthreads();

```
5. Reduce communication by moving variables to shared memory, like moving prefixsum array and other frequently used constants to shared memory to make the access fast. Make image variable to local variable, and update it only when render is complete for this thread.

6. In the first version, we use multiple threads to calculate each piexel with each circle, and we found the performance is very low. The problem is each piexl calculate each circle, whic produce a lot of work. To readuce this workload, we preprocess the circle, and find the circle that has intersect with this area. Then, we only render the circle that has intersect with area. In the end, we make image variable as the local variable to avoid the frequent read and write to global memory. 
   





