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
......
System Status --------------------------------------------
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
......
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
## Practise makes perfect: Python 

Let's do that again. This time rather than set submit a script to the scheduler that prints a generic hello world message, we will run python code that performs computation. 

Investigate the ***computepi_fire.py*** python file. Given a number of trials to run, this code estimates the number pi by randomly assigning points within a square and comparing the number of points that fall within a circle of radius one relative to the total number of points. 

The script that submits this python code to the scheduler is the ***estimate_pi.pbs*** script.

~~~
nano estimate_pi.pbs
~~~
{: .output}

Things to notice in the script: 

a, A specific version of python is required. The code uses open source software called ***python fire*** that exposes python functions and classes to the command line in an easy manner. This dependency is installed with the ***pip*** command, a python specific package manager. Python fire requires a python 3X version.

b, The ***cd $PBS_O_WORKDIR*** command changes the directory to the location where the pbs script was submitted. This is an example of a shell variable that has been set by the pbs environment. More examples of environment variables are given at the end of the material.

c, The code requires an ***input*** parameter representing the number of random points used in the simulation. Increasing this will improve the accuracy at the expense of longer run time. Lets try!

Submit this script to the scheduler. Check the log files for errors and the output.
 
~~~
qsub estimate_pi.pbs
~~~
{: .output}

After this, try increasing the simulation number in the pbs script to one million, and practice deleting a job by using the ***qdel*** command with the job number that was printed used as an input argument. Here is an example:

~~~
[TRAINING kmar7637@login4 datahpc]$ qsub estimate_pi.pbs
463292.pbstraining
[TRAINING kmar7637@login4 datahpc]$ qdel 463292
qdel: Job has finished
~~~
{: .output}

Task: If you look again at the python code, there are optional arguments that could make your code even faster (apart from requesting more resources). Can you guess what they are and how to run it???


## Practise makes perfect: Matlab
Lets try one more job submission to reinforce the habit. This time we will submit a completely different matlab script that performs aggregation calculations on a csv file. 

If you are familiar with Matlab on your local machine, using matlab is a little different here on artemis. High performance computers generally work best when engaging software via linux commands. Applications that rely on graphical interfaces, while achievable with a nomachine interface or x11 forwarding described earlier, generally are slow and buggy. We will submit a pbs file that executes a matlab script in the advisable manner that suppresses graphical interaction.

Before we begin, lets investigate the underlying data with bash commands. Working with large data files is covered in other training material, its an art in itself, however, we'll get a feel for what we are working with with the ***head*** and ***wc*** commands. 

~~~
head airline2008.csv
wc -l airline2008.csv
~~~
{: .output}

The output reveals the csv file contains 2000 rows of flight information and identify columns that give arrival and departure time information.

The pbs script that runs matlab data computations on this csv file is the pbs scripted named ***airline.pbs***. A couple things to notice are:

a, The extra flags -nodisplay -nosplash suppress the matlab graphical interaction.

b, The command line way to run matlab with the -r (run) flag. There are a few ways to do it, consult matlab documentation for more examples. 
~~~
nano airline.pbs
~~~
{: .output}

~~~
qsub airline.pbs
~~~
{: .output}

If there are no errors, investigate the output file to find out what date in incurred the longest flight delays.

___
**Notes**

<sup id="f1">1[↩](#a1)</sup> It's important to keep in mind that 'success' to the operating system is not necessarily the same thing as 'success' in generating the results that you wanted from your job. Even with an exit status of zero, you should still check that your job produced the expected output.

<sup id="f2">2[↩](#a2)</sup> The ```\``` backslash in this block allow the code to be broken over multiple lines, allowing for neater code that doesn't train off the screen. There must be a space before the backslashes, and no space between the backslash and 'enter'.

___
<br>



{% include links.md %}
