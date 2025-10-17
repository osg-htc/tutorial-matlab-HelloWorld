---
ospool:
    path: software_examples/matlab_runtime/tutorial-matlab-HelloWorld/README.md
---

# Create and use compiled MATLAB applications

## Introduction

This tutorial shows you how to create and use a compiled MATLAB applications in your jobs on the OSPool.

MATLAB is a licensed software unavailable by the OSPool—however, compiled MATLAB binaries can run on the OSPool. Use this guide to learn how to submit jobs that use MATLAB.

## Prepare to build your MATLAB application

[MATLAB®](http://www.mathworks.com/products/matlab/) is a licensed high level language and modeling toolkit. The [MATLAB Compiler™](http://www.mathworks.com/products/compiler/) lets you share MATLAB programs as standalone applications.  MATLAB Compiler is invoked with `mcc`. The compiler supports most toolboxes and user-developed interfaces. For more details, check the list of [supported toolboxes](http://www.mathworks.com/products/compiler/supported/compiler_support.html) and [ineligible programs](http://www.mathworks.com/products/ineligible_programs/).  

All applications created with MATLAB Compiler use [MATLAB Compiler Runtime™ (MCR)](http://www.mathworks.com/products/compiler/mcr/), which enables royalty-free deployment and use. **You must have access to a server that has MATLAB compiler** to build your application in the first section. The compiler is not available on the OSPool.

Although the compiled binaries are portable, they must a compatible, OS-specific MATLAB runtime to interpret the binary. We recommend the compilation of your MATLAB program against MATLAB versions that match the OSG [containers](https://portal.osg-htc.org/documentation/htc_workloads/using_software/available-containers-list/), with the compilation executed on a server with Scientific Linux so that the compiled binaries are portable on OSG machines.

!!! info "Check this before proceeding!"

    Make sure you have access to a server with the MATLAB Compiler (`mcc`) that is compatible with the MATLAB Runtime container you intend to use.


### Example MATLAB script: `hello_world.m` 

Lets start with a simple MATLAB script `hello_world.m` that prints `Hello World!` to standard output. This will be our application that we want to compilie.
    
```
function helloworld
    fprintf('\n=============')
    fprintf('\nHello, World!\n')
    fprintf('=============\n')
end
```  

### Compile your application

*The OSPool does not have a license to use the MATLAB compiler*.

On a Linux server with a MATLAB license, invoke the compiler `mcc`.  Turn off all graphical options (`-nodisplay`), disable Java (`-nojvm`), and instruct MATLAB to run this application as a single-threaded application (`-singleCompThread`):

```
mcc -m -R -singleCompThread -R -nodisplay -R -nojvm hello_world.m
```

The flag `-m` means C language translation during compilation, and the flag `-R` indicates runtime options.

After compiling, you should have the following files:

* `hello_world`
* `run_hello_world.sh`
* `mccExcludedFiles.log`
* `readme.txt`

The file `hello_world` is the standalone executable. The file `run_hello_world.sh` is MATLAB generated shell script. `mccExcludedFiles.log` is the log file and `readme.txt` contains the information about the compilation process.

To run our application on the OSPool, you will only need the standalone binary file `hello_world`. 

## Run standalone binary applications on the OSPool

To see which releases are available on OSG visit our available [containers](https://portal.osg-htc.org/documentation/htc_workloads/using_software/available-containers-list/) page:

### Tutorial files

We now have the standalone binary `hello_world`. Transfer the file `hello_world` to your Access Point. Alternatively, you may also use the readily available files by using the `git clone` command: 

```
$ git clone https://github.com/OSGConnect/tutorial-matlab-HelloWorld # Copies input and script files to the directory tutorial-matlab-HelloWorld.
```
 
This will create a directory `tutorial-matlab-HelloWorld`. Inside the directory, you will see the following files
   
```
hello_world             # compiled executable binary of hello_world.m
hello_world.m           # MATLAB program
hello_world.sub         # HTCondor submit file
hello_world.sh          # execution script
```

### Test the MATLAB application binary

The compilation and execution environment need to the same. The file `hello_world` is a standalone binary of the matlab program `hello_world.m` which was compiled using MATLAB 2018b on a Linux platform. The Access Point and many of the worker nodes on OSG are based on Linux platform. In addition to the platform requirement, we also need to have the same MATLAB Runtime version. 

Load the MATLAB runtime for 2018b version via apptainer/singularity command.  On the terminal prompt, type

```
apptainer shell /cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-matlab-runtime:R2018b
```

The above command sets up the environment to run the matlab/2018b runtime applications.

Now execute the binary:

```
$ apptainer/singularity> ./hello_world
```

which produces the folloing output:

```
=============
Hello, World!
=============
```

If you get the above output, the binary execution is successful. Now, exit from the apptainer/singularity environment typing `exit`. Next, we see how to submit the job on a remote execute point using HTcondor. 

!!! danger "Before you test MATLAB application binaries"
    
    In this tutorial, we test the `hello_world` binary directly on the Access Point since it is a simple program which does little-to-no compute. **Do not test your binary on the Access Point if it requires nonnegligible compute resources.** This can cause the Access Point to crash for you and other users. If you want to test your full binary, submit your test as an HTCondor job.

### Job execution and submission files

Let us take a look at `hello_world.sub` file: 

```
container_image = /cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-matlab-runtime:R2018b

executable =  hello_world                

output = log/job.$(Process).out
error =  log/job.$(Process).err
log =    log/job.$(Cluster).log

request_cpus = 1
request_memory = 1 GB
request_disk = 1 GB

queue 10
```

Before we submit the job, make sure that the directory `log` exists on the current working directory. Because HTcondor looks for `log` directory to copy the standard output, error and log files as specified in the job description file. 

From your work directory, type:

```
mkdir -p log
```

### Submit jobs

Submit your jobs using the `condor_submit` command:

```
condor_submit hello_world.sub
```

This submits a list of 10 MATLAB jobs. You can check the status of your jobs with the `condor_q` command:
    
```
condor_q
```

When your jobs are complete, each standard output file (i.e., `job.0.out`), should have the message, `Hello world` printed! We can confirm this with the command, `cat log/job.*.out`.

## What's next? 

Try compiling and testing your own MATLAB applications!

## Getting help
For assistance or questions, please email the OSG User Support team  at [support@osg-htc.org](mailto:support@osg-htc.org) or visit the [help desk and community forums](https://portal.osg-htc.org/documentation/).
