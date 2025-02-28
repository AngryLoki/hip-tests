## Using hipEvents to measure performance ###

This tutorial is follow-up of the previous one where we learn how to write our first hip program, in which we compute Matrix Transpose. In this tutorial, we'll explain how to use the hipEvent to get the performance score for memory transfer and kernel execution time.

## Introduction:

Memory transfer and kernel execution are the most important parameter in parallel computing (specially HPC and machine learning). Memory bottlenecks is the main problem why we are not able to get the highest performance, therefore obtaining the memory transfer timing and kernel execution timing plays key role in application optimization.

## Requirement:
For hardware requirement and software installation [Installation](https://rocm.docs.amd.com/projects/HIP/en/latest/install/install.html)

## prerequiste knowledge:

Programmers familiar with CUDA, OpenCL will be able to quickly learn and start coding with the HIP API. In case you are not, don't worry. You choose to start with the best one. We'll be explaining everything assuming you are completely new to gpgpu programming.

## Simple Matrix Transpose

We will be using the Simple Matrix Transpose application from the previous tutorial and modify it to learn how to get the performance score for memory transfer and kernel execution time.

## hipEvent_t

We'll learn how to use the event management functionality of HIP runtime api. In the same sourcecode, we used for MatrixTranspose we will declare the following events as follows:

```
   hipEvent_t start, stop;
```

We'll create the event with the help of following code:

```
   hipEventCreate(&start);
   hipEventCreate(&stop);
```

We'll use the "eventMs" variable to store the time taken value:
`  float eventMs = 1.0f;`

## Time taken measurement by using hipEvents:

We'll start the timer by calling:
`  hipEventRecord(start, NULL);`
in this, the first parameter is the hipEvent_t, will will mark the start of the time from which the measurement has to be performed, while the second parameter has to be of the type hipStream_t. In current situation, we have passed NULL (the default stream). We will learn about the `hipStream_t` in more detail latter.

Now, we'll have the operation for which we need to compute the time taken. For the case of memory transfer, we'll place the `hipMemcpy`:
`  hipMemcpy(gpuMatrix, Matrix, NUM*sizeof(float), hipMemcpyHostToDevice);`

and for kernel execution time we'll use `hipKernelLaunch`:
```
hipLaunchKernelGGL(matrixTranspose,
                   dim3(WIDTH/THREADS_PER_BLOCK_X, HEIGHT/THREADS_PER_BLOCK_Y),
                   dim3(THREADS_PER_BLOCK_X, THREADS_PER_BLOCK_Y),
                   0, 0,
                   gpuTransposeMatrix , gpuMatrix, WIDTH ,HEIGHT);
```

Now to mark the end of the eventRecord, we will again use the hipEventRecord by passing the stop event:
`  hipEventRecord(stop, NULL);`

Will synchronize the event with the help of:
`  hipEventSynchronize(stop);`

In order to calculate the time taken by measuring the difference of occurance marked by the start and stop event, we'll use:
`  hipEventElapsedTime(&eventMs, start, stop);`
Here the first parameter will store the time taken value, second parameter is the starting marker for the event while the third one is marking the end.

We can print the value of time take comfortably since eventMs is float variable.

## How to build and run:
- Build the sample using cmake
```
$ mkdir build; cd build
$ cmake .. -DCMAKE_PREFIX_PATH=/opt/rocm
$ make
```
- Execute the sample
```
$ ./hipEvent
Device name Navi 14 [Radeon Pro W5500]
hipMemcpyHostToDevice time taken  =  0.981ms
kernel Execution time             =  0.539ms
hipMemcpyDeviceToHost time taken  =  1.220ms
PASSED!
```

## More Info:
- [HIP FAQ](https://rocm.docs.amd.com/projects/HIP/en/latest/how-to/faq.html)
- [HIP Kernel Language](https://rocm.docs.amd.com/projects/HIP/en/latest/reference/kernel_language.html)
- [HIP Runtime API (Doxygen)](https://rocm.docs.amd.com/projects/HIP/en/latest/doxygen/html/index.html)
- [HIP Porting Guide](https://rocm.docs.amd.com/projects/HIP/en/latest/how-to/hip_porting_guide.html)
- [HIP Terminology](https://rocm.docs.amd.com/projects/HIP/en/latest/reference/terms.html) (including comparing syntax for different compute terms across CUDA/HIP/OpenL)
- [HIPIFY](https://rocm.docs.amd.com/projects/HIPIFY/en/latest/index.html)
- [Developer/CONTRIBUTING Info](https://github.com/ROCm/HIP/blob/develop/CONTRIBUTING.md)
