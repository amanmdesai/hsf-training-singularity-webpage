---
title: "Containers and Images"
teaching: 20
exercises: 5
questions:
- "How to pull Singularity images from the libraries?"
- "How to run commands inside the containers?"
objectives:
- "Learn to search and pull images from the Singularity library and Docker Hub."
- "Interact with the containers using the command-line interface."
keypoints:
- "Singularity images are accessible from the CLI, and they become containers during the execution."
- "A container can be  from a local `.sif` or directly with the URL of the image."
- "Singularity is also compatible with Docker images, providing access to the large collection of images hosted by Docker Hub."
---

# The Singularity Command Line Interface

Singularity provides a command-line interface (CLI) to interact with the containers. You can search, build or run
containers in a single line.

You can check the available options and subcommands using `--help`:

```bash
singularity --help
```

# Downloading Images

Apptainer/Singularity can store, search and retrieve images in registries.
Images built by other users can be accessible using the CLI, can be pulled down, and become containers at runtime.

Sylabs, the developers of one Singularity flavor, hosts a public image registry, the
[Singularity Container Library](https://cloud.sylabs.io/library) where many user built images are available.

The Linux Foundation flavor, Apptainer, does not point by default to the Sylab registry as previous versions did.
You can change that running these commands (documented [here](https://apptainer.org/docs/user/main/endpoint.html#restoring-pre-apptainer-library-behavior)):
```bash
apptainer remote add --no-login SylabsCloud cloud.sycloud.io
```
~~~
INFO:    Remote "SylabsCloud" added.
~~~
{: .output}
```bash
apptainer remote use SylabsCloud
```
~~~
INFO:    Remote "SylabsCloud" now in use.
~~~
{: .output}
```bash
apptainer remote list
```
~~~
Cloud Services Endpoints
========================

NAME           URI                  ACTIVE  GLOBAL  EXCLUSIVE
DefaultRemote  cloud.apptainer.org  NO      YES     NO
SylabsCloud    cloud.sycloud.io     YES     NO      NO
...
~~~
{: .output}

Once you have setup a working registry you can use search and pull.
The command `search` provides containers of interest
and information about groups and collections. For example:

```bash
singularity search centos7
```

~~~
No users found for 'centos7'

Found 1 collections for 'centos7'
        library://shahzebmsiddiqui/easybuild-centos7

Found 15 containers for 'centos7'
        library://gmk/default/centos7-devel
                Tags: latest
...
~~~
{: .output}

Downloading an image from the Container Library is pretty straightforward:
```bash
singularity pull library://gmk/default/centos7-devel
```
and the image is stored locally as a `.sif` file (`centos7-devel_latest.sif`, in this case).

> ## Docker Images
>
> Fortunately, Singularity is also compatible with Docker images. There are many more registries with Docker images.
> [Docker Hub](https://hub.docker.com/) is one of the largest libraries available,
> and any image hosted on the hub can be easily downloaded with the `docker://` URL as reference:
> ```bash
> singularity pull docker://centos:centos7
> ```
{: .callout}

# Running Containers

There are several ways to interact with images and start containers. Here we will review how to initialize a shell
environment and how to execute directly a command.

## Initializing a shell

The `shell` command initializes a new interactive shell inside the container.

```bash
singularity shell centos7-devel_latest.sif
```

~~~
Singularity>
~~~
{: .output}
In this case, the container works as a lightweight virtual machine in which you can execute commands.
Remember, inside the container you have the same user and permissions.

```bash
Singularity> id
```

~~~
uid=1001(myuser) gid=1001(myuser) groups=1001(myuser),500(myothergroup)
~~~
{: .output}

When an outside directory is accessible also inside Singularity we say it is bound, or bind mounted. The path to access it
may differ but anything you do to its content outside is visible inside and vice-versa.
By default, Singularity bind the home of the user, `/tmp` and `$PWD` into the container. It means your files
at `hostname:~/` are accessible inside the container. You can specify additional bind mounts using the `--bind` option.
For example, let's say `/cvmfs` is available in the host, and you would like to have access to CVMFS inside the
container. Then let's do

```bash
singularity shell --bind /cvmfs:/cvmfs centos7-devel_latest.sif
```

~~~
Singularity> ls /cvmfs/cms.cern.ch
bin                        etc                  SITECONF           slc7_aarch64_gcc530
bootstrap.sh               external             slc5_amd64_gcc434  slc7_aarch64_gcc700
...
~~~
{: .output}

> ## URLs as input
> Each of the different commands to set a container from a local `.sif` also accepts the URL of the image
> as input. For example, starting a shell with Scientific Linux 6 is as easy as
> ```bash
> singularity shell docker://sl:6
> ```
> ~~~
> 2020/12/17 21:42:46  info unpack layer: sha256:e0a6b33502f39d76f7c70213fa5b91688a46c2217ad9ba7a4d1690d33c6675ef
> INFO:    Creating SIF file...
> Singularity>
> ~~~
> {: .output}
{: .callout}

## Executing commands

The command `exec` starts the container from a specified image and executes a command inside it.
Let's use the official [Docker image of ROOT](https://hub.docker.com/r/rootproject/root) to start ROOT
inside a container:

```bash
singularity exec docker://rootproject/root root -b
```

~~~
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...

   ------------------------------------------------------------------
  | Welcome to ROOT 6.22/06                        https://root.cern |
  | (c) 1995-2020, The ROOT Team; conception: R. Brun, F. Rademakers |
  | Built for linuxx8664gcc on Nov 27 2020, 15:14:08                 |
  | From tags/v6-22-06@v6-22-06                                      |
  | Try '.help', '.demo', '.license', '.credits', '.quit'/'.q'       |
   ------------------------------------------------------------------

/bin/bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
root [0]

~~~
{: .output}

And just like that, ROOT can be used in any laptop, large-scale cluster or grid system
with Singularity available.

> ## Execute Python with PyROOT available
>
> Using the official Docker image of ROOT, start a Python session with PyROOT available.
>
> > ## Solution
> >
> > ```bash
> > singularity exec docker://rootproject/root python3
> > ```
> >
> > ~~~
> > INFO:    Using cached SIF image
> > Python 3.8.5 (default, Jul 28 2020, 12:59:40)
> > [GCC 9.3.0] on linux
> > >>> import ROOT
> > >>> # Now you can work with PyROOT, creating a histogram for example
> > >>> h = ROOT.TH1F("myHistogram", "myTitle", 50, -10, 10)
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}

## Exiting a singularity image

The `exit` command (Control+d) exits a singularity instance. Note that when exiting from the singularity image all the running processes are killed (stopped).
Changes saved into bound directories are preserved. By default anything else in the container is lost (we'll see later about writable images).

```bash
Singularity> exit
```

{% include links.md %}
