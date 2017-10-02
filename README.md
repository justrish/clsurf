
<BODY>

<DIV id="id_1">
<H1> Object Detection using SURF Features on Intel Xeon Phi </H1>
</DIV>
<DIV id="id_2_1">
<P class="p9 ft6"><H2> Description </H2></P>
<P class="p10 ft6">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Initial implementation of SURF using OpenCL
</P>
</DIV>
<DIV id="id_2_2">
<P class="p9 ft6"><H2> Abstract </H2></P>
<P class="p12 ft6">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Object detection and feature matching is a crucial and versatile application in the image processing and recognition domain. The Speed up Robust features (SURF) kernel is fast and popular feature detector and extractor used for object detection. In the High Performance Computing domain, Intel Xeon Phi has been released in 2012 as a new competitor to GPUs. It has given performance close to GPUs or even better in many benchmarks and compute intensive applications. It has additional advantage of offering multiple programming models for the developers to program like OpenCL, OpenMP (Offload &amp; Native), MPI and Cilk threads, whichever suits the developer the most. The aim of this project is to evaluate the programming models on Intel Xeon Phi for SURF algorithm and implement a feature matching application on Intel Xeon Phi. The programming models we analyze are OpenCL (Offload), OpenMP (Offload) and OpenMP (native). We would be comparing the constraints, flexibility and performance of these programming models for SURF algorithm.
</P>
<P class="p9 ft6"><H2> Results </H2></P>
<P class="p12 ft6"><H2> Serial SURF profiling on Intel Xeon </H2></P>
<P class="p12 ft6">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Results
After resolving all the dependencies, installing OpenCV and porting SURF C++ legacy code on Xeon E5 and Xeon Phi 5110P
CPU hosts for offload and native programming models respectively. From the results shown in Table 2 and pie chart 9 we saw that
major portion of computation time for a serial code on respective hosts was taken by kernels Get Descriptor, Fast Hessian and
Orientation. An important observation is that the total time taken by Xeon Phi coprocessor host to run code natively was almost
twice as much as time taken by Xeon E5 host. This indicates two things that a single core of Intel Xeon E5 is more powerful than a
single core on Intel Xeon Phi and irregular memory access from the global memory, as in Fast Hessian kernel, are more costly on
Intel Xeon Phi than Intel Xeon E5.From the serial profiling results, it is indicated Getdescriptor () should be targeted, and
Orientation () also scales on the basis of interest points as Getdescriptor(), so optimize both the kerne ls in tandem.
![fig1](https://user-images.githubusercontent.com/15110492/31065172-4c1b3166-a6f8-11e7-9ee0-126fc693ec18.PNG?raw=true)
In order to take advantage of Xeon Phi coprocessors we offloaded code and respective data on to the Xeon Phi cores.
OpenMP(Offload & Native): (Fig 9) Multithreading in getDescriptors() and getOrientations() is offloaded to Intel Xeon Phi on the
60 cores we observed an unexpected dip in performance for the algorithm for Offload in OpenMP. Multithreading in
getDescriptors() and getOrientations() happens on Intel Xeon Phi on the 60 cores we observed some gains with respective to native
serial implementation. We also multithreaded the buildResponsemap() of the Fast Hessian kernel though only on Intel Xeon E5 host
and Intel Xeon Phi host for respective models. We observe that execution time of serial code running on Intel Xeon Phi is much
higher than serial code execution time on Intel Xeon E5 host. This dip in performance could be attributed to the fact that due to the
simultaneous accesses in multithreading, the memory access bandwidth requirement from the main memory increases drastically for
Intel Xeon Phi. The memory access are unaligned access of single FP data by each thread which leads to repeated main memory
accesses for the same memory location. The feature descriptor points/Interest points are normally concentrated around regions of an
image not evenly distributed which leads to repeated memory access to same memory location in the integral image. This exerts a
traffic bottleneck on the Ring Interconnect of the Intel Xeon Phi, so it was required to minimize the memory accesses. For this
memory access needs to be aligned and un-coalesced. By aligning the buffer memory access and vectorization some single memory
accesses are converted to SIMD memory access at aligned memory boundaries. Though still the gains are not as per our expectation,
so we look into some compiler optimizations. The thread affinity was set to scatter as SURF is a bandwidth bound application. Also
various combinations for the number of threads were targeted. We found sweet spot around 36 cores and 60 cores in offload and
native models respectively. We additionally introduce -fno-alias which instructs the Intel compiler to allow vectorization when
accessing the same memory location by different pointer spread across the threads or within the thread. The compiler in OpenMP
adds automatic prefetching at optimization level -O2 or higher.
Though we had an intuition to have considerable gains in OpenMP offload with respect to Xeon E5 serial code but results were
less than expected, this doubt was resolved by the profiling analysis of the optimized SURF which showed only 2.1x gains. The Pie
charts in Fig 11(b) of OpenMP indicate that communication overload has considerably increased for the offload. This could be
attributed to the transfer of integral image and memory chunk of the interest points. The fast hessian kernel now becomes the
dominant kernel in SURF algorithm as expected in spite of multithreading in the buildResponsemap(). This could be attributed to 
the fact that the memory access are still unaligned and un-coalesced in Fast Hessian and currently using C++ constructs and buffers.
The integral image kernel also shows a higher percentage as the numbers in other kernel gets reduced
![fig2](https://user-images.githubusercontent.com/15110492/31065182-58cb2e66-a6f8-11e7-8ab1-1db2535e4307.PNG)

Though we expected substantial speedup but figure of 2.2x compared to Serial Xeon E5 and 4x in comparison to Xeon Phi
Serial code execution are less than expected for Native OpenMP model. Our doubt was resolved again by the results of profiling
analysis done for optimized SURF Native kernel. The Pie charts in Fig 11(c) of OpenMP Native indicate the Fast Hessian kernel is
sole dominant occupant in the time taken according to the pie chart. We know that Fast Hessian kernel have a lot of aligned and uncoalesced
memory accesses as was with OpenMP Offload model also, but just a higher percentage was surprising. The fast hessian
kernel is also mostly single threaded except buildResponselayermap(). This makes Fast Hessian our next kernel to be targeted in
future to extract desired level of performance from this SURF algorithm in native model.
![fig3](https://user-images.githubusercontent.com/15110492/31065185-5a12368e-a6f8-11e7-9023-58a6244e3e29.PNG)

![fig5](https://user-images.githubusercontent.com/15110492/31065188-5ccf6860-a6f8-11e7-85a1-5a72e5999239.PNG)

OpenCL (Offload Model): Multithreading and vectorization alignment numbers can’t be separated for OpenCL implementations hence both optimizations were targeted simultaneously on OpenCL model. So actually red and grey tabs shown in 
Fig 9 under OpenCL Offload, combined delivers the performance. It is to be noted that multithreading and vectorization gains were
completely masked out until we specified level 2 prefetch specifically for the OpenCL kernels while compilation. The OpenCL
experienced gains of approx. 2.6X over serial Xeon E5 post optimization. The multithreading, vectorization and memory alignment
were done only for getDescriptors() and getOrientations() kernels,. The remaining two kernels of fast hessian kernel and integral
image behave in the similar fashion in OpenMP offload.The Pie charts in Fig 11(a) of OpenCL Offload shows a similar trend as
OpenMP Offload where communication and Fast Hessian kernel are primary occupants. . In the OpenCL model the communication
overhead is incurred at the last point when fetching the final results from the memory mainly and other overheads include
configuring the kernels from the host. For communication, mapped buffers with non-blocking access were used, still such increase
is slightly surprising. The Fast Hessian kernel will be the next target for OpenCL model as well same reason as OpenMP
implementation. We were able to Structure of arrays in OpenCL for all data buffers used in the descriptor calculations for OpenCL, 
but was not able to do the same of OpenMP model. So for future scope, we could attempt to implement structure of arrays for
memory coalesced. This may be one reason why the gains in OpenCl are more than both OpenMP models
The programming model of offload (OpenCL and OpenMP) incur a high communication overhead. The bottleneck and single
threaded kernels are more costly on Intel Xeon Phi (Native) than Intel Xeon E5 host. The aligned and coalesced memory access is a
bigger bottleneck in Intel Xeon Phi than Intel Xeon E5. As the number of cores increases simultaneously memory accesses also
increases in Xeon Phi, in turn the memory bandwidth issues compound further. The algorithm and code for OpenMP both native
model is same besides the communication part, and the impact of which is seen in the first call for getorientation(). To reduce this
offload communication, it needs to be done while before they are used in the offload model. The OpenCL model implements a
similar algorithm but with completely different coding styles. OpenCL is not performance portable and can’t work on Native.
OpenMP codes are performance portable across Intel Architectures even in native and offload model Intel Xeon Phi Model.
The Fig 10(b) based on number of Interest points just shows that higher the number of interest points the higher the serial code
execution time numbers, but after parallelization this dependency goes away as both the kernels depending upon the interest points
are relatively less dominant now.
</P>

</BODY>
</HTML>
