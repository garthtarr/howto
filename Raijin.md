## R on Raijin

This document gives a brief overview on how to use R (and RStudio) on the National Computational Infrastructure.  The full list of supported applications can be found [here](http://nci.org.au/nci-systems/national-facility/peak-system/raijin/application-software/).  

### Getting an account on Raijin

Raijin, named after the Shinto God of thunder, lightning and storms, is a Fujitsu Primergy high-performance, distributed-memory cluster.

The system, which was installed in late 2012, and which entered production use in June 2013, comprises:

* 57,472 cores (Intel Xeon Sandy Bridge technology, 2.6 GHz) in 3592 compute nodes;
* 160 TBytes (approx.) of main memory;
* Infiniband FDR interconnect; and
* 10 PBytes (approx.) of usable fast filesystem (for short-term scratch space).

The memory specification across the nodes is heterogeneous in order to provide a configuration capable of accommodating the requirements of most applications, and providing also for large-memory jobs. Accordingly:

* Two-thirds of the nodes have 32 GBytes, i.e., 2 GBytes/core;
* Almost one-third of the nodes have 64 GBytes, i.e., 4 GBytes/core; while
* Two per cent of the nodes have 128 GBytes, i.e., 8 GBytes/core.

Follow the instructions [here](http://maths-intranet.anu.edu.au/itdocs/doku.php#nci_high-performance_cluster) to get an account.  Note that there are **two steps**.  First you need to get an individual user account and secondly you need to attach that user to the MSI group (fh0).  It is important that you give them your mobile number as that's where they send your password that you'll need to log in to Raijin.

### Logging in to Raijin

Log in to the Raijin "head node" using ssh

```
ssh -Y <username>@raijin.nci.org.au
```

E.g. for me it is:

```
ssh -Y ggt251@raijin.nci.org.au
```

The `-Y` is important because it enables [X11](http://en.wikipedia.org/wiki/X_Window_System) which is required for RStudio to work over a ssh connection.  On a Mac you'll need [XQuartz](http://xquartz.macosforge.org/landing/) or on Windows you'll need [Xming](http://sourceforge.net/projects/xming/) installed.

Your password will have heen sent to your mobile when you registered to join the MSI group.

Once you're in, the prompt will look something like this:

```
[ggt251@raijin5 ~]$
```

where `ggt251` is your username.  The number after raijin tells you which of the 6 possible login (head) nodes you're on.  

To run R you first need to load the R module:

```
module load R/3.1.0
```

then you can either run in the command line by typing `R`.  Using R is moderately well [documented](http://nci.org.au/nci-systems/national-facility/peak-system/raijin/application-software/).  You can request an updated version of R to be installed on module by contacting [help@nf.nci.org.au]().  To use version R 3.1.1, for example, you would need to do the following steps (assuming version 3.1.0 was already loaded):

```
module unload R/3.1.0
module load R/3.1.1
```

RStudio is not centerally supported by NCI, but it is installed locally for MSI users (users who belong to the `fh0` group).  To access it you use the following command (after loading the R module as above),

```
/short/fh0/rstudio-0.98.1028/bin/rstudio
```

The head node should only be used for prototyping and code testing, you're sharing the resources of the node with everyone.  To use Raijin properly you need to bury in further to one of the compute nodes.

### Using RStudio in interactive batch job mode

To access a compute node interactively (i.e. in the same sort of way that you access a head node) use the following command:

```
qsub -I -l walltime=00:10:00,mem=500Mb,ncpus=1,wd -P fh0 -q express -X
```

The argument `walltime=hh:mm:ss` specifies how long the interactive session will last. This is used to schedule your "batch job" - even though you're using the node interactively it is still allocated like a batch job.  If you terminate the session early MSI will only be charged for the amount of time actually used, but it's best not to use a smallish value here because it will be scheduled faster.

The argument `mem=500Mb` says that 500Mb or RAM will be allocated and `ncpus=1` allocates 1 core - if your code is designed to run on multiple cores or needs more RAM to run, you need to ask for it here.

The `-P fh0` argument is required - it specifies that the time charges will be allocated the MSI group.  You probably don't have the required permissions to charge to any other groups.

You can asked to be put in the priority queue using `-q express`.  This is recommended when running an interactive batch job because otherwise (using `q- normal`) you might have to wait a while before your request is scheduled.  The downside to using `-q express` is that the amount charged to MSI is  3 times the actual walltime.  The `express` should really only be used for  testing, debugging etc.  It also has smaller limits - the max for `ncpus` is 128 and max memory per core is 32GB.

Once you've accessed an interactive batch job your prompt will look something like this:

```
[ggt251@r3151 ~]$
```

In this case, I was allocated to the 3151 node (it will be a number between 1 and 3592).  The resources you asked for are yours and yours alone for the time you asked for.  Once that time has elapsed (or you type `exit`) you will be logged out and the resources will go back to the scheduler to redistribute.

You can now run R or Rstudio just as you would have on the head node (or your own computer).

### Batch processing R scripts

When running a simulation or a program that you expect to run for quite some time it makes more sense to run the script in *batch mode*.

#### Inputs

To do this you need an R script and a batch script (a special plain text file).

The batch script can be created using RStudio on Raijin when you're at the login node level (or on your personal computer) by selecting `File > New File > Text File`.  The structure of the batch script needs to look like this:

```
#!/bin/csh
#PBS -l wd
#PBS -q normal
#PBS -l walltime=00:00:30
#PBS -l mem=50MB
#PBS -l ncpus=1
#PBS -P fh0
module load R/3.1.0
R CMD BATCH batch.R
```

Note that these are the same arguments as used in the interactive batch job mode.  The tricky one is `walltime` if this is too long, then you're wasting resources (which are limited, MSI has a finite walltime allocation).  Worse, if it is too short any unsaved results will be lost.  Identifying an appropriate walltime takes practice and experience.

The lines after the `#PBS` inputs get executed at the command line.  So as before, the first thing to do is load the R module, then the next line effectively opens R and runs the file `batch.R`.

In this example the contents of `batch.R` are:

```
### Test batch script
rm(list=ls())
N=100
n=1000
m1 = m2 = vector(length=n)
for(i in 1:n){
  x = rnorm(N)
  m1[i] = mean(x)
  m2[i] = median(x)
}
save.image(file="image.RData")
```

Note the `rm(list=ls())` at the start and the `save.image` function at the end.  This clears the workspace first then at the end saves the workspace so you can load it later and look at the results.  Of course, you can be more targetted in what and how you save the results, but this is a start.

Once you've got both an R script and a batch script you run it using

```
qsub batchfile
```

where `batchfile` is the name of the plain text batch script file.

* To check the process of your batch jobs you can use the `nqstat_anu`
* If you want to cancel a batch job use `qdel JOBID`

To find out more about the PBS (Portable Batch System) job submission and scheduling system see [here](http://nci.org.au/services-support/getting-help/pbspro-commands/).

#### Batch R script with arguments

If you want to run an R script multiple times with different arguments you need to put a line like this at the bottom of your batch file:

```
R CMD BATCH '--args arg1=5 arg2="A"' batch.R
```
Then at the start of your R file you need
```
args=(commandArgs(TRUE))
for(i in 1:length(args)){
  eval(parse(text=args[[i]]))
}
````
These two changes pass the variables `var1=5` and `var2="A"` to R when it starts running `batch.R`. Note that `var1` and `var2` can be whatever name you want, reflecting variables used later on in the `batch.R` file.

#### Outputs

After the run time has passed the you'll notice a few extra files in your working directory. One has the output (what code was run) and another has any errors.  In this case it will be `batchfile.o****` and `batchfile.e****` where `batchfile` is the name of the batch script used and `****` is a number.

You can quickly inspect these files using the `cat` command:

```
cat batchfile.o****
cat batchfile.e****
```

Successful jobs should have an empty error file.  Things to look out for in the error file are  

* Command not found.
* =>> PBS: job terminated: walltime 172818sec exceeded limit 172800sec
* =>> PBS: job terminated: per node mem 2227620kb exceeded limit 2097152kb
* Segmentation fault.

You'll also have the RData file which you saved your workspace to.  You can open this on the Raijin head node or download it to your computer.  You can load RData files into a R using the `load` function:

```
load("~/PATH/image.RData")
```

### Transferring files from Raijin to your computer

We can use the `scp` (secure copy) command to transfer files between Raijin and your computer.

First locate where the file is on Raijin.  To get a list of files in the current directory you can use `ls`.  However, if you want to get the full path to a given file, you need to use something like

```
readlink -f FILENAME
```

For example if there is a file called `batch.R` in the current working directory (i.e. you can see it in the list when you type `ls`) then you can get the full path by typing
```
readlink -f batch.R
```

Which (on my account) outputs something like:
```
/home/251/ggt251/batch.R
```

We need this full path when using `scp`.  To copy a file from Raijin to your personal computer, open a new terminal window on your personal computer (i.e. do not SSH into Raijin) and enter the following:
```
scp USERNAME@raijin.nci.org.au:/home/251/USERNAME/FILENAME .
```
Note that the `.` at the end signifies that the folder on your personal computer that the files will be copied to is the current working directory.  You can find out what the current working directory is by typing `pwd` (print working directory).  You could optionally specify another directory on your personal computer instead of using `.`

To give a concrete example, my username is `ggt251` and I want to copy `batch.R` from Raijin to my personal computer.  I would use this:
```
scp ggt251@raijin.nci.org.au:/home/251/ggt251/batch.R .
```

Say you have a bunch of RData files saved in a folder on Raijin and want to transfer all of them to your personal computer, you can use something like this:
```
scp USERNAME@raijin.nci.org.au:/home/251/USERNAME/*.RData .
```

The `*` is a wildcard charater meaning that it will look for all files that end in .RData in the `/home/251/USERNAME/` folder and transfer all of them to your computer.

If you'd like to transfer in the other direction, i.e. from your personal computer to Raijin, the process is very similar:
```
scp LOCAL/PATH/TO/FILENAME ggt251@raijin.nci.org.au:/home/251/ggt251/
```

Note 1: if you want to `scp` directories you need to use the `-rp` flag.  For example if I want to copy my entire Raijin directory to the current working directory on my personal computer I'd use:
```
scp -rp ggt251@raijin.nci.org.au:/home/251/ggt251/* .
```

Note 2: if you're using **zsh** (instead of, for example, bash) you will need to escape the wildcard.  This means using `\*` wherever you see `*`.


### Parallel processing in R

Now that you've got access to lots of cores, the easiest way to use it (in a simulation) is to replace big loops using the [foreach](http://cran.r-project.org/web/packages/foreach/index.html) package.  The only trick is that you need to register a "parallel backend" - [doMC](http://cran.r-project.org/web/packages/doMC/index.html) works well for unix systems (such as Raijin and macs but won't work on Windows).

Specifying the .combine argument allows you to customise how the results are aggregated at the end of the loop.

```{r}
require(doMC)
require(foreach)
registerDoMC(cores=3) # this should equal ncpus
n=10
result = foreach(j = 1:n, .combine=rbind) %dopar% {
  # EXPERIMENT
  # last line is returned as a row in the result matrix
  rep(j,4)
}
result
```

### Further resources

* <http://nci.org.au/services-support/training/introduction-nci/>
* <http://cran.r-project.org/web/packages/foreach/vignettes/foreach.pdf>
