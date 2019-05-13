---
title: Vectorization I
date: 2019-05-12 22:53:25
tags: 
mathjax: true
---
**Q: Recently, I saw many new product launching presentations of smart phones. To run those bigger and bigger mobile video games, the mobile phones are gettting faster and faster.**
A: Exactly. However, I want to say although smart phones are upgraded to cater to the emerging apps, many apps are far from exploiting all the hardware resources yet.
**Q: Really? Doesn't the mobile apps run always faster on more recent devices?**
A: Not always. Actually it may be slower in most cases.
**Q: Oh? Why? The CPUs are faster and faster, right?**
A: No, not that case. See this figure. The CPU's frequency has stagnated for around 10 years. However, the number transistors and logical cores are increasing rapidly, which lead to the speedup of microprocessors.
**Q: Hmm... I don't know much details about these parameters, but why the frequency can not be improved evidently?**
A: It relates to a lot of physics. To make it short, the power cost will increase exponentially with the increasing clock speed. Also, as a result, the generated heat can not be handled by the cooling system. It will burn. This is the reason why vendors are turnning to multi-core strategy.
**Q: Hmm... I see. Let's dot try to improve the clock speed. But I think if an computer or smartphone has more cores, the apps should be faster than running on a single core, because we have more workers, right?**
A: Not the case by nature. Let me first say the "frequency". The frequency is a free lunch for developers, because it determines the execution speed of each instruction, thus programmers don't need to pay any special attention on it. It means that, on a 1GHz processor, a program can run 10 times faster than that on a 100MHz one without any change of the code. However, a program can not exploit the multi-cores automatically, because the original program was designed to run on a single core. While it is running, other cores are just idle there. To use the power of multi-cores is the work of programmers.
<!-- more -->
**Q: So... programmers, nowadays, have to manipulate the multi-core hardware directly while programming? That will be very difficult.**
A: No. If that's the case, most of programmers will be laid off, because that will be a very steep learning curve. Also, different OS and types of machines have different ways to manipulate the multi-core, thus developers would need to write several versions of the same app to run on different platforms, which will be a huge burden.
**Q: I believe if that's the case, there must be some genius trying to solve it. One saying in computer science is "there must be some connection by interlayers between a gap."**
A: Exactly. In the computer world, it always will happen. In fact, in the past decades, many individals, institutions, universities and companies have developed many tools to make multi-core programming much easier, like OpenMP, which makes parallelism on shared memory machines much easier.
**Q: Sounds great! Is OpenMP a library? Sometimes, it is not an easy job to compile and link an library to user's programs. Especially for the libraries with poor installation guides.**
A: No. It isn't. It is just a set of APIs built in your compiler, like GNU GCC, LLVM, Intel C Compiler, IBM XL etc. The OpenMP is evolving, and the compilers keep updating their supports to OpenMP (see [OpenMP Compilers][1]). You don't need any extra installations besides your compiler.

**Q: I like this design, a very clean way. Now everything is ready (just compiler, no extra preparation!), could we start?**
A: Sure, but could you tell me what compiler you use?
**Q: Just GNU GCC compiler.**
A: In fact, GCC is a great opensource and free compiler, but to learn the parallelism technique, I want to start from a commercial compiler: Intel C/C++ compiler: icc/icpc. It has been optimized for parallelism for a long time. Also it has more interesting functions.
**Q: I can't afford its price probably.**
A: Every student has a one-year free license. It has been included into the Intel Parallel Studio XE, see[Intel Parallel Studio XE][2].
**Q: That's great!**
A: Parallelism has a hierarchy with many differnt layers. We can start from the simplest case. What's the simplest operation you can think about in your code?
**Q: Addition operation to two scalars or vectors. Because I am doing research on computational science, the most operations are arithmetic. I want to parallelize the addition operations on vectors. For example, $c[i]=a[i]+b[i]$, and each array $a[\ ],\ b[\ ],\ c[\ ]$ has 100 elements. I want to add those 100 elements at the same time. Is it possible?**
A: Yes. There are many ways to realize that. I call them different parallelism levels:
![parallel_layers](/images/parallel_layers.png)
1. The highest level is you can split the 100 elements and send one element per array $a[i]$ and $b[i]$ to 100 CPU cores and do the addition operations. At the end, one core gather all $c[i]$ from other 99 cores. As you see, there is very much communication between cores. In fact, it's hard to have one single machine with 100 cores, so usually you have like 5 machines, we call them nodes, with 20 cores for each. An problem comes here. Because machines are normally connected via Ethernet network, whose bandwidth is about 10 Gigabit per second, the cost on communication is the dominated factor of speedup. Of course, some latests clusters use Infiniband hardware with higher bandwidth [Infiniband Wik][4]. Even for those cores in the same machine, their memories are not shared. Each core has its own part of physical memory on that node. Unlike the cross-machine communication, within one machine, cores can communicate with each other with a much faster speed (see DDR4's bandwidth is ~60 Gbit/s [DDR4 bandwidth][5] and also the MPI library and OS will do some optimization on this), so the cross-node message passing is the bottle-neck.
2. The second highest level is that you can put those 100 elements to the 20 cores within one machine. Each core will take 5 elements from array $a$ and $b$ and do 5 addition operations. This time, some cores can share the memory, thus there is no need to send and receive any data between cores. This is the so called multi-threading technique.
3. The next level is the parallelism on a single core. There are some concepts, which I even don't know much right now, like pipelining, superscalar execution, out-of-order execution and so on. Currently, I will only focus on a situation called vectorization. It is easy to guess what it will do. For the previous example, if we only use a single core to add array $a$ and array $b$, we don't need to add their elements one by one, like adding scalars intead of vectors. Many modern CPU supports vectorization in hardware, because it has the so-called vector register, which can be used for summing up a segment of array $a$ and $b$ within one clock cycle. A simple picture is below
4. Other types of parallelism, for example add some other devices (e.g. GPU) to help CPU on the computation intensive tasks. This topic has been covered in one of my previous blogs in [GPU blog][6].

**Q: Hmm... so many levels of parallelism. parallel programming sounds very complex. If only we can increase the CPU frequency more! Now there will be no free lunch any more. Programmers have to learn how to make their applications run in parallel to scale.**
A: Exactly, you are right. The CPU frequency almost approaches the physical limit. We have to admit this fact. Before a new technology comes up and replace the current technology, computer scientists need to think about how to scale. Parallelism seems to be the most promising one. Actually, most of current programs, e.g. various apps in your mobile phone, do not exploit the benefits of the multi-core hardware, so software should catch up!
**Q: Really? That's a huge waste. I want to learn parallel programming. Could we start from the vectorization?**
A: Sure. It is in a relatively low level of parallelism. It's hard to imagine how multiple arithmetic operations can be done using one clock cycle. It must be implemented in the hardware, right?
**Q: Agree. Software can not solve this kind of problems.**
A: Yes. At the beginning, all the CPUs are scalar processors, which can only deal with scalars. Then some scientists think about how to speedup. Because in many numerical code, operations on a large vectors are very common, so that each element of a array executes the same instruction. They want to let the CPU execute those same instructions at the same time, so vector processors come out. It can't be realized from software side, it is a new type of CPU. This kind of CPU can conduct arithmetic operation on a set of several scalars, which is called a "vector". Before talking about the vector addition, do you know what happens when two scalars are added in CPU?
**Q: Yes. The CPU firstly retrieve these two scalars into registers, and then add them up using the ALU, and finally write the result back to the memory. Is that right?**
A: Right. Similarly, now we want to add 2 arrays, each has 4 elements. The "powerful" CPU firstly retrieve these two vectors into "vector registers", and then add $a[i]$ and $b[i]$ for $i=0,1,2,3$ using a single "vector" instruction, like below
![parallel_layers](/images/vectorization.png)
**Q: Amazing. Which kind of CPUs can support vectorization?**
A: I only know a little about Intel CPUs, so I can only tell you the Intel stuff for now, but it should be similar to AMD cores. The vectorization capability of Intel CPU is supported by the instruction set. It has been evovling for almost two decades from 64-bit MMX (meaningless initialism[MMX-Wikipedia][8]) to 128-bit SSE (Streaming SIMD Extentions), SSE2, SSE3, SSE4.2, and then to 256-bit AVX (Advanced Vector Extentions), AVX2 and now the 512-bit AVX-512. The number in xx-bit means how many bits can be stored in a vector register where the data can be processed together. Of course, the larger the vector register is, the better the performance will be.
**Q: Hmm... these names are very unfamiliar for me. Normally when I buy a computer, it writes like Intel i3, i5 or i7. How can I know their instruction sets?**
A: The i3/i5/i7 belongs to the Intel Core series. This series is mainly for general purpose, like gaming, graphics, or office. You can check their instruction sets in Intel website (e.g. [Intel Core i7-8565U][9]). The i7 can only support up to AVX. Usually for scientific computing, we should use Intel Xeon series because it has more cores inside and better vectorization. For example, the Intel Xeon Phi 7250 has 68 cores and supports AVX-512. ([Intel Xeon Phi 7250][10]).
**Q: I see. Even the Intel CPU has many kinds for different purposes. OK, let's go back to the vectorization. Assume I have a Intel Xeon machine, which supports vectorization, do I need to do something to let the machine or my program to exploit this feature? Just like my smartphone, it has many physical sensors to sense the rotation and movement of my phone, but it will only function after I install a racing game app. I remember my teacher never talked about vectorization when I took the C/C++ course in the first year of my undergraduate.**
A: Good question. The vectorization is more related to the hardware, so it will be too much if it is included into a C/C++ course. The instruction set is what the CPU can do, which we can not change, and the code, which is in high level, is what our programmers can do. It seems that if a programmer doesn't pay any attention on vectorization when he/she is programmin, the AVX-512 instruction set will be wasted. Is it right?
**Q: It is right for me. We programmers have to tell the machine "go to execute these additions using vectorized instruction" explicitly via some code, right?**
A: Correct but not common. For Intel CPUs, there is a good way to exactly control the vectorization, called Intel Intrinsics. The intrinsics is pretty much like a language between C and Assembly. It will be directly translated into corresponding assembly-level vectorized instruction by the Intel Compiler, but it is hardware dependent, thus less flexible. Here is an example. The following function is a intrinsics to multiply a and b, then add the intermediate results with c. 
{% codeblock lang:c%}
__m128d _mm_fmadd_pd (__m128d a, __m128d b, __m128d c)
{% endcodeblock %}
"fmadd" means "fused multiplication and addition". "pd" means "packed double precision floating numbers", where "packed" is equivalent to "vector", which is opposite of scalar. "__m128d" is a data type of length = 128-bit. This kind of 128-bit number will be stored in xmm vector register (128-bit). There are 3 different kinds of vector registers shown in following table

| vector register      | length (bit) | supported ext.|
|------|:------------:|:-------:|
| xmm|128|SSE, AVX, AVX512|
| ymm|256|AVX, AVX512|
| zmm|512|AVX512|

Therefore, to use ymm vector registers (if supported), programmers have to call another intrinsics, 
{% codeblock lang:c%}
__m256d _mm256_fmadd_pd (__m256d a, __m256d b, __m256d c)
{% endcodeblock %}

which makes too much trouble when tranplanting a code from one machine to another, because the intrinsics is hardware-dependent. Normally, we won't do that.
**Q: That puts too much burden to programmers to develop as well as maintain. Is there any better way? like a wrapper of intrinsics?**
A: Correct! There is one "wrapper"! But as far as I know, there is only one explicit directive, which is hardware independent, to guide the vectorization. It is a directive provided by OpenMP:
{% codeblock lang:c%}
    #pragma omp simd
{% endcodeblock %}

"simd" means single intruction multiple data, which is similar to the idea of vectorization. If a loop is acted by the above directive, the compiler will vectorize the body of loops forcely.
{% codeblock lang:c%}
    #pragma omp simd
    {
      for (i=0; i<n; i++) {
        c[i] = a[i] + b[i];
      }
    }
{% endcodeblock %}

The array a[] and b[] will be put to available vector registers (xmm, ymm, or zmm) in the current hardware. No need to worry about data type or vector length for programmers. The only thing is to make sure there is no data dependence between the components of vectors. A bad example of vectorization is
{% codeblock lang:c%}
    #pragma omp simd
    {
      for (i=2; i<n; i++) {
        a[i] = a[i-1] + a[i-2];
      }
    }
{% endcodeblock %}

The above vectorization will produce wrong results because the array a[]'s elements are dependent to each other. 
**Q: This directive is very useful. I think it's enough for vectorization, right?**
A: Not really. There are many other things you must be careful about, which will be talked about next time!

Note:
Most content in this post comes from the online free course "Fundamentals of Parallelism on Intel Architecture" available at [https://www.coursera.org/learn/parallelism-ia][11].


  [1]: https://www.openmp.org/resources/openmp-compilers-tools/
  [2]: https://software.intel.com/en-us/c-compilers
  [3]: http://static.zybuluo.com/felix007/n8bzocec8palw8rpmfbifiyr/parallel_layers.png
  [4]: https://en.wikipedia.org/wiki/InfiniBand
  [5]: https://www.techspot.com/news/62129-ddr3-vs-ddr4-raw-bandwidth-numbers.html
  [6]: https://feilinx.com/2019/05/05/burn-gpu-calorie-3/
  [7]: http://static.zybuluo.com/felix007/76jk9gf71b16ratc0knkuuja/vectorization.png
  [8]: https://en.wikipedia.org/wiki/MMX_%28instruction_set%29
  [9]: https://www.intel.com/content/www/us/en/products/processors/core/i7-processors/i7-8565u.html
  [10]: https://ark.intel.com/products/94035/Intel-Xeon-Phi-Processor-7250-16GB-1_40-GHz-68-core
  [11]: https://www.coursera.org/learn/parallelism-ia
