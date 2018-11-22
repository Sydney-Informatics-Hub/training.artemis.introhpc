---
title: "Welcome to Artemis HPC"
teaching: 20
exercises: 0
questions:
- "Who are the Sydney Informatics Hub?"
- "What is an 'Artemis'..?"
objectives:
- "Learn how to connect to Artemis."
keypoints:
- "Connecting to Artemis requires a terminal emulator, and an account with access."
- "Users connect to Artemis' _**login nodes**_ only."
- "On Windows, use X-Win32, PuTTY, or another shell and terminal application of your choice."
- "GUI login access is also available."
---
This episode introduces the [Sydney Informatics Hub](https://informatics.sydney.edu.au/), Artemis HPC and how to get connected.


# The Sydney Informatics Hub

The Sydney Informatics Hub (SIH) is a _[Core Research Facility](https://sydney.edu.au/research/facilities.html)_ of the University of Sydney. Core Research Facilities centralise essential research equipment and services that would otherwise be too expensive or impractical for individual Faculties to purchase and maintain. The classic example might be the room-size electron-microscopes, built into specialised rooms in the Sydney Microscopy & Microanalysis unit.

<figure>
  <img src="{{ page.root }}/fig/01_crf.png" style="margin:10px;height:400px"/>
  <figcaption> USyd Core Research Facilities <a href="https://sydney.edu.au/research/facilities.html">https://sydney.edu.au/research/facilities.html</a></figcaption>
</figure><br>

**Artemis HPC** itself is a multi-million dollar set of equipment, a 'supercomputer', is the main piece of equipment supported by SIH. However, we also provide a wide range of research services to aid investigators, such as

* [Training](https://informatics.sydney.edu.au/training/)
* [Statistics Consulting](https://informatics.sydney.edu.au/services/statistics/)
* Modeling/Simulation/Visualisation consulting and [platforms](https://informatics.sydney.edu.au/services/artemis/)
* [Bioinformatics](https://informatics.sydney.edu.au/services/bioinformatics/) consulting
* [Research Data Management](https://informatics.sydney.edu.au/rdm/) Consulting and platforms
* [Data Science & Research Engineering](https://informatics.sydney.edu.au/services/data-science/) project work

We also aim to cultivate a **data community** at USyd, organising monthly [Hacky Hours](https://informatics.sydney.edu.au/hackyhour/), outside training events (eg NVIDIA, Pawsey Center), [conferences](https://informatics.sydney.edu.au/hpc_conference/), and data/coding-related events. We are currently running an quarterly [Publication Incentive](https://informatics.sydney.edu.au/news/sihincentive/) contest, with $2000 worth of prizes for the winning peer-reviewed publications submitted to us, which both use our services _and_ acknowledge them.

# Artemis HPC

We've mentioned 'Artemis HPC' many times now, but what is it? HPC stands for 'High Performance Computing', but you might also simply call Artemis a 'supercomputer'. Technically, Artemis is a _computing cluster_, which is a whole lot of individual computers networked together. At present, Artemis consists of:

* 7,636 cores (CPUs)
* 45 TB of RAM
* 108 NVIDIA V100 GPUs
* 378 TB of storage
* 56 Gbps FDR InfiniBand (networking)

Artemis computers (which we'll call _machines_ or _nodes_) run a **Linux** operating system, 'CentOS' v6.9. Computing performed Artemis' nodes is managed by a **_scheduler_**, and ours is an instance of 'PBS Pro'.

## Why use Artemis?

Artemis is ideal for calculations that require
* A long time to complete (long _walltime_)
* High RAM usage
* Big data input or outputs, or
* Are able to use multiple cores or nodes to run in parallel, and hence much faster

Artemis is **available free of charge to all** University researchers. You do need a unikey, and a valid RDMP (_Research Data Management Plan_) with Artemis access enabled.

Artemis is also a great **incentive to funding bodies** to view your projects favourably -- as they know you have the resources required to get the work done.

Finally, if you do use Artemis for your research, please acknowledge us! This ensures that we continue to get the funding we need to provide you with what is really a first-grade computing resource. And don't forget to apply to the Publication Incentive! A suggested acknowledgment might say:

> _The authors acknowledge the Sydney Informatics Hub and the University of Sydney’s high performance  computing cluster, Artemis, for providing the computing resources that have contributed to the results reported herein._

# Connecting to Artemis

Connections to Artemis are **remote connections** -- you'll never sit at one of Artemis' machines, which are stored in a secure data-centre in Western Sydney. Instead, you connect remotely into one of Artemis' **login nodes**. Login nodes are Artemis machines that don't perform any actual computing jobs, but simply provide users with an access gateway to Artemis' filesystems and the PBS Pro **job scheduler**.

You can thus connect to Artemis from _anywhere_, requiring only a **terminal emulator** with an **SSH client**. (If you're not on the USyd network (ie off-campus), you'll also need to connect to the University's **[VPN](http://staff.ask.sydney.edu.au/app/answers/detail/a_id/519/kw/vpn)**, or use Artemis' intermediate **_Jump server_**).

If you followed the [Setup]({{ page.root }}/setup) instructions, then you should already have the required software installed. If not, _please go do this now_!


<h2 data-toc-text="via SSH command line"> Connecting via SSH in a terminal (recommended)</h2>

Depending on your computer's operating system, there may be several ways to connect to Artemis. The simplest way is to open your **terminal emulator** application, and 'ssh' into the Artemis login-servers. This is our recommended method, as to use Artemis effectively you should get comfortable working on the **command line**.

Linux and Mac both have native terminal apps, so you only need to open them. You may also have installed one on your Windows machine.<sup id="a1">[1](#f1)</sup> Go ahead and do that now. The last line displayed in your terminal window should have some information about your computer's name, and you user name, followed by a **$** symbol. This is the **command prompt** -- you type your commands after the '$'.

<figure>
  <img src="{{ page.root }}/fig/01_bash.png" style="margin:10px;height:400px"/>
  <figcaption> An iTerm2 terminal window on Mac</figcaption>
</figure><br>

To connect to Artemis securely, we'll use the **SSH** (Secure Socket Shell) protocol; on most systems, any installed SSH client will be invoked by the command 'ssh'. Before you connect, make sure you know your **username** and **password**. When you use Artemis for your research, these will be your **Unikey** and **Unikey password**; however, for this training course we'll be using _training accounts_, which are:

* Username: **ict_hpctrain\<N\>**, with N from 1-20 (replace _**\<N\>**_ with your assigned number)
* Password: _will be written on the whiteboard!_

At your command prompt, execute the following (type it and press 'return/enter'):

~~~
ssh -X ict_hpctrain<N>@hpc.sydney.edu.au
~~~
{: .language-bash}

or, if using XQuartz on a Mac

~~~
ssh -Y ict_hpctrain<N>@hpc.sydney.edu.au
~~~
{: .language-bash}

The ```-X``` or ```-Y``` flags tell **ssh** to enable X-forwarding, which lets GUI programs on Artemis serve you graphical windows back on your local machine.

If connecting for the first time, you may get the following output, requesting authorisation to connect to a new **host** server:

~~~
The authenticity of host 'hpc.sydney.edu.au (10.250.96.203)' can't be established.
RSA key fingerprint is SHA256:qq9FPWBcyvvOWOMdFs8uZES0tF3SVzJsNx1cdn56GSE.
Are you sure you want to continue connecting (yes/no)?
~~~
{: .output}

Enter 'yes'. You will then be asked for your password; type it and press 'enter'. and you should then be logged in!

<figure>
  <img src="{{ page.root }}/fig/01_granted.png" style="margin:10px;height:420px"/>
  <figcaption> Access granted! </figcaption>
</figure><br>


<h2 data-toc-text="via SSH GUI apps"> Connecting via an SSH GUI (common for Windows users) </h2>

If you're on Windows, and followed the [Setup]({{ page.root }}/setup) guide, then you will likely be connecting through an X-window or shell client program, like 'X-Win32' or 'PuTTY'. Following the instructions in the [Setup]({{ page.root }}/setup) guide
* Open your installed program
* Select the "Artemis" session you configured earlier
* Click 'Launch' (X-Win32) or 'Open' (PuTTY)

If this is the first time connecting to Artemis, you will be asked to authorise it as a trusted **host** server; click 'Accept' (X-Win32) or 'Yes' (PuTTY).

<figure>
  <img src="{{ page.root }}/fig/01_xwinhosts.png" style="margin:10px;height:240px"/>
  <img src="{{ page.root }}/fig/01_puttyhosts.png" style="margin:10px;height:250px"/>
  <figcaption> Unknown host challenges: X-Win32 (top), PuTTY (bottom) </figcaption>
</figure><br>

* If using 'X-Win32', you'll then be asked for your **password** and once entered, you should be logged on to Artemis! A terminal window and command prompt on Artemis will appear.

* If using 'PuTTY', a terminal window will appear and prompt you for your **username**, and then your **password**. Once entered, you should be logged on to Artemis! A command prompt on Artemis will appear in that window.

<figure>
  <img src="{{ page.root }}/fig/01_xwin.png" style="margin:10px;height:220px"/>
  <img src="{{ page.root }}/fig/01_putty.png" style="margin:10px;height:350px"/>
  <figcaption> Access granted! X-Win32 (top) cuts the welcome messages, PuTTY (bottom) </figcaption>
</figure><br>


<h2 data-toc-text="via graphical login nodes"> Connecting via an the Graphical Login Nodes (advanced users)</h2>

For some users, it is occasionally necessary to have more reliable graphical access to the Artemis **login nodes**, in order to check intermediate results when using software with graphical outputs. Setup instructions are provided on the [Setup]({{ page.root }}/setup) page.


<br>   

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup> Such as 'Cygwin', 'MinGW', or even the _very handy_ ['Git for Windows'](https://gitforwindows.org/).

___
<br>



{% include links.md %}
