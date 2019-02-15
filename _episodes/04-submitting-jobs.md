---
title: "Submitting and monitoring Artemis Jobs"
teaching: 35
exercises: 15
questions:
- "What submission options are available?"
- "How do I monitor the status of a job?"
objectives:
- "Gain experience editing PBS scripts."
- "Practice submitting jobs to Artemis."
- "Learn how to monitor and troubleshoot jobs."
keypoints:
- "```qsub``` submits your jobs to Artemis!"
- "Use ```qstat``` and ```jobstat``` to monitor job status"
- "Artemis has different **job queues** for different job types."
---

In this episode we practice **submitting** and **monitoring** jobs on Artemis. We also explore the different **job queues** available.

## Artemis Job Queues

Artemis' scheduler reserves portions of the cluster for different job types, in order to make access fairer for all users. However, the most commonly used queues are not specified in a resource request, but are allocated automatically based on what resources are requested. These primary queues all fall under the **defaultQ** (which is also the default queue set for any job that does not specify its queue with a ```-q``` directive).

The queues under **defaultQ** are _small_, _normal_, _large_, _highmem_ and _gpu_. Jobs are allocated to them according to the resource limits set for each queue: jobs will only run in a queue whose limits it satisfies. Recall that resources are requested with ```-l``` directives. The resource limits for each queue are listed below. Importantly, as soon as GPU resources are requested, jobs are assigned to the GPU queue -- there is no other queue with GPU resources.

| Queue | Invocation | Max Walltime  | Max Cores per<br>Job / User / Node | Memory (GB) per<br>Node / Core | Fair Share Weight |
|:--|:--:|:--:|:--:|:---:|:---:|
| -small<br>-normal<br>-large<br>-highmem<br>-gpu  |<br><br>**defaultQ**<br><br>| 1 day<br>7 days<br>21 days<br>21 days<br>7 days  | 24 / 128 / 24<br>96 / 128 / 32 <br>288 / 288 / 32<br>192 / 192 / 64<br>252 / 252 / 36 | 123 / < 20<br>123 / < 20<br>123 / < 20<br>123-6144 / > 20<br>185 / -- | 10<br>10<br>10<br>50<br>50  |   
| small express | **small-express** | 12 hours  | 4 / 40 / 4  | 123 / --  | 50  |   
| scavenger | **scavenger**| 2 days  | 288 / 288 / 24 | 123 / --  | 0 |   
| data transfer | **dtq** | 10 days  | 2 / 16 / 2  | 16 / --  | 0 |   
| interactive | ```qsub -I```  | 4 hours  | 4 / 4 / 4 | 123 / --  | 100 |

Take note of the maxima for each queue. Note especially the _maximum cores per node_: if you request more than this number of CPUs in a ```-l select=``` directive, your job **can never run**. The highest limit for CPU _cores / node_ is 64, as this is the number of cores of the largest CPUs available on Artemis.

Each queue also has a different contribution factor to your _Fair Share_ count. For example, use of **small-express** will accumulate Fair Share 50 times faster than using the **defaultQ**.

There are also a number of additional queues which are not part of **defaultQ**. These are:

| Queue | Purpose |
|:---:|:---|
| small-express | For quick jobs that require few resources. |
| scavenger | Allows jobs to use any idle resources available in other people's _allocations_; however, **your job will be suspended if the allocation owner requests those resources!**<br>Suspended scavenger jobs will be **killed** after 24 hours. |
| dtq | This queue is reserved for transferring data into or out of Artemis. Users may **not** try to perform computation in these queues, and the system generally won't let you. |
| interactive | This is the queue for _interactive_ jobs. It can only be accessed via a ```qsub -I``` command. |


#### Allocations

**Scavenger** uses idle resources available in _allocations_. An allocation refers to Artemis resources (ie nodes) which have been assigned to certain research groups for priority use. Allocations can be purchased, won, or granted via the [Facilities Access Scheme](https://informatics.sydney.edu.au/services/fas/). Remember, your **scavenger** jobs will be _paused_ (suspended) if the allocation owner requests those resources; they'll be _killed_ if they are suspended for longer than 24 hours. This makes **scavenger** an excellent option if you have many small jobs that you can easily re-run if they happen to be killed; some users get thousands of 'free' CPU-hours from scavenging!


## Submitting Jobs

### Adjusting PBS directives

We're now ready to **submit** a compute job to Artemis. Navigate to the **sample data** we extracted, and open **basic.pbs** in your preferred text editor.

~~~
nano basic.pbs
~~~
{: .language-bash}

<figure>
  <a name="nanobasic"></a>
  <img src="{{ page.root }}/fig/04_nanobasic2.png" style="margin:10px;height:420px"/>
  <figcaption> The <b>basic.pbs</b> PBS script. </figcaption>
</figure><br>

We need to make a few edits before we can submit this script. Can you guess what they are?

> ## Change #1
> Specify your **project**.
>
> Use the ```-P``` PBS directive to specify the _**Training**_ project, using its _short name_.
> ~~~
> #PBS -P Training
> ~~~
> {: .language-bash}
{: .solution}

> ## Change #2
> Give your job a **name**
>
> Use the ```-N``` PBS directive to give your job an easily identifiable name. You might run **lots** of jobs at the same time, so you want to be able to keep track of them!
>
> ~~~
> #PBS -N Basic_hayim
> ~~~
> {: .language-bash}
> Substitute a job name of your choice!
{: .solution}

> ## Change #3
> Tailor your **resource** requests.
>
> Use the ```-l``` PBS directive to request appropriate compute **resources** and **walltime** for your job.
>
> This script will not be asked to do much, as it'll just be a first test, so request just **1 minute** of walltime, and the minimum RAM, **1 GB**.
> ~~~
> #PBS -l select=1:ncpus=1:mem=1GB
> #PBS -l walltime=00:01:00
> ~~~
> {: .language-bash}
{: .solution}

> ## Change #4
> Specify a **job queue**.
>
> Use the ```-q``` PBS directive to send the job to the **defaultQ** queue. You can also try **small-express** if you like; whose jobs start sooner?
> ~~~
> #PBS -q defaultQ
> ~~~
> {: .language-bash}
{: .solution}


> ## Optional: Set up email notification
> Set up **email notification** for your job.
>
> Use the ```-M``` and ```-m``` PBS directive to specify a destination email address, and the events you wish to be notified about. You can receive notifications for when your job **(b)**egins, **(e)**nds or **(a)**borts.
> ~~~
> #PBS -M hayim.dar@sydney.edu.au
> #PBS -m abe
> ~~~
> {: .language-bash}
{: .solution}

To begin, we're going to run a very simple 'test' job, so <u>delete everything below the directives</u>, from ```# Load modules``` onward, and replace with

~~~
cd /project/Training/<YourName>

mkdir New_job
cp basic.pbs New_job/copy.pbs
perl hello.pl <YourName>
~~~
{: .language-bash}

PBS uses your **/home/\<unikey\>** directory as your default _working directory_ in which to start all PBS scripts. Since your data is not likely to ever be in your home directory, the first command in any script will probably involve setting or changing to the correct folder.

The rest of these commands (i) create a new folder, (ii) make a copy of the **basic.pbs** file within the new folder, and (iii) run a '_Perl_' script using the ```perl``` programming language and interpreter. The script **hello.pl** accepts one argument.

Save this PBS script (on nano <kbd>Ctrl</kbd>+<kbd>o</kbd>), and exit the editor (on nano <kbd>Ctrl</kbd>+<kbd>x</kbd>).

### Submitting PBS scripts to Artemis

Finally, submit the PBS script **basic.pbs** to the Artemis scheduler, using ```qsub```:

~~~
qsub basic.pbs
~~~
{: .language-bash}

~~~
[jdar4135@login3 hayim]$ qsub basic.pbs
2556851.pbsserver
~~~
{: .output}

Congratulations, you have now submitted your first Artemis job! :tada::tada:

## Monitoring Artemis jobs

### qstat

You've submitted a job, but how can you check what it's doing, if it's finished, and what it's done?

Note that when we submitted our PBS script above, we received the feedback

~~~
2557008.pbsserver
~~~
{: .output}

This number, **XXXXXX.pbsserver** is the **job ID** number for this job, and is how we can track its status.

We can query a current Artemis job using the PBS command ```qstat``` and the **job ID**. The flag ```-x``` will give us job info for historical jobs, so include it to see our job even if it has already completed

~~~
qstat -x 2557008
~~~
{: .language-bash}

~~~
[jdar4135@login3 hayim]$ qstat -x 2557008
Job id            Name             User              Time Use S Queue
----------------  ---------------- ----------------  -------- - -----
2557008.pbsserver Basic_hayim      jdar4135                 0 Q small
~~~
{: .output}

**qstat** shows that my job, **2556851.pbsserver**, with the name I gave it, **Index_hayim**, is currently _queued_ in the **small** job queue. The _**job status**_ column ```S``` shows whether a job is **(Q)**ueued, **(R)**unning, **(E)**xiting, **(H)**eld, **(F)**ininshed, or is a **(B)**atch array job. More status codes can be found in the docs (```man qstat```).

**qstat** has many other options. Some common ones are:

| Flag | Option |
|:--:|---|
| ```-T``` | show an _estimated Start Time_ for jobs |
| ```-w``` | print wider output columns, for when your job names are longer than 10 characters |
| ```-u``` | show jobs for a specific _user_ (Unikey) |
| ```-x``` | show finished jobs also |
| ```-f``` | show full job details |
| ```-Q``` | print out numbers of jobs queued and running in for all of Artemis' job queues |

Print out the entire job list, by not specifying a **job ID**, with estimated start times, by running ```qstat -T```. How far down are our training jobs?

### jobstat

Artemis provides another tool for checking your jobs, which also shows some extra information about Artemis. This is ```jobstat```:

~~~
[jdar4135@login3 hayim]$ jobstat
Job Summary for user jdar4135
                                                Requested -------------------------             Elapsed  --------------------------
Job ID---- Queue- Job Name--- Project--- State- Chunks Cores GPU       RAM Walltime  Start Time CPU Hours   CPU% Progress End Time
2557008    small  Basic_hayim Training   Queued      1     1   -     1.0Gb       1m  (n/a)          (n/a)      -        - (n/a)
 * Times with an asterix are estimates only
 * End time is start time + walltime so job may finish earlier
 * Progress is accumulated walltime vs specified walltime - so see above

System Status ----------------------------------------------------------------------------------------------------
CPU hours for jobs currently executing: 1482286.8
CPU hours for jobs queued:              489049.4
Storage Quota Usage ------------------------------------------------
/home                             jdar4135       5.214G          10G
/project              RDS-CORE-Training-RW       34.07G           1T
/project                   RDS-CORE-CLC-RW           4k           1T
/project                   RDS-CORE-ICT-RW       514.3M           1T
/project            RDS-CORE-SIHclassic-RW         240k           1T
/project            RDS-CORE-SIHsandbox-RW       236.7G           1T
Storage Usage (Filesystems totals) ---------------------------------
Filesystem Used     Free
/scratch   378.1Tb  1.5%
~~~
{: .output}

Neat!

<br>
By this time, our tiny test jobs should have run and competed. Check again

~~~
qstat -x 2556851
~~~
{: .language-bash}

~~~
[jdar4135@login3 hayim]$ qstat -x 2557008
Job id            Name             User              Time Use S Queue
----------------  ---------------- ----------------  -------- - -----
2557008.pbsserver Basic_hayim      jdar4135          00:00:00 F small
~~~
{: .output}

My job finished! Has yours? If you requested email notifications, did you get any?

### PBS log files

Now, list the contents of your working directory

~~~
ls -lsh
~~~
{: .language-bash}

~~~
[jdar4135@login3 hayim]$ ls -lsh
total 271M
-rw-r----- 1 jdar4135 RDS-CORE-Training-RW 116M Nov 30  2016 134_R1.fastq.gz
-rw-r----- 1 jdar4135 RDS-CORE-Training-RW 117M Nov 30  2016 134_R2.fastq.gz
-rw-r----- 1 jdar4135 RDS-CORE-Training-RW  748 Oct 25 11:48 align.pbs
-rw-r----- 1 jdar4135 RDS-CORE-Training-RW  203 Oct 25 14:34 basic.pbs
-rw-r----- 1 jdar4135 RDS-CORE-Training-RW  39M Nov 30  2016 canfam3_chr5.fasta
-rw------- 1 jdar4135 RDS-CORE-Training-RW    0 Oct 25 15:35 Basic_hayim.e2557008
-rw------- 1 jdar4135 RDS-CORE-Training-RW   31 Oct 25 15:35 Basic_hayim.o2557008
-rw-r--r-- 1 jdar4135 RDS-CORE-Training-RW 1.3K Oct 25 15:35 Basic_hayim.o2557008_usage
-rw-r----- 1 jdar4135 RDS-CORE-Training-RW  376 Aug 24 10:52 index.pbs
drwxr-sr-x 2 jdar4135 RDS-CORE-Training-RW 4.0K Oct 25 15:16 New_job
~~~
{: .output}

Notice anything new? There should be three new files, all beginning with your **job name**
~~~
-rw------- 1 jdar4135 RDS-CORE-Training-RW    0 Oct 25 15:35 Index_hayim.e2557008
-rw------- 1 jdar4135 RDS-CORE-Training-RW   31 Oct 25 15:35 Index_hayim.o2557008
-rw-r--r-- 1 jdar4135 RDS-CORE-Training-RW 1.3K Oct 25 15:35 Index_hayim.o2557008_usage
~~~
{: .output}

These are the **log files** for your job:

|:--|---|
| JobName.**e**JobID | **E**rror log: This is where error messages -- those usually printed to **stderr** -- are recorded |
| JobName.**o**JobID | **O**utput log: This is where output messages -- those usually printed to **stdout** -- are recorded |
| JobName.**o**JobID | **Usage** report: gives a short summary of the **resources** used by your job |

Check whether there were any errors in your job by inspecting the contents of the **error** log with ```cat```:

~~~
[jdar4135@login3 hayim]$ cat Index_hayim.e2557008
~~~
{: .output}

Empty! That's a good sign. Now let's have a look at the **output** log:

~~~
[jdar4135@login3 hayim]$ cat Index_hayim.o2557008
Hello, world! My name is Hayim.
~~~
{: .output}

The output from the **hello.pl** script appears in the PBS output log as expected.

Finally, let's have a look at the resource **usage** report:

~~~
[jdar4135@login3 hayim]$ cat Index_hayim.o2557008_usage
-- Job Summary -------------------------------------------------------
Job Id: 2557008.pbsserver for user jdar4135 in queue small
Job Name: Index_hayim
Project: RDS-CORE-Training-RW
Exit Status: 0
Job run as chunks (hpc016:ncpus=1:mem=1048576kb)
Walltime requested:   00:01:00 :      Walltime used:   00:00:03
                               :   walltime percent:       5.0%
-- Nodes Summary -----------------------------------------------------
-- node hpc016 summary
    Cpus requested:          1 :          Cpus Used:    unknown
          Cpu Time:    unknown :        Cpu percent:    unknown
     Mem requested:      1.0GB :           Mem used:    unknown
                               :        Mem percent:    unknown

-- WARNINGS ----------------------------------------------------------

** Low Walltime utilisation.  While this may be normal, it may help to check the
** following:
**   Did the job parameters specify more walltime than necessary? Requesting
**   lower walltime could help your job to start sooner.
**   Did your analysis complete as expected or did it crash before completing?
**   Did the application run more quickly than it should have? Is this analysis
**   the one you intended to run?
**
-- End of Job Summary ------------------------------------------------
~~~
{: .output}

What does the report show?

> ## Exit status
>
> Across *NIX systems and programming generally, an **Exit Status** code of **0** indicates that a program completed successfully.<sup id="a1">[1](#f1)</sup> Any exit code above 0 usually indicates an error.
>
> One way to check for errors in your jobs then, is to search for the words '_Exit Status_' in your log files
> ~~~
> grep -se "Exit Status" *
> ~~~
> {: .language-bash}
{: .callout}

### Did what we expected happen?

The final test of whether a job ran correctly is to check whether the outputs you were expecting to be produced actually were produced. In our case, the script **basic.pbs** was meant to create a new folder and copy itself into it. We have already seen that **hello.pl** was run successfuly, so check for the rest:

Our ```ls``` command earlier revealed that the new folder was successfully created:
~~~
drwxr-sr-x 2 jdar4135 RDS-CORE-Training-RW 4.0K Oct 25 15:16 New_job
~~~
{: .output}

so check inside:

~~~
[jdar4135@login3 hayim]$ ls -lh New_job/
total 512
-rw-r----- 1 jdar4135 RDS-CORE-Training-RW 203 Oct 25 15:35 copy.pbs
~~~
{: .language-bash}

Success.

<br>
## Practise makes perfect

Let's do that again.

### Index a genome

Open **index.pbs** and make any edits necessary. What will you need to change?

~~~
nano index.pbs
~~~
{: .language-bash}

<figure>
  <a name="nanoindex"></a>
  <img src="{{ page.root }}/fig/04_nanoindex.png" style="margin:10px;height:500px"/>
  <figcaption> The <b>index.pbs</b> PBS script. </figcaption>
</figure><br>


> ## Change #1
> Specify your **project**.
>
> Use the ```-P``` PBS directive to specify the _**Training**_ project, using its _short name_.
> ~~~
> #PBS -P Training
> ~~~
> {: .language-bash}
{: .solution}

> ## Change #2
> Give your job a **name**.
>
> Use the ```-N``` PBS directive to give your job an easily identifiable name. You might run **lots** of jobs at the same time, so you want to be able to keep track of them!
> ~~~
> #PBS -N Index_hayim
> ~~~
> {: .language-bash}
{: .solution}

> ## Change #3
> Tailor your **resource** requests.
>
> Use the ```-l``` PBS directive to request appropriate compute **resources** and **walltime** for your job.
>
> This script performs an 'indexing' operation against reference DNA sequence, but doesn't require more than a couple of minutes for the data we're using -- so request **2 minutes**.
>
> This small reference sequence (a tiny portion of 1 mammalian chromosome) also won't require much RAM, so reduce it to the minimum, **1 GB**.
> ~~~
> #PBS -l select=1:ncpus=1:mem=1GB
> #PBS -l walltime=00:02:00
> ~~~
> {: .language-bash}
{: .solution}

> ## Change #4
> Specify a **job queue**.
>
> Use the ```-q``` PBS directive to send the job to the **defaultQ** queue. You can also try **small-express** if you like; whose jobs start sooner?
> ~~~
> #PBS -q defaultQ
> ~~~
> {: .language-bash}
{: .solution}


> ## Change #5
> Set the working directory for this job. Either just ```cd``` to it, or set up a Bash variable to use within the script.
>
> Edit the line beginning ```io=```
> ~~~
> io=/project/Training/hayim
> ~~~
> {: .language-bash}
> Substitute your working directory name where I have put 'hayim'!
{: .solution}

When you're done, save the script (on nano <kbd>Ctrl</kbd>+<kbd>o</kbd>), and exit (on nano <kbd>Ctrl</kbd>+<kbd>x</kbd>).

Submit the job as before, with ```qsub```:

~~~
qsub index.pbs
~~~
{: .language-bash}

~~~
[jdar4135@login3 hayim]$ qsub index.pbs
2557043.pbsserver
~~~
{: .output}

This time I'm going to check on the job using ```qstat -wTu jdar4135``` to show my jobs, in wide format, with a start time estimate -- it's a little wide for this broswer window!

~~~
[jdar4135@login3 hayim]$ qstat -wTu jdar4135

pbsserver:
                                                                                  Est
                                                                                                   Req'd  Req'd   Start
Job ID                         Username        Queue           Jobname         SessID   NDS  TSK   Memory Time  S Time
------------------------------ --------------- --------------- --------------- -------- ---- ----- ------ ----- - -----
2557043.pbsserver              jdar4135        small-express   Index_hayim        49847    1     1    4gb 00:10 R --
~~~
{: .output}

<br>
When your job finishes (how will you know?) have a look at any log files produced. What is one way to check if there were any errors?

> ## Answer
> Search for Exit Status messages with ```grep```:
>
> ~~~
> grep -se "Exit Status" *
> ~~~
> {: .language-bash}
>
> ~~~
> [jdar4135@login3 hayim]$ grep -se "Exit Status" *
> Index_hayim.o2557008_usage:Exit Status: 0
> Index_hayim.o2557043_usage:Exit Status: 0
> ~~~
> {: .output}
> Both of my jobs have **Exit Status** 0 -- perfect.
{: .solution}

Have a look at your error file anyway. What do you notice?

> ## Answer
> Not every message to **stderr** is an actual error! Blame the program authors for this...
{: .solution}

Finally, have a look at the resource usage for this job

~~~
[jdar4135@login3 hayim]$ cat Index_hayim.o2557043_usage
-- Job Summary -------------------------------------------------------
Job Id: 2557043.pbsserver for user jdar4135 in queue small-express
Job Name: Index_hayim
Project: RDS-CORE-Training-RW
Exit Status: 0
Job run as chunks (hpc056:ncpus=1:mem=4194304kb)
Walltime requested:   00:02:00 :      Walltime used:   00:00:41
                               :   walltime percent:       34.2%
-- Nodes Summary -----------------------------------------------------
-- node hpc056 summary
    Cpus requested:          1 :          Cpus Used:    unknown
          Cpu Time:    unknown :        Cpu percent:    unknown
     Mem requested:      1.0GB :           Mem used:    unknown
                               :        Mem percent:    unknown

-- WARNINGS ----------------------------------------------------------

** Low Walltime utilisation.  While this may be normal, it may help to check the
** following:
**   Did the job parameters specify more walltime than necessary? Requesting
**   lower walltime could help your job to start sooner.
**   Did your analysis complete as expected or did it crash before completing?
**   Did the application run more quickly than it should have? Is this analysis
**   the one you intended to run?
**
-- End of Job Summary ------------------------------------------------
~~~
{: .output}

This job still did not run long enough for the system to properly estimate CPU or RAM usage.

<br>
### Align a genome

Last one. Open **align.pbs** and make any edits necessary. What will you need to change?

~~~
nano align.pbs
~~~
{: .language-bash}

<figure>
  <a name="nanoalign"></a>
  <img src="{{ page.root }}/fig/04_nanoalign.png" style="margin:10px;height:650px"/>
  <figcaption> The <b>align.pbs</b> PBS script. </figcaption>
</figure><br>

> ## Change #1
> Specify your **project**.
>
> Use the ```-P``` PBS directive to specify the _**Training**_ project, using its _short name_.
> ~~~
> #PBS -P Training
> ~~~
> {: .language-bash}
{: .solution}

> ## Change #2
> Give your job a **name**
>
> Use the ```-N``` PBS directive to give your job an easily identifiable name. You might run **lots** of jobs at the same time, so you want to be able to keep track of them!
> ~~~
> #PBS -N Align_hayim
> ~~~
> {: .language-bash}
> Substitute a job name of your choice!
{: .solution}

> ## Change #3
> Tailor your **resource** requests.
>
> Use the ```-l``` PBS directive to request appropriate compute **resources** and **wall-time** for your job.
>
> This script performs an 'alignment' of DNA reads against a reference genome. **10 minutes** should be plenty, and we won't use more than **1 GB** RAM.
>
> Notice here that **we are requesting 2 CPUs**. Are we sure our programs can use them?
> ~~~
> #PBS -l select=1:ncpus=2:mem=8GB
> #PBS -l walltime=00:10:00
> ~~~
> {: .language-bash}
{: .solution}

> ## Change #4
> Specify a **job queue**.
>
> Use the ```-q``` PBS directive to send the job to the **defaultQ** queue. You can also try **small-express** if you like; whose jobs start sooner?
> ~~~
> #PBS -q defaultQ
> ~~~
> {: .language-bash}
{: .solution}


> ## Change #5
> Set the working directory for this job. Either just ```cd``` to it, or set up a Bash variable to use within the script.
>
> Edit the line beginning ```io=```
> ~~~
> io=/project/Training/hayim
> ~~~
> {: .language-bash}
> Substitute your working directory name where I have put 'hayim'!
{: .solution}

When you're done, save the script (on nano <kbd>Ctrl</kbd>+<kbd>o</kbd>), and exit (on nano <kbd>Ctrl</kbd>+<kbd>x</kbd>).

Submit the job as before, with ```qsub```:

~~~
qsub align.pbs
~~~
{: .language-bash}

~~~
[jdar4135@login3 hayim]$ qsub align.pbs
2557080.pbsserver
~~~
{: .output}


<!-- Have a look at the _full output_ from ```qstat```

~~~
qstat -f 2557080
~~~
{: .language-bash}

There's a lot. Let's ```grep``` just a few lines:

~~~
qstat -f 2557080 | grep -e ""
~~~
{: .language-bash}

~~~

~~~
{: .output}

The line -->

<br>
We mentioned above that we requested 2 CPUs. How did we make use of them? Have a look at the _program call_ for ```bwa mem```<sup id="a2">[2](#f2)</sup>

~~~
bwa mem -M -t 2 -R '@RG\tID:134\tPL:illumina\tPU:CONNEACXX\tSM:MS_134' \
    ${io}/canfam3_chr5.fasta ${io}/134_R1.fastq.gz  \
    ${io}/134_R2.fastq.gz | samtools view -bSho ${io}/134.bam -
~~~
{: .language-bash}

What do those option flags do? Are any relevant here? Check the help or manual page for ```bwa```.

> ## Answer
> The ```bwa mem``` help instructions lists the ```-t``` flag as setting the number of _threads_. Multi-threading is a requirement for any program wanting to use more than one CPU core.
>
> On Artemis, the default is to run one thread per core, so selecting ```-t 2``` threads will lead Artemis to use **2 CPU cores**, if at least this many are requested.
{: .solution}

Has your **align.pbs** job completed yet? Check for errors:

~~~
grep -se "Exit Status" *
~~~
{: .language-bash}

~~~
[jdar4135@login3 hayim]$ grep -se "Exit Status" *
Align_hayim.o2557080_usage:Exit Status: 0
Index_hayim.o2557008_usage:Exit Status: 0
Index_hayim.o2557043_usage:Exit Status: 0
~~~
{: .output}

Great. However, note again that the **error log** is full of messages -- none though are actual errors. You could try searching for the word 'error' in the error log using ```grep``` as well, and see that it is empty -- however, we have no guarantee that the program authors would use the word 'error' in their error messages. Add the ```-i``` flag to make your search case _insensitive_:

~~~
[jdar4135@login2 hayim]$ grep -ie error Align_hayim.e2557080
~~~
{: .output}

As before, check the resource usage log:

~~~
cat Align_hayim.o2557080_usage
~~~
{: .language-bash}

~~~
[jdar4135@login2 hayim]$ cat Align_hayim.o2557080_usage
-- Job Summary -------------------------------------------------------
Job Id: 2557080.pbsserver for user jdar4135 in queue small
Job Name: Align_hayim
Project: RDS-CORE-Training-RW
Exit Status: 0
Job run as chunks (hpc018:ncpus=2:mem=3145728kb)
Walltime requested:   00:10:00 :      Walltime used:   00:04:34
                               :   walltime percent:      45.7%
-- Nodes Summary -----------------------------------------------------
-- node hpc018 summary
    Cpus requested:          2 :          Cpus Used:       1.54
          Cpu Time:   00:07:01 :        Cpu percent:      76.8%
     Mem requested:      3.0GB :           Mem used:      1.6GB
                               :        Mem percent:      54.6%

-- WARNINGS ----------------------------------------------------------

** Low Walltime utilisation.  While this may be normal, it may help to check the
** following:
**   Did the job parameters specify more walltime than necessary? Requesting
**   lower walltime could help your job to start sooner.
**   Did your analysis complete as expected or did it crash before completing?
**   Did the application run more quickly than it should have? Is this analysis
**   the one you intended to run?
**
** Low Memory utilisation on hpc018. While this may be normal, it may help to check
** the following:
**   Did the job parameters specify too much memory? Requesting less RAM could
**   help your job to start sooner.
**   Did you use MPI and was the work distributed correctly? Correcting this
**   could allow your job to run using less RAM in each chunk.
**   Did your analysis complete as expected or did it crash before completing?
**   Did the application use less RAM than it should have? Is this analysis the
**   one you intended to run?
**
-- End of Job Summary ------------------------------------------------
~~~
{: .output}


This is better resource utilisation: we used **45% of our walltime**, **77% CPU**, and **55% of our RAM**. We could trim our requests further, but walltime can vary for similar data depending on the algorithm, and providing too little memory (unlike too few CPUs) can cause a job to _fail_. So err on the side of caution.

<br>
#### Helping sick Doggos :dog2: :worried:

The sequence alignment you just created reveals the causal mutation for canine _spondylocostal dysostosis_ (genomic position
cfa5:32,945,846).

Neat, hey?

<figure>
  <a name="cali"></a>
  <img src="{{ page.root }}/fig/04_cali.png" style="margin:10px;height:650px"/>
  <figcaption> Willet et al 2015, PLoS ONE 10(2): e0117055a </figcaption>
</figure><br>

<br>

___
**Notes**

<sup id="f1">1[↩](#a1)</sup> It's important to keep in mind that 'success' to the operating system is not necessarily the same thing as 'success' in generating the results that you wanted from your job. Even with an exit status of zero, you should still check that your job produced the expected output.

<sup id="f2">2[↩](#a2)</sup> The ```\``` backslash in this block allow the code to be broken over multiple lines, allowing for neater code that doesn't train off the screen. There must be a space before the backslashes, and no space between the backslash and 'enter'.

___
<br>



{% include links.md %}
