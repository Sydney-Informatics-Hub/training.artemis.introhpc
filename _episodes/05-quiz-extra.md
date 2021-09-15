---
title: "Quiz! and Case Study"
teaching: 10
exercises: 10
questions:
- "Let's see how much we've learned!"
objectives:
- "Enhance your use of Artemis HPC"
keypoints:
- "PBS **environment variables** can make your life easier"
- "You can customise your log files"
- "link commands together in scripts to form a computation pipeline"
- "Artemis is _**not**_ backed up!"
---

## Quiz!

Have a go at the questions below **_BEFORE_** revealing the answer!!

Better yet, _wait for the instructor_ and we'll go through the quiz all together. :blush:

1. How do you submit a job called align.pbs?

> ## 1. How do you submit a job called align.pbs?
> ~~~
> qsub align.pbs
> ~~~
> {: .language-bash}
{: .solution}

2. How do you check the status of the job?
 
> ## Answer
> ~~~
> qstat <JOBID>
> qstat -x -u <UNIKEY>
> ~~~
> {: .language-bash}
{: .solution}

3. How might you know when the job is finished?
 
> ## Answer
> The output of ‘qstat’ shows an ‘F’ in the ‘S’ (status) column.
> We receive an email from the system indicating termination. 
> We can see the log files for this job (they are only created at completion).
{: .solution}

4. How much space are you allocated in project, and how do you check how much you have used?
 
> ## Answer
> Space per project: 1 TB, command ‘pquota’
{: .solution}

5. What log files are produced after a job completes, and what do they contain?

> ## Answer
> ```JobName.oJobID```–standard output (things that are usually printed to terminal)
>
> ```JobName.eJobID```–standard error (OS or application error messages)
>
> ```JobName.oJobID_usage```–job resource usage
{: .solution}

6. What do the ```-P```,```-N```, and ```-q``` directives indicate to the scheduler?

> ## Answer 
> ~~~
> #PBS -P yourproject # Specify your project.
> #PBS -N yourjobname #Specify the name of you job.
> #PBS -q defaultQ #Request which queue to run your job on.
> ~~~
> {: .language-bash}
{: .solution}

7. Write the directive requesting 10.5 hours wall time

> ## Answer
> ~~~
> #PBS –l walltime=10:30:00
> ~~~
> {: .language-bash}
{: .solution}

8. Write the directive to specify a 6-core job using 2 GB RAM per core

> ## Answer
> ~~~
> #PBS –l select=1:ncpus=6:mem=12GB
> ~~~
> {: .language-bash}
> or
> ~~~
> #PBS –l select=6:ncpus=1:mem=2GB
> ~~~
> {: .language-bash}
{: .solution}

9. Your job uses the software ‘beast’. What needs to come before your command to run beast?

> ## Answer 
> ~~~
> module load beast
> ~~~
> {: .language-bash}
{: .solution}

10. Where should all important input and output data be stored long-term and why?

> ## Answer
> The Research Data Store (RDS). RDS is backed up and Artemis is not!
{: .solution}

## Additional notes

### PBS Environment Variables

Shell _'environment variables'_ are variables automatically set for you, or specified in a shell's configuration. When you invoke a PBS shell environment, for example with a ```qsub``` command, certain variables are set for you. Some of the more useful are listed below:

| PBS variable | Meaning | Use |
|:---:|:----|:---|
| **PBS_O_WORKDIR** | Is set to the _current working directory_ from which you ran ```qsub``` | ```cd $PBS_O_WORKDIR``` at the beginning of your script will change PBS into the current directory, allowing access to any data stored there |
| **NCPUS** | The number of CPUs requested via ```-l select=``` | To ensure you tell any programs the correct number of CPUs they have access to, pass ```$NCPUS``` instead of a number as the argument |
 **PBS_JOBID** | The JobID assigned to the job | Use ```$PBS_JOBID``` to give a unique name to any output or log files generated |

<br>
How might we have used **NCPUS** in the a command within our pbs jobscript?

> ## Answer
> Use the ```$NCPUS``` variable to specify the number of threads we want, e.g. in the ```bwa mem``` program you can use:
> ~~~
> bwa mem -M -t $NCPUS -R '@RG\tID:134\tPL:illumina\tPU:CONNEACXX\tSM:MS_134'
> ~~~
> {: .language-bash}
{: .solution}

<br>
### qdel

If you find you have submitted a job incorrectly, or simply wish to cancel it for whatever reason, you can use ```qdel``` to remove it from the **queue**, or remove its historical record with ```-x``` if it is finished.
~~~
qdel [-x] JobID [JobID ...]
~~~
{: .language-bash}

More than one JobID may be supplied, separated with spaces.

<br>
### Log files

The log files we have seen so far use Artemis' default options and naming conventions. You can specify your own log files and names as follows

~~~
#PBS -o OutputFilename
#PBS -e InputFilename
~~~
{: .language-bash}

You can also combine both log files into one, using the **join** directive; **o**e combines both into the output log file, and **e**o combines them in the error file, using the default names unless you specify otherwise.
~~~
#PBS -j oe
~~~
{: .language-bash}
~~~
#PBS -j eo
~~~
{: .language-bash}

PBS log files are also only created when your job _completes_. If you want to be able monitor the progress of a program which outputs such information to **stdout** or **stderr**, you can **_pipe_** these outputs to a file of your choosing, with ```>```. Eg:
~~~
# Program commands
myProg -flag option1 inputFile > myLogFile.txt
~~~
{: .language-bash}

To redirect both the output (**1**) and error (**2**) streams:

~~~
# Program commands
myProg -flag option1 inputFile 1> myLogFile_$PBS_JOBID.txt 2> myErrorFile_$PBS_JOBID.txt
~~~
{: .language-bash}

If you redirect a message stream via piping to a file, it will no longer be available to PBS, and so the PBS log for that stream will be empty.

By default, your log files carry a [**_umask_** ](https://en.wikipedia.org/wiki/Umask) of **077**, meaning that no-one else has any permissions to write, execute or even read your logs. If you want other people in your project to be able to read your log files (eg for debugging), then set the _umask_ to **027**; if you want everyone to be able to read your log files, set the _umask_ to **022**. This is done via the **additional_attributes** directive
~~~
#PBS -W umask=022
~~~
{: .language-bash}

<br>
### Common error exit codes

An **Exit Status** of 0 generally indicates a successfully completed job. Exit statuses up to **128** are the statuses returned by the program itself that was running and failed. Here are a few other common codes to watch out for when your job doesn't run as expected and you want to know why:

|Code|Meaning|
|:--:|---|
|3| The job was killed before it could be run |
|137| The job was killed because it _ran out_ of **RAM** |
|271| The job was killed because it _ran out_ of **walltime** |

<br>
### Still need help?
If your solution is not in these notes or on the support pages [https://sydneyuni.atlassian.net/wiki/spaces/RC/overview](https://sydneyuni.atlassian.net/wiki/spaces/RC/overview)

Then log a ticket!

Connect to Sydney University's Service Managment Portal: [https://sydneyuni.service-now.com/sm](https://sydneyuni.service-now.com/sm)

Click on **"Submit a Request"**

Navigate to the **"High Performance Computing Request"**

**Home >  Technology > Research services > HPC - High Performance Computing request**

These requests go straight to the Artemis Service Management team. You can request help installing software, compiling code, and anything else. Also, if you think something weird is happening with Artemis log a ticket (under "High Performance Computing issue") - these fault reports will help us diagnose what is happening and improve the Artemis service!

<br>
### Artemis is NOT backed up!

Artemis is **not** intended to be used as a data store. Artemis is not backed up, and has limited space. Any data you have finished working with should be transferred to your **_RCOS_** space.

How to do this is covered in the next course, [‘_Data transfer and RDS for HPC_’](https://informatics.sydney.edu.au/training/coursedocs/Introduction_to_RDS.pdf)

<!-- Awaiting the day when data transfer course is on github ({{ site.sih_pages }}/training.artemis.rds)! -->

<figure>
  <img src="{{ page.root }}/fig/05_backup.png" style="margin:10px;height:300px"/>
  <figcaption> Back up your data. </figcaption>
</figure><br>


> ## Course survey!
>
> **_Please_** fill out our **[course survey](https://redcap.sydney.edu.au/surveys/?s=FJ33MYNCRR&training=12&training_date=2021-12-21)** before you leave!
>
> Help us help you! :smiley:
{: .testimonial}

{% include links.md %}
