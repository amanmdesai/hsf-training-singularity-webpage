---
title: Setup
---

In this document we use Apptainer and Singularity interchangeably. See the [Introduction]({{ page.root }}{% link _episodes/01-introduction.md %})
for more details about existing Apptainer and Singularity versions and the differences between them.

Apptainer/Singularity has become popular and usually it is available in the institutional computing resources.
Check if singularity is available with
```bash
singularity --version
```
If installed, you will see `apptainer version ...` or `singularity version ...`, depending on the flavor installed.
If it is not in your PATH you may still be able to use it via CVMFS: check if you have user namespaces enabled and CVMFS to run singularity that way:
```bash
[[ $(cat /proc/sys/user/max_user_namespaces) -gt 0 ]] && ls /cvmfs/oasis.opensciencegrid.org/mis/ &>/dev/null && { export PATH=/cvmfs/oasis.opensciencegrid.org/mis/apptainer/bin/:"$PATH"; echo "Added to PATH"; singularity --version; } || echo "Unable to run Singularity/Apptainer via CVMFS"
```
If this works, it will be added to your path and you will see your singularity/apptainer version.

Otherwise you will need to install or ask to install Apptainer/Singularity.
You will need a Linux system (including WSL on Windows computers) to run Apptainer/Singularity natively and it’s easiest to
[install if you have root access](https://apptainer.org/docs/user/main/quick_start.html#quick-installation).
If your local computing system does not have Apptainer/Singularity installed, you may
[request it to your system administrator as suggested here](https://apptainer.org/docs/user/main/quick_start.html#apptainer-on-a-shared-resource).
If the above is not possible and you cannot use the CVMFS distribution you can
[install from source without root privileges](https://github.com/apptainer/apptainer/blob/main/INSTALL.md).
If you have user namespaces (`cat /proc/sys/user/max_user_namespaces` is bigger than 0) you have one more option, you can use [cvmfsexec](https://github.com/cvmfs/cvmfsexec) to get CVMFS.
This is a bit more complex, you can follow the instrictions summarized also in
[this paper](https://indico.cern.ch/event/885212/contributions/4120683/attachments/2181040/3684201/CernVMWorkshopCvmfsExec20210201.pdf).


{% include links.md %}
