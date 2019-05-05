---
title: Burn GPU's Calories (3)
date: 2019-05-05 11:59:46
tags:
---
**Q: From on the previous 2 blogs, I've grasped the main idea of parallelism in CPU. I think it is already very powerful, so why do people use GPU to do intensive computing? Why not add more and more cores in CPU?**
A: Because of the complex structure and design pattern of CPU cores, it is impossible to put thousands of cores in a CPU chip. However, GPU has a totally different design and the 'core' in GPU is extremely simplified (only the arithmetic related units are remained), so that to achieve massive parallelism for large scale intensive computation, GPU is the best choice for now.
**Q: I see, so the core in CPU is very powerful, but that in GPU is very weak. Is there any quantitative way to measure that?**
A: Of course. The speed clock in CPU's core is about 2~5GHz, but it's is about 500-1300MHz in GPU's core. In a CPU's core, it has multiple ALU, FPUs, so may support multi-threading execution. However, in a GPU's core (actually called CUDA core, named by NVIDIA), it has only one ALU and/or one FPU, so no multi-threading capability. Look at the picture below
![CPU](/images/cpu_gpu_core.png)
<!-- more -->
The CPU's core is like a math professor in a college, but the GPU's core is similar to a student in a primary school. To do an intensive computing task in most science and engineering, a famous mathematician is not necessarily faster than 2000 students, because all the tasks are just simple arithmetic operations.
**Q: I see. The massive parallelism really changes things. You mean the GPU is designed to meet this emerging requirement in science and engineering?**
A: Not really. The history is a little complex. In fact, initially, the GPU card was just for visualization purpose in gaming. Because there are many pixels (10K~100M) in a screen and their colors (RGB values) need to be updated 20-60 times every each second, if this task is sent to CPU, the pixels have to be updated one by one (or several by several depending on number of cores in CPU), which is not affordable, especially with the increasing screen resolution and refreshing rate. For example, the refresh rate of a gaming monitor, ASUS ROG Swift PG279Q, can be overclocked to 165Hz!
**Q: That's impossible for CPU to handle.**
A: Yes. The initial GPU is only for gaming purpose, but later on, maybe some scientists found why not use this high parallel structure to do computation, e.g. particle simulation? Then the general purpose GPU (GPGPU) was created, and then more and more scientists and engineers, such as people in academics, oil companies, NASA and many national labs, purchased GPUs to accelerate their simulations. A big market was emerging suddenly. Many small GPU manufacturers were created to cater this market.
**Q: Who is the biggest winner up to now?**
A: NVIDIA. Nowadays, there are only three key players in ths market: Intel, AMD and NVIDIA. Two of them are actual CPU manufacturers as well and they keep incorporating GPUs into their CPUs, while NVIDIA keeps making discrete GPUs as an extension to CPUs. Nowadays, when you see a NVIDIA logo attached on a desktop PC or laptop, it means the NVIDIA GPU card is installed. Here is the history of market value of Intel and NVIDIA.
![CPU](/images/intel_nvidia_value.png)
The NVIDIA is almost like boosted by a rocket during the recent 5 years.
**Q: Impressive rising. The Intel has a history of 50 years, but the NVIDIA is only 26 year old. Now they have comparable values. I think NVIDIA will eat most of the GPU market soon.**
A: Not really. NVIDIA is only popular in desktop environment, while in mobile market, a completely different set of palyers emerge: QualComm and Broadcom. Therefore, in the future there are still many possibilities.
**Q: I see...what kind of GPUs would you like to talk about? Integrated or discrete?**
A: NVIDIA's discrete GPUs, because they are widely used in many supercomputers like TOP1 Summit by OakRidge National Lab. The GPUs provide its 95% computing power.
**Q: Hard to imagine. It seems like my previous knowledge of CPU is going to be useless soon.**
A: No. The CPU is very important, because the current GPUs are not processors but "co-processors". They work under the supervision of a host CPU. All of the initial data in GPU must come from CPU via the connection, e.g. PCI Express bus. This mode is almost like a commander + workhorses in figure below
![CPU](/images/cpu_gpu_commander.png)
Like the radio cards and wifi units, the GPUs are just extensions in PCI bus, after its driver is installed, the mechanism of GPU is very similar to the Pthreads in my last blog. Programmers just need to create functions for each GPU core (like thread in CPU) and let it run!
**Q: Hmm...perfect! If they are similar, now I can do the GPU programming using my existing development environment, e.g. LINUX+GCC？**
A: No...not that easy case. Because CPU cores and GPU cores are completely different hardwares, so they have different "instruction set architectures" (ISA), which means they use different languages. Image, a commander uses English to give orders to the workhorses who can only understand spanish. That won't work. GCC can only translate your code to the language used by CPU, so GPU can not understand it and will do nothing.
**Q: Reasonable. Things are more complex than I thought. It seems we need to develop a new language understood by GPU.**
A: Yes. In GPU, the assembly level instructions are quite different from CPU, but it is not a realistic thing to let our scientists and engineers learn a brand new programming language just to accelerate computation. In fact, in some early stage of GPUs, some geeks used some low-level languages to program for GPUs, but of course, it was too hard. Therefore, some ideas come out, like CUDA and OpenCL. Here I will only talk about CUDA (nvcc compiler) released by NVIDIA in 2007. To alleviate the pain to learn a new language, it just defines many new keywords based on C language, therefore the learning is much easier. Also for compiling, "nvcc" compiler can automatically seperate the host code and the device code and compile them into different assembly level codes for different hardwares.
**Q: This is very helpful for all learners. Before programming, could you show some structures of GPU? I still wonder how it can have thousands of cores in it.**
A: Sure. Look at the figure below
![CPU](/images/gpu_struct.png)
It looks very similar to CPU, but different from CPU, GPU has no L3 cache. The last level cache (LLC) is L2. All the computing units are integrated into so called streaming multiprocessor (SM). It is pretty much like the physical core in CPU from outside, but the inner structures are quite different. See the picture below
![CPU](/images/gpu_sm.png)
Compared with physical cores in CPU, there are not so many built-in controllers, but many cores inside. As I said before, each core has 1 ALU and/or 1 FPU. Another difference is the register files are shared by all cores. This is the reason why some people don't want to call the processing units in GPU "cores", because the cores don't have their own registers. Also there are many Load/Store units (LD/ST) to handle memory access between cores and caches. SFU is for special function calculation, like sin(), cos(), square root, logarithm, and exponential functions.
**Q: I know instruction cache is to store machine code for execution. What's warp and warp scheduler?**
A: Good question. Warps make programming tricky but computing much faster. Because of some inner design in hardware, every 32 cores constitute a "super-thread". If the logics of these 32 cores are the same, they can execute simultaneously, which means there is no divergence. However, if the divergence happen, the diverged parts have to be executed serially by different cores in that warp. Like the figure below
![CPU](/images/warp.png)
This feature puts more pressure to programmers to try to avoid warp divergence for performance purpose. Actually, programming on GPU usually needs more rounds of coding optimization before the production run, because there are many features like warp in GPU, which can influence the performance significantly.
**Q: It seems like the CPU takes over much burden from programmers to itself.**
A: Yes. CPU has many advanced features, which can do much optimiztion for programmers. Even a new programer writes a moderately good code, but it is still possible that CPU can make it run perfectly through super-scalar, piplining techniques automatically. However, for GPU, it puts all pressure to programmers to handle every details. It is the price that programmers must pay for the superior parallelism. Today's knowlege is already a little bit much. Next time, we will dive into GPU programming.
**Q: See you next time.**
