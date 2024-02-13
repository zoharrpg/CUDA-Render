# Assignment2

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
