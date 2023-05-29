---
title: "Example Job Submission"
teaching: 15
exercises: 35
questions:
- "How do I run a Python script?"
- "How do I run a Matlab program?"
- "How do I run an R script?"
objectives:
- "Learn how to run jobs with common scripting languages"
keypoints:
- "Use the pbs scripts in these examples as templates for your own work"
- "Most software is similar in how the commands are run"
- "Consider where extra packages and libraries of your environments will be stored on Artemis"
---

In this episode we practice with some real use cases! 

To catch up:
```
cd /project/Training
mkdir <yourname>
cd <yourname>
tar -xzvf /project/Training/DATA/intro_hpc.tar.gz
cd datahpc
```
Then you are good to go with the following examples.



# Practise makes perfect: Python 

Let's submit a job which runs python code that performs computation. 

Investigate the ***computepi_fire.py*** python file. Given a number of trials to run, this code estimates the number pi by randomly assigning points within a square and comparing the number of points that fall within a circle of radius one relative to the total number of points. 


The script that submits this python code to the scheduler is the ***estimate_pi.pbs*** script.

~~~
nano estimate_pi.pbs
~~~
{: .language-bash}

Things to notice in the script: 

a. A specific version of python is required. The code uses open source software called ***python fire*** that exposes python functions and classes to the command line in an easy manner. This dependency is installed with the ***pip*** command, a python specific package manager. Python fire requires a python 3X version.

b. The ***cd $PBS_O_WORKDIR*** command changes the directory to the location where the pbs script was submitted. This is an example of a shell variable that has been set by the pbs environment. More examples of environment variables are given at the end of the material.

c. The code requires an ***input*** parameter representing the number of random points used in the simulation. Increasing this will improve the accuracy at the expense of longer run time. Lets try!

Submit this script to the scheduler. Check the log files for errors and the output.
 
~~~
qsub estimate_pi.pbs
~~~
{: .language-bash}

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

If you are familiar with Matlab on your local machine, using Matlab is a little different here on Artemis. High performance computers generally work best when engaging software via Linux commands. Applications that rely on graphical interfaces (while achievable with a Nomachine interface or X11 forwarding described earlier) generally are slow and buggy. We will submit a pbs script that executes a Matlab script in the advisable manner that suppresses graphical interaction.

Before we begin, lets investigate the underlying data with bash commands. Working with large data files is covered in other training material - it's an art in itself - however, we'll get a feel for what we are working with with the `head` and `wc` commands. 

~~~
head airline2008.csv
wc -l airline2008.csv
~~~
{: .language-bash}

The output reveals the csv file contains 1,754 rows of flight information containing columns that give arrival and departure time information.

The pbs script ***airline.pbs*** runs Matlab data computations on this csv file. A couple of things to notice are:

a. The extra flags `-nodisplay -nosplash` suppress the matlab graphical interaction.

b. The command line way to run Matlab with the `-r` (run) flag. There are a few ways to do it - consult Matlab documentation for more examples. 
~~~
nano airline.pbs
~~~
{: .language-bash}

~~~
qsub airline.pbs
~~~
{: .language-bash}

If there are no errors, investigate the output file to find out what date incurred the longest flight delays.


# Practise makes perfect: R
Running R code on Artemis requires installing and referencing libraries needed for your R script. The recommended way to do this is to use the ***renv*** package to replicate libraries and specific versions created locally via a "lockfile". The lockfile is created by renv. 


Three example scripts to demostrate using R (with an established renv lock file) in this course are:

1. ```install_env.pbs``` : Houses the initial setup of installing R libraries to a path you have read / write permissions to. Here ```/home``` is used, but in practice it should reside somewhere in your ```/project``` directory (for storage considerations). 
2. ```run_network.pbs``` : Main pbs script that references a path where libraries exist (created by the above file).  
3. ```network.R``` : main R script that does data manipulation and visualisation via common R libraries including dplyr, plotly and network libraries. Generates a network plot based on flight information. 

The installation has already been run and libraries saved in the "/project/Training/DATA/Libs" folder. ***Please chage the library path in run_network.pbs to this folder and run:*** 
~~~
qsub run_network.pbs
~~~
{: .language-bash}
 
___
<br>



{% include links.md %}
