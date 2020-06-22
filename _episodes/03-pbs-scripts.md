---
title: "Writing PBS submission scripts"
teaching: 30
exercises: 10
questions:
- "How do we communicate with the HPC scheduler?"
objectives:
- "Learn how to write job submission scripts."
keypoints:
- "The PBS 'scheduler' runs jobs on Artemis, according to _**directives**_ supplied by you."
- "The '**Project**' directive governs everything that happens on Artemis."
- "The **resources** you request affect when your job can run, and if it will finish!"
---

This episode introduces the **_scheduler_** and how to communicate with it. Primarily, this will be through '_**PBS directives**_' in _Bash_ scripts.

## The 'scheduler'

**Artemis HPC** is a _computer cluster_ -- a network -- whose _**resources**_ are assigned to users by a _**scheduler**_. The cluster's resources are the CPUs, RAM and GPUs which belong to all the constituent computers in the network. Curently, Artemis has:

* 7,636 cores (CPUs)
* 45 TB of RAM
* 108 NVIDIA V100 GPUs
* 378 TB of storage

These resources are distributed between all of its computers, which we sometimes call 'machines', and often refer to as _**nodes**_. It's important to realise that these are real computers, and each node actually has a _set number_ of CPUs and RAM etc that form it -- it's not all one huge computer.

In order to make the most efficient use of this network, a very complicated algorithm tries to allocate nodes and their resources to different users who want them, with the twin aims of having no resources sitting idle and as little waiting time as possible for the users. Clearly, these aims are in conflict!

<figure>
  <img src="{{ page.root }}/fig/03_cluster.png" style="margin:10px;height:200px"/>
  <figcaption> A cartoon scheduler -- compute jobs will try to be resourced as efficiently as possible! </figcaption>
</figure><br>

The **scheduler** does these computations. Artemis' scheduler is a version of ['PBS Pro'](https://www.pbsworks.com/PBSProduct.aspx?n=PBS-Professional&c=Overview-and-Capabilities). The scheduler maintains a **queue** of computing jobs that users have submitted to be run. It constantly recomputes the best order in which to run them all, based on what each job needs, what resources are available, and how long a job has been waiting for. Spare a thought for the much put-upon scheduler!

### Cluster resources

The primary cluster resources available to compute jobs are:

* **ncpus**: the number of CPU cores
* **mem**: the amount of RAM
* **ngpus**: the number of GPU cores
* **walltime**: the length of time all these resources will be made available to the requesting job

However, as we noted, these resources aren't freely floating, but are tied to specific nodes. Therefore, when specifying HPC resources, we need to do so _at the node level_.

For example, the character string ```select=1:ncpus=8:mem=16GB```   
specifies **1** node (```select=1```), with **8** CPUs (```ncpus=8```) and **16 GB** of RAM (```mem=16GB```).

The string ```select=3:ncpus=2:mem=6GB```   
specifies **3** nodes (```select=3```), _each with_ **2** CPUs (```ncpus=2```) and **6 GB** of RAM (```mem=6GB```).

The string ```walltime=01:30:00``` specifies 1 hour and 30 minutes of elapsed time as measured by a clock (on the wall!). The time string is specified as **HH:MM:SS**, for hours:minutes:seconds.

<br>
#### Your Project is your key

Cluster resources used are 'billed' to your **project**. Although use of Artemis is free for USyd researchers, your usage will be tallied against your _RDMP_, which defines your project.<sup id="a1">[1](#f1)</sup> In order to maintain fair access to all researchers, a count of recent resource usage by each project is maintained, and those projects with lower recent usage will be given a small amount of priority in the scheduling queue.

This recent tally is called your '_fair share_ weighting'. You can't really do anything about it, and you can't even check your _fair share_ either, so our advice is don't worry about it!<sup id="a2">[2](#f2)</sup>

> ## Your Project is your key!
>
> Let's say this again: **Everything** that can happen on Artemis is managed by _projects_.
>
> You need an Artemis-enabled _project_ to gain access to Artemis.
>
> You need to specify a _project_ when requesting Artemis resources.
>
> You can only access the **/project** and **/scratch** directories of _your project_.
>
> _**Project**_!!  :smirk:
{: .keypoints}

<br>
### Estimating resource requests

You might be wondering, "How do I know how much X to request??". This is a good question, and one without an easy answer. You definitely don't want to **underestimate** your job's requirements, because if you do your job will likely fail and be killed by the scheduler. But you also don't want to overestimate **too much**, because then you'll be requesting huge resources that the scheduler will have trouble finding for you (resulting in longer queue time).

Some good approaches are:
* If you have ever run the code before, use the specs of the computer you ran it on as a starting estimate. Ie, if I know a program runs on my laptop, and takes 2 hours, then I know that my **4 cores** and **16GB RAM** for **2 hours** are enough to run the job to completion.
* If you can run your job on only a subset of the data (or _runs_, or _training epochs_, etc) then run it on only a couple 'units' of your data/runs and see how much resources it uses. You can start by requesting **2 CPUs** and **8 GB** of RAM for **2 hours**. You can then run it on more units, and perhaps even try to estimate a scaling relationship to guess how much running N units of data will require.
* If all else fails, start by requesting a small amount, say **2 CPUs** and **8 GB** of RAM for **2 hours**, and keep increasing until your job completes successfully!

> ## Using multiple CPUs (cores)
>
> Not all software _can_ use more than 1 CPU core.
>
> If requesting multiple cores, make sure your software can use them. Otherwise, they'll just be sitting idle, and you'll be waiting longer for resources you won't use to become available.
>
> **Check the documentation** of the programs you wish to use, and look out for options or flags to enable _multi-threading_ or _multi-cpu_ operation. This is sometimes also called '_OpenMP_'.
{: .callout}

> ## Using multiple _**chunks**_
>
> Not all software _can_ spread over _multiple chunks_.
>
> If requesting multiple chunks (with **select=n**, _n>1_), make sure your software can use them. Otherwise, they'll just be sitting idle, and you'll be waiting longer for resources you won't use to become available.
>
> **Check the documentation** to see if your program supports distributed computing, usually called **MPI** (Message Passing Interface). If not, requesting more than one chunk won't help!
>
> Note that if you _can_ use multiple chunks, the scheduler will have an easier time finding you, eg, 16 CPUs spread over _different_ chunks, than _1 chunk of 16 CPUs_. Something to keep in mind!
{: .callout}

### Job queues

The Artemis queue is broken down into sub-queues, called _**job queues**_, which each have different resources and limits allocated to them. Depending on what resources you wish to use, you will be assigned to a particular queue. In some cases, you'll need to choose a special queue to use particular resources, such as GPUs. This will be discussed further in the [next Episode]({{ page.root }}/04-submitting-jobs).

<br>
### Scheduler 'PBS' directives

We communicate with the scheduler through the use of 'PBS **directives**', which are simply instructions that we pass to the scheduler when we _submit_ a job, like options passed to a program. These directives are invoked using option **flags**, eg ```-P <project_name>``` for the 'project name' directive.

The main command that we use to submit jobs to the scheduler queue is ```qsub```. Scheduler directives are passed along as options to the ```qsub``` command, eg ```qsub -P <project_name>```.

#### Common PBS directives

The most common PBS directives are listed below. **The only compulsory directive is -P (project)**. Without specifying a project name, an attempted job submission will _fail_.

| Flag | Argument | Notes |
| --- | --- | --- |
| ```-P``` | Project _short name_ | **Required directive!** |
| ```-N``` | Job name | Name it whatever you like; no spaces |
| ```-l``` | Resource request | Usually a ```select:``` or ```walltime:``` block |
| ```-q``` | Job queue | Defaults to ```defaultQ```  |
| ```-M``` | Email address | Notifications can be sent by email on certain events |
| ```-m``` | Email options: **abe** | Receive notification on (**a**)bort, (**b**)egin, or (**e**)nd of jobs |
| ```-J``` | Job array indices ```i-j:k```| Integers; indices will be **i** to **j** in steps of **k** |
| ```-I``` | Interactive mode | Opens a shell terminal with access to the requested resources |
| ```-X``` | X-Window forwarding; use with -I | Allows use of GUIs in interactive jobs |
| ```-W``` | Further arguments | See manual page for more info: ```man qsub``` |

<br>
## PBS scripting

There are a number of ways to pass the PBS **directives** to the ```qsub``` submission command. However, the most flexible and reliable way is to write a **submission script**. We call these 'PBS scripts', after the name of the scheduling software, 'PBS Pro'.

A **PBS script** is simply a _shell script_: a text file that lists commands to send to your command interpreter '_shell_'.<sup id="a3">[3](#f3)</sup> On Artemis, the default shell is '[Bash](https://www.gnu.org/software/bash/)' ("Bourne-again shell"); so we'll also sometimes call these 'Bash scripts'. A **PBS script** is a _bash script_ written to be executed by the PBS scheduler's ```qsub``` command -- which means it contains PBS **directives** to tell the scheduler what we want it to do.

Let's take a look at a real life **PBS script** to make all this concrete.

Navigate to our Training project directory -- the **/project** folder for **RDS-CORE-Training-RW**:

~~~
cd /project/Training
~~~
{: .language-bash}

Locate the data archive we'll be using for this course, in the **DATA** folder:

~~~
ls -lsh DATA
~~~
{: .source}

~~~
[jdar4135@login3 Training]$ ls -lsh DATA/
total 2.5G
1.7G -rw-r-xr-- 1 nbut3013 RDS-CORE-Training-RW 1.7G Jul  9  2019 Automation.tar.gz
244M -rw-r--r-- 1 nbut3013 RDS-CORE-Training-RW 244M Feb 14  2019 Sample_data.tar.gz
 25M -rw-r--r-- 1 nbut3013 RDS-CORE-Training-RW  25M Sep  7  2017 cfa_ref_CanFam3.1_chr5.fa.gz
244M -rw-r--r-- 1 nbut3013 RDS-CORE-Training-RW 244M Jun 22 17:15 data_hpc.tar.gz
4.0K -rw-r--r-- 1 nbut3013 RDS-CORE-Training-RW  671 Jul  9  2019 dogScripts.tar.gz
244M -rw-r--r-- 1 nbut3013 RDS-CORE-Training-RW 244M Feb 14  2019 sample_data.tar.gz
~~~
{: .output}

These files are ```.tar``` archives, like ```.zip``` files you might be more familiar with. They are made and read using the ```tar``` command. We'll be using **sample_data.tar.gz**.

Before "untar"ing it, create a working directory for yourself. Since we'll all be working on the same files, we can't all do that in **/project/Training**, as we'd either overwrite each other's, or get '_Permission denied_' errors if we tried.

Create your own directory with the 'make directory' command ```mkdir```, naming it whatever you like, then change into it. I'll use my name, but you should obviously substitute the directory name you have chosen:

~~~
mkdir hayim
cd hayim
~~~
{: .language-bash}

Now untar (decompress) the data archive into your directory:

~~~
tar -xzvf ../DATA/data_hpc.tar.gz
~~~
{: .language-bash}

~~~
[jdar4135@login3 hayim]$ tar -xvzf ../DATA/data_hpc.tar.gz
datahpc/
datahpc/hello.pl
datahpc/canfam3_chr5.fasta
datahpc/align.pbs
datahpc/134_R2.fastq.gz
datahpc/index.pbs
....
~~~
{: .output}

The option flags ```-xzvf``` mean e**x**tract files, use G**z**ip compression (for ```.tar.gz```), be **v**erbose and print what is being done, and use the archive **f**ile specified.

As can be seen in the output above, the archive contains a folder 'sample_data', and this folder has been recreated in your working directory. (Check this by running ```ls``` to list the current directory contents!). For convenience, let's move (```mv```) all the files (```*```) out of this extra folder, and remove it (```rmdir```). Since I am currently in my ```hayim``` working directory, and I want the files here also, I'll use the 'here' shortcut (```./```) as my destination argument:

~~~
mv datahpc/* ./
rmdir datahpc
~~~
{: .language-bash}

Finally, make a copy of the **index.pbs** file, naming the copy **basic.pbs**, and open it in your text editor of choice:

~~~
cp index.pbs basic.pbs
nano basic.pbs
~~~
{: .language-bash}

or if you prefer to use 'GEdit': ```gedit basic.pbs &```.

<figure>
  <a name="nanobasic"></a>
  <img src="{{ page.root }}/fig/03_nanobasic2.png" style="margin:10px;height:500px"/>
  <figcaption> The <b>basic.pbs</b> PBS script. </figcaption>
</figure><br>

The PBS script **basic.pbs** is constructed from 3 main sections. Let's take a look at them. But first, take note of the very first line of the script, ```#!/bin/bash```. This declaration is called the _shebang_, or _hash-bang_, because it literally starts with a hash **#** and an exclamation mark **!**.

This first line actually tells the _shell_ how to 'interpret' everything that follows -- by indicating what _command interpreter_ the commands are written for, and where its program is stored (in ```/bin```). In our case, **basic.pbs** is a script written for 'Bash', using Bash language and commands.

Also note the second line of the script, which contains a _**comment**_. This is a message to _human_ readers of the script (such as its author!), and is ignored by your computer. In Bash, comments begin with a **#** character.

### 1. PBS directives declaration

This is the first section of our **PBS script**. Whilst the first line was a directive for the shell, these lines contain directives for the **PBS scheduler**. Each line here contains a PBS **directive**, which we discussed earlier.

Each PBS directive begins with the phrase ```#PBS```.

At this point you might be thinking, "Hang on -- don't the shebang and PBS directives all begin with a **#**, indicating a _comment_?". And you'd be right! These are _all_ comments, in the sense that they are not executed commands, but are all 'read' by someone. Normal comments are read by humans; the shebang is read by the shell; and the **#PBS** lines are read by the PBS program -- it looks out for these lines, and interprets them as its own directives.

~~~
#PBS -P Training
#PBS -N Index_YourName
#PBS -l select=1:ncpus=1:mem=4GB
#PBS -l walltime=00:10:00
#PBS -q small-express
~~~
{: .language-bash}

Each of these lines declare a PBS directive, telling the scheduler how we would like it to execute our job. Without them, the scheduler has no way to know what resources our job will need. In this example

- The nominated **project** has short-name _Training_
- We have set a **job name** _Index_YourName_
- We have requested **resources** of _1 chunk_ with _1 cpu_ and _4 GB RAM_
- We have requested **resources** of _10 minutes wall time_
- We have requested the _small-express_ job **queue**

Other directives, such as email notifications, have not been set, and are not required.

Only the **Project** ```-P``` is actually compulsory -- the scheduler will choose defaults for other necessary unset directives. These defaults are:

| Directive | Flag | Default value |
| --- | --- | --- |
| Compute | ```-l``` | select=1:ncpus=1:mem=4GB |
| Wall time | ```-l``` | walltime=01:00:00 |
| Job queue | ```-q``` | defaultQ |

### 2. Loading modules

The next part of our PBS script is reserved for loading Artemis software **modules**. Software can be loaded any time before its used, but for clarity and to minimise human error, it's good practice to place all such loading calls at the top of a script (the lilac [section](#nanobasic) above).

~~~
# Load modules
module load bwa/0.7.17
module load samtools/1.6
~~~
{: .language-bash}

Here we have loaded two bioinformatics programs: **bwa** and **samtools**. Note that we have also specified the **version** of the programs we are asking _modules_ to load for us. This is because software packages sometimes change significantly from version to version, using different _syntax_ to run them, or even different _methods_ internally -- which can lead to different results on the same data!

***Specifying the versions of software you use is key for scientific reproducibility.***


### 3. Program commands

Finally, we are ready to tell the scheduler what we want it to actually do -- the programs and commands we wish to run. At this point, it is also helpful to first declare any variables or aliases/abbreviations we'd like to use. Eg, we might not want to type out a long directory path name multiple times, so we can create a Bash variable for that path. In the above example:

~~~
io=/project/Training/YourName
~~~
{: .language-bash}

creates a variable '**io**' which can later be retrieved by using Bash's **$** referencing syntax, as ```$io```. This code is the green section of our [script](#nanobasic):

~~~
io=/project/Training/YourName

# BWA index:
bwa index -a bwtsw ${io}/canfam3_chr5.fasta

# SAMtools index:
samtools faidx ${io}/canfam3_chr5.fasta
~~~
{: .language-bash}

This part of your script will always be the most variable, as it depends most on the software and your project. In fact, you could store the 'directives' section only as a _template_ script that you base all other scripts on.

Program function calls generally follow a stereotypical pattern:

~~~
PROGRAM [SUB-FUNCTION] [OPTIONS] INPUT [OUTPUT]
~~~
{: .source}


Everything after the **program name** are called the **arguments**. The ```[]``` bracketed arguments above are more likely to be optional, and ```[]``` is the general convention for indicating optional arguments.

In the example above, the program ```bwa``` has a sub-function ```index```. To this is passed an option flag ```-a``` followed by its value ```bwtsw```, and then finally an input file ```${io}/canfam3_chr5.fasta```, using our directory variable ```io```.

In general, program options will be specified by a **flag** followed by its desired **value**, though some options that don't have multiple values will just be invoked by their **flag**. Options also often have _long_ and _short_ names; eg to specify an output file you may be able to write either ```--output=filename.ext``` or simply ```-o filename.ext```.

***Finding help with commands?***

The full range of functions, options and usages that a program offers can be found by invoking its ```--help``` option (assuming you've loaded the program with **Modules** first)

Sometimes more information is provided in the program's **manual page**, which can usually be loaded by ```man PROGRAM```, eg ```man samtools```. However, note that not all programs are consistent! Some don't have the ```--help``` command engaged. Its good practice but not essential for software developers. Manual pages and help commands are generally alike.

***Try these commands and notice the output***
~~~
samtools --help
~~~
{: .language-bash}

~~~
man samtools
~~~
{: .language-bash}


<br>

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup> To get access to **Artemis HPC** you must have a valid Unikey and be a member of an RDMP (Research Data Management Plan) with Artemis access. RDMPs are managed in the **[DashR](https://dashr.sydney.edu.au)** _Researcher Dashboard_ portal.

<sup id="f2">2[↩](#a2)</sup> Your _fair share_ count decays with a half life of 2 weeks. The contribution of resource usage to your _fair share_ count depends on the queue your job runs in. More on this in [Episode 4]({{ page.root }}/04-submitting-jobs).

<sup id="f3">3[↩](#a3)</sup> A _shell_ is a user interface for communicating with the operating system, ie the computer's main software layer. Shells can be 'command-line' (CLI) or graphical (GUI). It's called a 'shell' because it sits around the operating system's 'kernel', which contains the core programming that actually drives the hardware.

___
<br>




{% include links.md %}
