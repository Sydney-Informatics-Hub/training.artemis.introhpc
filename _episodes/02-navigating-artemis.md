---
title: "Navigating Artemis"
teaching: 20
exercises: 0
questions:
- "How is Artemis HPC organised?"
objectives:
- "Learn how to navigate Artemis' filesystem"
- "Find out what software is available on Artemis"
- "Learn how to write script files"
keypoints:
- "Storage on Artemis is arranged by _project_, which corresponds to a single RDMP."
- "There are 3 key branches of the Artemis filesystem: **/project**, **/scratch** and **/home**"
- "Software is managed by the 'Environment Modules' package, and must be loaded before use."
---
This episode gets users familiar with the Artemis servers, available software and directory structure. It also introduces users to text editors on Artemis for scripting.

# The Artemis directory structure

<br>
## When you first arrive

At the end of last lesson we logged on to Artemis, and were presented with a command prompt in a terminal window. The last line in that window would look something like this:

<figure>
  <img src="{{ page.root }}/fig/02_landing.png" style="margin:10px;height:150px"/>
  <figcaption> Command prompt on Artemis, using X-Win32 </figcaption>
</figure><br>


The **command prompt**, which is where in the terminal you type your commands, ends with a **$** dollar sign. Before that is some text, in the form of ```[username @ host current_directory ]```. In the screencap above, the username is **ict_hpctrain1**, the host server that we have logged into is **'login3'** (the 3rd Artemis login node), and our current cirectory is **~**, which is a shortcode for the user's 'home' directory.

Even though no files are visible or listed in the terminal window, the command prompt is always 'located' at a specific place in the filesystem -- we can print that location to the window with the ```pwd``` (_print working directory_) command. Type it and press 'enter':

~~~
[ict_hpctrain1@login3 ~]$ pwd
~~~
{: .language-bash}

The output should look something like this (if your username is **ict_hpctrain1**):

~~~
/home/ict_hpctrain1
~~~
{: .output}

This is your **home** directory. Every user on Artemis (and any Linux system) has a home directory located at ```/home/<username>```. Your username may be your Unikey, or it may be the training accounts we're using today. Every time you log in to Artemis, this is where you'll end up: home. :blush:

<br>
> ## Directions for the student
>
> In these courses, **<font color="#6e5494">purple</font>** coloured block-quotes as above indicate commands that you need to _enter_ into your command prompt.
>
> **Black** coloured block-quotes indicate _output_ you should expect as a result.)
{: .callout}

<br>
<h2 data-toc-text="Artemis' 3 main directories">The three branches of the Artemis tree</h2>

<figure>
  <img src="{{ page.root }}/fig/02_arttree.png" style="margin:10px"/>
  <figcaption> Artemis filesystem structure </figcaption>
</figure><br>

The Artemis filesystem has three main branches: **/home**, **/project**, and **/scratch**. We have just met **/home**. The **root** folder of the filesystem is signified by a single forward-slash **/**; and so every other location, which is a subfolder of the root, must also begin with this slash. This it why 'home' is located at ```/home```. (If this is all new to you, then you probably haven't taken the [Intro to Unix](https://intersect.org.au/training/course/unix/) prerequisite course. You should do that ASAP).

### i. /home

**/home** is where your data is. But not your _project_ data, just your own personal data. If you have any project-related data that needs to remain _private_ to you alone then also keep that in **/home**, as only you can see or read files in your home directory. Home directories are allocated **10 GB** of storage only.

### iii. /project

The **/project** branch is where researchers on a project should keep all the data and programs or scripts that are _currently_ being used. We say 'currently' because project directories are allocated only **1 TB** of storage space -- that may sound like a lot, but this space is shared between all users of your project, and it runs out faster than you think. Data that you're not currently working on should be deleted or moved back to its permanent storage location (which should be the Research Data Store<sup id="a1">[1](#f1)</sup>).

This also means that everyone working on your project can see all files in the project directory by default.<sup id="a2">[2](#f2)</sup>

Project directories are all subfolders of **/project**, and will have names like

~~~
/project/RDS-CORE-Training-RW
~~~
{: .output}

which take the form ```/RDS-<Faculty>-<Short_name>-RW```. In the case above, 'Training' is the project name and 'CORE' stands for Core Research Facilities.

If you forget what your projects are called, you can always check to see which **user groups** (which include projects) you are a member of, using the ```groups``` command:

~~~
[ict_hpctrain1@login3 ~]$ groups
~~~
{: .language-bash}

~~~
[ict_hpctrain1@login1 ~]$ groups
linuxusers HPC-ACL RDS-ICT-HPCTRAINING-RW RDN-USERS RDN-CORE-Training RDS-CORE-Training-RW RDN-CORE-SIHsandbox RDS-CORE-SIHsandbox-RW
~~~
{: .output}

Note that this user ```ict_hpctrain1``` is a member of the ```RDS-ICT-HPCTRAINING-RW``` and ```RDS-CORE-Training-RW``` and ```RDS-CORE-SIHsandbox-RW``` projects.

The middle part of the **project name** is also called the _**short name**_, and was chosen by whomever applied for the **RDMP** (Research Data Management Plan) that defines the project. The full folder name ```/RDS-<Faculty>-<Short_name>-RW``` is technically the actual project name as known to Artemis, however the _short names_ can be used throughout Artemis instead.

So, for example, change into the project directory of the 'Training' project with the _change directory_ command ```cd```:

~~~
[ict_hpctrain1@login3 ~]$ cd /project/Training
[ict_hpctrain1@login1 Training]$ pwd
~~~
{: .language-bash}

~~~
/project/Training
~~~
{: .output}


### iii. /scratch

Every project also has a **/scratch** directory, at ```/scratch/<Short_name>```. **/scratch** is where you should actually _perform_ your computations. What does this mean? If your workflow:
* Has big data inputs
* Generates big data outputs
* Generates lots of intermediate data files which are not needed afterward

then you should put and save this data in your **/scratch** space. The reasons are that you are not limited to **1 TB** of space in **/scratch** as you are in **/project**, and there is also no reason to clutter up your shared project directories with temporary files. Once your computation is complete, simply copy the important or re-used inputs and outputs back to your **/project** folder, and delete your data in **/scratch** -- that's why it's called scratch!

<br>
> ## Your data should not be kept on Artemis!
>
> Wait, what? Your data, _long-term_, should _not_ be stored on Artemis, neither in **/project** nor **/scratch**. Artemis is _not_ backed-up, and has _limited space_.
> <hr>
> <p style="text-align:center;font-weight:bold;text-decoration:underline">Inactive data in /project is wiped after 6 months, and in /scratch after 3 months!</p>
> <hr>
> Data that you are _currently_ working with should be transferred into your **/project** folder (or **/scratch** if it is very large!), and then transferred back to it's permanent location when you're done with that part of your investigation. For more, see the [Data transfer and RDS for HPC]({{ site.sih_pages }}/training.artemis.rds) course.
>
> <p style="text-align:center"><img src="{{ page.root }}/fig/02_data_flow.png" style="margin:10px;height:100px"/></p>
>
{: .callout}
<br>

#### Checking your disk usage

Artemis provides a handy tool to quickly see how much disk space is currently being used by all of the projects you are a member of, and your personal **/home** directory. That tool is ```pquota```:

~~~
ict_hpctrain1@login3 ~]$ pquota
~~~
{: .language-bash}

~~~
Disk quotas for user jdar4135 (uid 572557):
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
          /home  4.973G     10G     11G       -    1505       0       0       -
Disk quotas for group RDS-CORE-SIHclassic-RW (gid 16700):
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
       /project    240k      1T  1.074T       -      47       0       0       -
Disk quotas for group RDS-CORE-CLC-RW (gid 22099):
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
       /project      4k      1T  1.074T       -       1       0       0       -
Disk quotas for group RDS-CORE-SIHsandbox-RW (gid 16198):
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
       /project  233.9G      1T  1.074T       -   81018       0       0       -
Disk quotas for group RDS-CORE-Training-RW (gid 14206):
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
       /project  31.89G      1T  1.074T       -    1746       0       0       -
Disk quotas for group RDS-CORE-ICT-RW (gid 15839):
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
       /project  514.3M      1T  1.074T       -      17       0       0       -
~~~
{: .output}

The output from ```pquota``` helpfully lists the disk quotas allowed for each of the directories you can write to, as well as the number of files you have in that filesystem. There is no limit on the number of files allowed. The usages in **/project** comprise the files of all members of that project, not just your own. **/scratch** usage is not reported.

To see the disk usage of a project's **/scratch** directory (or any directory), you can use the Unix 'used disk space' command ```du```:

~~~
du -sh /scratch/Training/
~~~
{: .language-bash}

~~~
244M	/scratch/Training/
~~~
{: .output}

Or, if you'd like to know how much space _members of a project_ have used in a directory, you can query the filesystem using its built-in utility ```lfs```:

~~~
lfs quota -hg RDS-CORE-Training-RW /scratch
~~~
{: .language-bash}

~~~
Disk quotas for group RDS-CORE-Training-RW (gid 14206):
     Filesystem    used   quota   limit   grace   files   quota   limit   grace
       /scratch  243.5M      0k      0k       -       2       0       0       -
~~~
{: .output}

The amount of free **/scratch** space can be reported via the 'free disk space' command ```df```:

~~~
df -h /scratch
~~~
{: .language-bash}

~~~
Filesystem            Size  Used Avail Use% Mounted on
192.168.69.211@o2ib0:192.168.69.212@o2ib0:/Scratch
                      379T  350T  9.3T  98% /scratch
~~~
{: .output}

Note that in the above commands the flag ```-h``` requests 'human-readable' file sizes (in B, M, G, T, etc), and the flag ```-s``` summarised the ```du``` output into one total count, rather than a number for every sub-directory.

<br>
## Software on Artemis

### Existing software

A [list](https://sydneyuni.atlassian.net/wiki/spaces/RC/pages/285474817/Artemis+Software+List) of programs currently installed on Artemis can be found in the [Artemis User Guide](https://sydneyuni.atlassian.net/wiki/spaces/RC/pages/185827329/Artemis+User+Guide). Software is not '_enabled_' on Artemis by default, and must be loaded before use. This is to avoid conflicts between different software and different versions of software, which may each be dependent on specific versions of other software which will also need to be loaded.

This is all handled by the **modules** package. You can list **available** programs on Artemis using the ```module avail``` command

~~~
[ict_hpctrain1@login3 ~]$ module avail
~~~
{: .source}


~~~
[ict_hpctrain1@login3 ~]$ module avail
--------------------------- /usr/local/Modules/versions ---------------------------
3.2.10

---------------------- /usr/local/Modules/3.2.10/modulefiles ----------------------
dot         module-git  module-info modules     null        use.own
-------------------------- /usr/local/Modules/modulefiles -------------------------
abaqus/2016(default)    glew/2.1.0(default)     openmpi-gcc/3.0.0
abaqus/6.14-1           glib/2.34.0(default)    openmpi-gcc/3.0.0-64
...
git/2.14.1(default)     openmpi-gcc/1.8.4-6.x   zlib/1.2.8(default)
git/2.16.2              openmpi-gcc/1.8.4-backup
gl2ps/1.4.0(default)    openmpi-gcc/2.1.2
~~~
{: .output}

This command may take a minute to run, as there is a _lot_ of software installed! If you know roughly what the program you want is called, you can refine the search with a keyword as extra argument, eg:

~~~
[ict_hpctrain1@login3 ~]$ module avail mat
~~~
{: .source}

~~~
-------------------------- /usr/local/Modules/modulefiles --------------------------
matam/1.0(default)          matlab/R2013a             matlab/R2015b     matlab/R2017a     matlab/R2018a
mathematica/11.1.1(default) matlab/R2014b(default)    matlab/R2016b     matlab/R2017b
~~~
{: .output}

Note that multiple versions of programs may be installed, and one will be designated _default_ -- that is the version that will be loaded if you don't specify a version.

Eg, **load** a program with the ```module load``` command and its name:

~~~
[ict_hpctrain1@login3 ~]$ module load matlab
~~~
{: .source}

then **list** currently loaded programs with ``module list``:

~~~
[ict_hpctrain1@login3 ~]$ module list
~~~
{: .source}

~~~
Currently Loaded Modulefiles:
  1) matlab/R2014b
~~~
  {: .output}

The version ```matlab/R2014b``` was loaded, as it is the current default. **Unload** a loaded module with ```module unload``` and its name. For more info, check out the ```module``` manual page, by executing ```man module```.

<br>
> ## Don't get burned by a version mismatch!
>
> Software usage, syntax and behaviour sometimes change between versions, and if you don't specify a version you may soon find that your code no longer runs or, much worse, it _does_ run, but with different results!
>
> For the sake of your research, and **scientific reproducibility**, you should always _specify the version_ of the module you are loading in your scripts, and record it in your notes.  
{: .callout}
<br>


### Installing new software

All software on Artemis is stored in ```/usr/local```. Users do not have persmission to write to this folder, and hence cannot install new software. If you require a particular piece of software on Artemis, you can submit a [High Performance Computing Request](https://sydneyuni.service-now.com/sm?id=sc_cat_item&sys_id=1ab0bb626d2935008dd31a4dcf150a21&sysparm_category=62460abd4fe297c054a346501310c7d9) through the [ICT Service Portal](https://sydneyuni.service-now.com/sm).

Alternatively, you may be able to install Linux software directly into your _userspace_, ie in your **/home** directory. Not all software can be installed in this way, and there may also be licensing issues -- so don't try this unless you know what you're doing, and please contact us if you don't!


<br>
## Writing script files

### Text editors

Artemis has a number of **text editors** available for use, and of course you could install your own. Text editors are simple-to-sophisticated programs that let you, quite simply, write text! At the most basic level, they do not have any _formatting_, like **bold** etc, only 'plain' text. Many allow the composition of _'rich text'_, which does add formatting.<sup id="a3">[3](#f3)</sup> However, our purpose will be to write _scripts_ that other programs will read in and execute. These programs won't be looking at _formatting_, but will be parsing what we write according to the _syntax_ and keywords of the _programming language_ in use.


To aid with this, some text editors feature _syntax highlighting_, which involves formatting text differently depending on what function that text performs in a given programming language, for example the _MATLAB_ code below:

~~~
function hello_world()
% A function to say hello!
  disp('Hello, World!')
end
~~~
{: .language-matlab}


### i. nano (recommended)

'**Nano**' is a basic, text-only editor with very few other features, and runs inside a _terminal window_. This makes it fast and simple to use, but it may take some getting used to for those unfamiliar with command-line programs. There is no 'point-and-click' interface to nano, only special key combinations to send commands to the program. Eg, to save a file in nano, you would hit the hotkey for 'WriteOut': <kbd>Ctrl</kbd>+<kbd>o</kbd>.

To open nano, simply execute ```nano``` at the command prompt:

~~~
[jdar4135@login2 ~]$ nano
~~~
{: .language-bash}

<figure>
  <img src="{{ page.root }}/fig/02_nano.png" style="margin:10px;height:300px"/>
  <figcaption> The simple, text-only <b>nano</b> text editor. Note the hotkey command list at bottom </figcaption>
</figure><br>

### ii. gedit

For those who'd prefer to use a GUI (graphical user interface) with mouse support, '**gedit**' is good option, and very user friendly. It also performs syntax highlighting, which can be activated from the _View_ menu: _View > Highlight Mode_, and then select the language you would like to parse.

To use gedit on Artemis, you will need to have **X-Forwarding** enabled (see [Setup]({{ page.root }}/setup) guide), which means you'll need to be either on Linux, Mac, or using 'X-Win32' on Windows.<sup id="a4">[4](#f4)</sup>

Open gedit by executing ```gedit &```; the **&** character (called an 'ampersand') tells the **shell** to open the process running gedit in the _background_, allowing you to continue using your terminal while gedit remains open.

~~~
[jdar4135@login2 ~]$ gedit &
~~~
{: .language-bash}

<figure>
  <img src="{{ page.root }}/fig/02_gedit.png" style="margin:10px;height:380px"/>
  <figcaption> A <b>gedit</b> window, with syntax highlighting, served by XQuartz on a Mac </figcaption>
</figure><br>

<br>
_Common operations in nano and gedit_

| Operation | in _nano_ | in _gedit_ |
| --- | --- | --- |
| open new file | ```nano``` | ```gedit &``` |
| open new file 'NewFile' | ```nano NewFile``` | ```gedit NewFile &``` |
| open existing file | ```nano ExistingFile``` | ```gedit ExistingFile &``` |
| save | <kbd>Ctrl</kbd>+<kbd>o</kbd> | <kbd>Ctrl</kbd>+<kbd>s</kbd> |
| exit | <kbd>Ctrl</kbd>+<kbd>x</kbd> | <kbd>Ctrl</kbd>+<kbd>q</kbd> |

<br>
### iii. others

There are dozens of text editors. Some others that you may be familiar with include '**Emacs**' and '**vi/vim**'. These are available on Artemis, and Emacs can be used in GUI mode as well.

<br>
_Common text editors_

| Editor | Ease of use | Power/flexibility | GUI |
| --- | --- | --- | --- |
| **nano** | **Moderate to High** | **Low** | **No** |
| **gedit** | **High** | **Low** | **Yes** |
| Emacs | High (GUI), Moderate | High | Yes |
| vi/vim | Low | High | No |


<br>
> ## Windows character encoding errors
>
> Windows and Linux/OSX use different **character encoding**. This means that some characters (or invisibles!) that appear the same are actually _encoded_ differently -- leading to obscure errors!
>
> This will arise when writing code _on Windows_ machines to be used _on Linux_ ones (like Artemis).
>
> Common encoding issues include:
>* _dashes_ of different lengths, eg - vs -- vs ---. This is often a problem when you _**copy and paste**_ from PDFs onto the command line.
>* _line endings_. Some editors like '**Notepad++**' on Windows let you set which kind of line endings to encode with. Select 'Unix (LF)' if you have the option.
{: .callout}

<br>   

___
**Notes**   
<sup id="f1">1[↩](#a1)</sup> The Research Data Store (RDS) is covered in the 2nd lesson of this series, [Introduction to the Research Data Store and Data Transfer]({{ site.sih_pages }}/training.artemis.rds).

<sup id="f2">2[↩](#a2)</sup> There are ways to change the user permissions of files and folders in Linux, but we won't cover that here. Don't try it unless you know what you're doing!

<sup id="f3">3[↩](#a3)</sup> Programs such as Microsoft Word are not really text-editors, but more '_word processors_', in that they do a lot more than simply compose text. There is not a hard line between the two categories.

<sup id="f4">4[↩](#a4)</sup> If you prefer to use PuTTY terminal, you can use GUI text editors by having both PuTTY and X-Win32 installed. Having X-Win32 open acts as the X server for PuTTY.


___
<br>



{% include links.md %}
