Welcome to CS264 Lab 1
======================

Justin Riley (staff 'at' cs264 'dot' org)

Software Tools for Academics and Researchers

Office of Educational Innovation and Technology

Massachusetts Institute of Technology

Office Hours
------------

**Date/Time**: Fridays 3:00pm - 5:00pm 

**Location**: NE-48-308, 77 Massachusetts Ave, Cambridge, MA 02139

Homework 0 Survey
-----------------
As part of homework 0 you are responsible for setting up all of the necessary user accounts in order to fully take advantage of the class. In order to fully complete homework 0 you *must* fill out the `homework 0 survey <http://www.cs264.org/homeworks/homeworks/hw0.php>`_. The survey will be at the very bottom of Homework 0.

Resonance GPU Cluster
---------------------
If you have not done so already you will need to create a SEAS account and request access to the resonance GPU cluster at Harvard. If you have issues with this please send an email to ircshelp@seas.harvard.edu. If you've just requested an account please wait at least a day before contacting the IRCS IT staff.

Once you have access to resonance, please use the `SSH Access to SEAS Hosts <http://ircs.seas.harvard.edu/display/USERDOCS/SSH+Access+to+SEAS+Hosts>`_ guide to create an SSH key and add your new key to your ~/.authorized_keys file on resonance.

Sign-up for an Amazon Web Services Account
------------------------------------------
**NOTE**: You will need a personal credit card in order to sign up for an Amazon Web Services Account.

To sign up for an Amazon Web Services account:

1. Sign up for AWS using either your existing Amazon account (if you have one) or by creating a new separate AWS account
2. After you've created your AWS account sign up for the EC2 web service. Use your AWS account email and password when prompted. Signing up for EC2 will automatically add you to the S3 web service which is required by EC2.
3. Next to obtain your AWS user ID navigate to your "Security Credentials" page.  
4. Scroll down to the bottom of the page and look for the "Account Identifiers" section. You should see your "AWS Account ID". Please save your AWS account id somewhere safe.
5. After you've retrieved your AWS account ID please complete the HW0 survey in order to receive your $200 worth of credit offered by Amazon for the course.

In order to familiarize yourself with the Amazon Web Services console please follow the instructions in Homework 0 to create a new SSH key for Amazon EC2.

Overview of CUDA Compiler Suite
-------------------------------
The CUDA compiler suite consists of the following core tools:

 - CUDA Compiler (nvcc) - converts your CUDA code to binary form
 - CUDA Debugger (cuda-gdb) - allows you to "pause" and inspect your code while it's executing
 - CUDA Memcheck (cuda-memcheck) - detects and reports memory access errors in your code
 - CUDA Visual Profiler (computeprof) - used to measure performance and find potential opportunities for optimization
 - and more...

Installing CUDA [optional]
--------------------------
You will be able to develop your code on the SEAS GPU cluster (resonance) or at the 53 Church St. computing labs, so no specific hardware is required for the course.

However, if you want to run your CUDA code on your own machine here's what you will need:

 * A (fairly) recent NVIDIA graphics card (see `here <http://www.nvidia.com/object/cuda_gpus.html>`_ for a list of supported devices).
 * CUDA 3.2 Toolkit
 * CUDA 3.2 SDK

You will need to `download CUDA 3.2 Toolkit/SDK <http://developer.nvidia.com/object/cuda_3_2_downloads.html>`_ for your OS and follow the install instructions provided by NVIDIA.

Debugging CUDA using cuda-gdb
-----------------------------
When you're developing CUDA code there will be bugs and errors that can be difficult to figure out just by compiling and running the code alone. This is especially true when the problem lies within the GPU kernel itself. This is because a GPU kernel runs exclusively on the GPU and on a GPU "print statements" have no meaning or effect. Fortunately the CUDA toolkit comes with a CUDA debugger that allows you to "step" through your code and inspect local variables both in CPU code and within your GPU kernel(s). The debugger also allows you to switch to different threads on the GPU and inspect the local variable stack of a given GPU thread.

In order to debug CUDA you must compile your code with the debug flags enabled. For example:

.. code-block:: none

        nvcc -g -G mycudacode.cu -o mycudacode

The -g flag means compile the CPU portion of the code with debugging enabled. The -G flag means compile the GPU portion of the code with debugging enabled. These options are needed so that you can debug your code on both the CPU and GPU.

Let's try compiling the example.cu code included with HW0 on the resonance GPU cluster with the debug flags enabled:

.. code-block:: none

        nvcc example.cu -o example -I$CUDASDK_HOME/C/common/inc -L$CUDASDK_HOME/C/lib -lcutil_x86_64

This will create an executable or "binary" called example in the current directory. 

Now that we've compiled the example CUDA code from Homework 0 we'll now use cuda-gdb to debug our code. Before we start here's a list of all the commands we'll use and some brief descriptions:

1. breakpoint (b) - set a "breakpoint" at key places in the code. the argument can either be a method name or line number
2. run (r) - run your application within the debugger. this must executed before the rest of these commands can be used.
3. next (n) - move to the next line of code
4. continue (c) - continue to the next breakpoint or until program ends
5. backtrace (bt) - shows you where you are in the code
6. thread - lists the current CPU thread
7. cuda thread - lists the current active GPU threads (if any)
8. cuda kernel - lists the currently active GPU kernels and also allows you to switch 'focus' to a given GPU thread. 

These commands are used throughout the following example.

Next we run the example code using the CUDA debugger. To do this we use the "cuda-gdb" command:

.. code-block:: none

        cuda-gdb --args example -string="tester"
        NVIDIA (R) CUDA Debugger
        3.2 release
        Portions Copyright (C) 2008-2010 NVIDIA Corporation
        GNU gdb 6.6
        Copyright (C) 2006 Free Software Foundation, Inc.
        GDB is free software, covered by the GNU General Public License, and you are
        welcome to change it and/or distribute copies of it under certain conditions.
        Type "show copying" to see the conditions.
        There is absolutely no warranty for GDB.  Type "show warranty" for details.
        This GDB was configured as "x86_64-unknown-linux-gnu"...
        Using host libthread_db library "/lib64/libthread_db.so.1".

This will bring you to a cuda-gdb prompt:

.. code-block:: none

        (cuda-gdb)

Next let's add some breakpoints. Breakpoints define places in the code to stop execution and allow the debugger to inspect the current state of threads and variables. In the example file we have main, mangleCPU, and mangleGPU methods. We will add a breakpoint at each method and run through 'stepping' through each line of these methods and inspecting local/global variables on different GPU threads. To set a breakpoint at each of these methods:
   
.. code-block:: none

        (cuda-gdb) b main
        Breakpoint 1 at 0x4053fa: file example.cu, line 43.
        (cuda-gdb) b mangleCPU
        Breakpoint 2 at 0x403f1e: file example.cu, line 105.
        (cuda-gdb) b mangleGPU
        Breakpoint 3 at 0x4041fb: file example.cu, line 98.

Now that we have the breakpoints set, the debugger will stop at each method and allow us to inspect the method's local variables. In order to reach a breakpoint we must first run the code:

.. code-block:: none

        (cuda-gdb) r
        Starting program: /home/jtriley/cudadb/example -string=tester
        [Thread debugging using libthread_db enabled]
        [New process 1072]
        [New Thread 47173892559264 (LWP 1072)]
        [Switching to Thread 47173892559264 (LWP 1072)]

        Breakpoint 1, main (argc=2, argv=0x7fff3f6d5448) at example.cu:43
        43      CUT_DEVICE_INIT(argc,argv);


Once the program starts it will run until a breakpoint is reached. In the above output we see that we're at Breakpoint 1 which stops at the main() method in our example.cu code. At this point let's inspect the CPU and GPU threads:

.. code-block:: none

        (cuda-gdb) thread
        [Current thread is 2 (Thread 47173892559264 (LWP 1072))]
        (cuda-gdb) cuda thread
        No CUDA kernel is currently running.

Here the "thread" command displays information on the current CPU thread and "cuda thread" shows the current CUDA thread we're inspecting. Since we haven't yet reached the mangleGPU kernel breakpoint the command "cuda thread" just prints a message that no CUDA kernel is running. 

For now let's skip past the main method and move to the mangleGPU method. We can allow the code to continue running until the next breakpoint using:

.. code-block:: none

        (cuda-gdb) c
        Continuing.
        Using device 0: Tesla T10 Processor
        [Launch of CUDA Kernel 0 (mangleGPU) on Device 0]
        [Switching to CUDA Kernel 0 (<<<(0,0),(0,0,0)>>>)]

        Breakpoint 3, mangleGPU<<<(1,1),(6,1,1)>>> (instr=0x100000 <Address 0x100000 out of bounds>, outstr=0x100100 <Address 0x100100 out of bounds>, len=6, x=1295995981)
            at example.cu:99
        99      int i = blockIdx.x * blockDim.x + threadIdx.x;

Now we see that we're at the third breakpoint: mangleGPU. Notice that we did not first go to Breakpoint 2 (mangleCPU). This is because the order that the breakpoints will occur depends on what gets executed first by the running program. In this case mangleGPU is called before mangleCPU so the mangleGPU breakpoint occurs first.

Now that we're at the mangleGPU breakpoint, you'll notice that the "cuda thread" command now lists a CUDA kernel thread. Let's switch 'focus' to that thread:

.. code-block:: none

        (cuda-gdb) cuda thread
        [Current CUDA kernel 0 (thread (0,0,0))]
        (cuda-gdb) cuda kernel 0
        [Switching to CUDA Kernel 0 (<<<(0,0),(0,0,0)>>>)]
        99      int i = blockIdx.x * blockDim.x + threadIdx.x;

Next we'll inspect some local variables on the GPU (blockIdx, threadIdx):

.. code-block:: none

        (cuda-gdb) p blockIdx
        $2 = {x = 0, y = 0}
        (cuda-gdb) p threadIdx
        $3 = {x = 0, y = 0, z = 0}
        (cuda-gdb) p i
        warning: Variable is not live at this point. Returning garbage value.

Notice that we can't yet access the integer "i" that gets created in the first line of the kernel. This is because we haven't yet stepped past this line. Let's step past:

.. code-block:: none

        (cuda-gdb) n
        100     MANGLE(instr,outstr,i,len,x);

Now that we're at the next line after 'i' has been defined in the kernel we can inspect the value of 'i':

.. code-block:: none

        (cuda-gdb) p i
        $4 = 0

Similarly we can switch to other GPU threads and check their value for 'i':

.. code-block:: none

        (cuda-gdb) cuda thread 1
        [Switching to CUDA Kernel 0 (device 0, sm 0, warp 0, lane 1, grid 1, block (0,0), thread (1,0,0))]
        100     MANGLE(instr,outstr,i,len,x);
        (cuda-gdb) p i
        $5 = 1
        (cuda-gdb) cuda thread 2
        [Switching to CUDA Kernel 0 (device 0, sm 0, warp 0, lane 2, grid 1, block (0,0), thread (2,0,0))]
        100     MANGLE(instr,outstr,i,len,x);
        (cuda-gdb) p i
        $6 = 2

After you're done inspecting "i" across GPU threads let's continue on to the next breakpoint:

.. code-block:: none

        (cuda-gdb) c    
        Continuing.
        [Termination of CUDA Kernel 0 (mangleGPU) on Device 0]
        [Switching to Thread 47173892559264 (LWP 1072)]

        Breakpoint 2, mangleCPU (instr=0xaf2d3e0 "tester", outstr=0xafa8700 " \206\n", len=6, x=1295995981) at example.cu:105
        105     for(int i=0; i<len; i++)

We're now at Breakpoint 2 (mangleCPU) which means we're now back on a CPU thread and there is no longer an active GPU thread. You can verify this:

.. code-block:: none

        (cuda-gdb) thread
        [Current thread is 2 (Thread 47173892559264 (LWP 1072))]
        (cuda-gdb) cuda thread
        No CUDA kernel is currently running.

We can now step through this function line by line and inspect variables:

.. code-block:: none

        (cuda-gdb) n
        106         MANGLE(instr,outstr,i,len,x);
        (cuda-gdb) n
        105     for(int i=0; i<len; i++)
        (cuda-gdb) n
        106         MANGLE(instr,outstr,i,len,x);
        (cuda-gdb) n
        105     for(int i=0; i<len; i++)
        (cuda-gdb) p i
        $7 = 1
        (cuda-gdb) n
        106         MANGLE(instr,outstr,i,len,x);
        (cuda-gdb) n
        105     for(int i=0; i<len; i++)
        (cuda-gdb) print i
        $8 = 2

Now that we're done inspecting the code, let's exit the debugger. We can either continue and let the program finish executing or just type quit at the next command prompt:

.. code-block:: none

        (cuda-gdb) continue
        Continuing.
        Current date/time: (1295995981) Tue Jan 25 17:53:01 2011
        Input string:      tester
        CPU result:        teetet
        GPU result:        teetet

        Program exited normally.
        (cuda-gdb) quit

For more info on how to use the CUDA debugger please consult the cuda-gdb user manual.

Using CUDA Memcheck to Detect Memory-Access Errors
--------------------------------------------------
The CUDA memcheck tool helps to detect errors in your code related to memory-access issues. There are two ways to use the CUDA memcheck utility. You can use the cuda-memcheck command line tool to simply run your code and report the errors it finds:

.. code-block:: none

    cuda-memcheck example -string='hi there class'

    ========= CUDA-MEMCHECK
    Using device 0: Tesla T10 Processor
    Current date/time: (1296164555) Thu Jan 27 16:42:35 2011
    Input string:      hi there class
    CPU result:        sishtiitetst c
    GPU result:        sishtiitetst c
    ========= ERROR SUMMARY: 0 errors

This approach will simply print out any errors it found. In this case since the example.cu code included in Homework 0 does not have any memory errors the number of errors reported is 0. 

Another option is to use the memcheck utility within the CUDA debugger. This has the advantage that when a memory-access error is detected cuda-gdb will immediately drop you to a shell and allow you to inspect the state of the world. This approach is a bit more complicated given that you have to launch and manage the debugger but it can be extremely powerful when fixing those hard to find bugs.

.. code-block:: none

    (cuda-gdb) set cuda memcheck on
    (cuda-gdb) r
    Starting program: memcheck_demo
    [Thread debugging using libthread_db enabled]
    [New process 23653]
    Running unaligned_kernel
    [New Thread 140415864006416 (LWP 23653)]
    [Launch of CUDA Kernel 0 on Device 0]

    Program received signal CUDA_EXCEPTION_1, Lane Illegal Address.
    [Switching to CUDA Kernel 0 (<<<(0,0),(0,0,0)>>>)]
    0x0000000000992e68 in unaligned_kernel <<<(1,1),(1,1,1)>>> () at
    memcheck_demo.cu:5
    5
    *(int*) ((char*)&x + 1) = 42;
    (cuda-gdb) p &x
    $1 = (@global int *) 0x42c00

    (cuda-gdb) c
    Continuing.
    Program terminated with signal CUDA_EXCEPTION_1, Lane Illegal Address.
    The program no longer exists.
    (cuda-gdb)

The above output is from the code included in the `cuda-memcheck user manual <http://people.maths.ox.ac.uk/gilesm/cuda/doc/cuda-memcheck.pdf>`_. This code purposefully makes misaligned and out of bound memory accesses in order to demonstrate the behavior within cuda-gdb.

Profiling CUDA code using the Visual Profiler
---------------------------------------------
The CUDA profiler


 * Launch the CUDA visual profiler using the "computeprof" command
 * In the dialog that comes up press the "Profile application" button in the "Session" pane.
 * In the next dialog that comes up type in the full path to your compiled CUDA program in the "Launch" text area.
 * Provide any arguments to your program in the "Arguments" text area. Leave this blank if your code doesn't take any arguments.
 * Make sure the "Enable profiling at application launch" and "CUDA API Trace" settings are checked
 * Press the "Launch" button at the bottom of the dialog to begin profiling.

The profiler will now run and analyze your code execution times, memory usage patterns, etc for each function in your code. When it's finished it will display these stats in an excel spreadsheet-like fashion. You can also plot any given column of data for each method by right-clicking the column and selecting "Column plot". This is useful in determining which functions are your bottlenecks and might need fixing or refactoring.

For more info on how to use the CUDA debugger please consult the `CUDA visual profiler user manual <http://developer.download.nvidia.com/compute/cuda/3_2_prod/toolkit/docs/VisualProfiler/Compute_Visual_Profiler_User_Guide.pdf>`_.

Source Code Control (GIT)
-------------------------
There is a git server setup by SEAS for this course that is available to you while you develop the solutions to the homework problems and for your final projects. Time permitting, let's take a quick look at using git for simple version control.

 * One of the most used distributed version control systems
 * Originally written by Linus Torvalds for supporting Linux Kernel development
 * Available in most free software distributions
 * Available for many operating systems (including Linux/Mac/Windows)
 * Forges based on it available: Github, Gitorious, etc.

**Links:**
 * GIT Homepage (http://git-scm.com/)
 * Mac GIT Client (http://code.google.com/p/git-osx-installer/)
 * Windows GIT Client (http://code.google.com/p/msysgit/)
