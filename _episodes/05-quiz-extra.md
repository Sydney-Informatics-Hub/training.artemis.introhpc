---
title: "Quiz! and Additional Notes"
teaching: 10
exercises: 10
questions:
- "Let's see how much we've learned!"
objectives:
- "Enhance your use of Artemis HPC"
keypoints:
- "PBS **environment variables** can make your life easier"
- "You can customise your log files"
- "Artemis is _**not**_ backed up!"
---

## Quiz!

Read through **all the questions** on the first slide **_BEFORE CLICKING_** to advance slide!!

Better yet, _wait for the instructor_ and we'll go through the quiz all together. :blush:

<br>
<iframe src="https://unisyd-my.sharepoint.com/:p:/g/personal/hayim_dar_sydney_edu_au/ESMUb4t3T8xNtvUT01KmkkcB1ER1VNimkf7aOl6mq4asQg?e=prI08i&amp;action=embedview&amp;wdAr=1.7777777777777777" width="730px" height="570px" frameborder="0">This is an embedded <a target="_blank" href="https://office.com">Microsoft Office</a> presentation, powered by <a target="_blank" href="https://office.com/webapps">Office Online</a>.</iframe>
<br>

## Additional notes

### PBS Environment Variables

Shell _'environment variables'_ are variables automatically set for you, or specified in a shell's configuration. When you invoke a PBS shell environment, with a ```qsub``` command eg, certain variables are set for you. Some of the more useful are listed below

| PBS variable | Meaning | Use |
|:---:|:----|:---|
| **PBS_O_WORKDIR** | Is set to the _current working directory_ from which you ran ```qsub``` | ```cd $PBS_O_WORKDIR``` at the beginning of your script will change PBS into the current directory, allowing access to any data stored there |
| **NCPUS** | The number of CPUs requested via ```-l select=``` | To ensure you tell any programs the correct number of CPUs they have access to, pass ```$NCPUS``` instead of a number as the argument |
 **PBS_JOBID** | The JobID assigned to the job | Use ```$PBS_JOBID``` give a unique name to any output or log files generated |

<br>
How might we have used **NCPUS** in the last job we submitted?

> ## Answer
> Use the ```$NCPUS``` variable to specify the number of threads we want the ```bwa mem``` program to use
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

To redirect both the output (**1**) and error (**2**) streams

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

An **Exit Status** of 0 generally indicates a successfully completed job. Exit statuses up to **128** are the statuses returned by the program itself that was running and failed. Here are a few other common codes to watch out for when your job doesn't run as expected and you want to know why

|Code|Meaning|
|:--:|---|
|3| The job was killed before it could be run |
|137| The job was killed because it _ran out_ of **RAM** |
|271| The job was killed because it _ran out_ of **walltime** |

<br>
### Artemis is NOT backed up!

Artemis is **not** intended to be used as a data store. Artemis is not backed up, and has limited space. Any data you have finished working with should be transferred to your **_RCOS_** space.

How to do this is covered in the next course, [‘_Data transfer and RDS for HPC_’]({{ site.sih_pages }}/training.artemis.rds)!

<figure>
  <img src="{{ page.root }}/fig/05_backup.png" style="margin:10px;height:300px"/>
  <figcaption> Backup your data. </figcaption>
</figure><br>



{% include links.md %}
