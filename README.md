
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
Orientation () also scales on the basis of interest points as Getdescriptor(), so optimize both the kernels in tandem.
</P>
![fig1] https://user-images.githubusercontent.com/15110492/31065172-4c1b3166-a6f8-11e7-9ee0-126fc693ec18.PNG
</BODY>
</HTML>
