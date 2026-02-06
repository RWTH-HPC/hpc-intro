---
title: Using software via containers
teaching: 30
exercises: 60
---



::::::::::::::::::::::::::::::::::::::: objectives

- Create a container image for an application
- Use the container image on the login node
- Submit batch jobs using the container image in parallel execution

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How to provide a complex software environment with many dependencies?
- How to pass software environments to others for reproducibility?

::::::::::::::::::::::::::::::::::::::::::::::::::

In the previous episode you ran the Amdahl Python MPI program on the Cluster using the Python and MPI modules.

In practice, scientific software stacks can be

- **complex** (i.e., have many dependencies),
- **fragile** (i.e., dependencies on and conflicts with different versions of software in the stack), and
- **different** on every HPC system,

**Containers** help with this by packaging what defines your software environment into a single file that you can move between systems and run reproducibly.

In this episode we will

- introduce [Apptainer][apptainer] (formerly named "Singularity"), a container engine often used for HPC systems,
- build a container for the Amdahl MPI Python example, and
- run amdahl from inside the container.

::: callout
You may have heard about other container engines, such as Docker or Podman and their functionality is often similar.
Apptainer is a popular choice for HPC systems, used by many different people, as it runs images as **you** without the need for administrative rights to the system.
:::

## Apptainer basics

As the software stack on HPC systems is often very specific, the apptainer executable can be provided in different ways.


The `apptainer` executable on CLAIX is installed directly as a system package and is available without loading additional modules.

Running apptainer without any arguments will yield a list of commands available.
The most basic ones are

- `apptainer pull` to download an existing container image from a remote repository
- `apptainer build` to build a container image from a given configuration file
- `apptainer exec` to execute a specifc command inside the image
- `apptainer run` to execute the configured default command inside the image
- `apptainer shell` to start an interactive shell inside the image

::: callout
Apptainer containers share the host kernel, but have their own user-space (i.e., libraries, executables, etc.).
This is why they are much lighter than virtual machines and well-suited for HPC.
:::

## Best practices

When building containers, several best practices should be considered to keep the container as broadly useable as possible.

### Containers should be stateless

State is additional information that might be used for execution of the containerized software.
The container image should only change with version updates to the containerized software.
Updating 
However, when the container is supposed to be used in many different scenarios, it is best to use so-called **bindpaths** to pass such additional information in to the container environment.

::: callout
Packaging state into a container can make sense when you want to preparte a container for reproducibility in a single execution context.
:::

### Keep definition files small

Large container images make it harder to move the container due to its size.
Furthermore, the more software is installed as part of the image, the more can potentially interact and reduce portability.
Moreover, small image footprints improve overall container performance.
As a rule of thumb, only include software in the container that is specifically needed.

### Keep your image as SIF on the right filesystem.

Build containers do not need to be backed up, as you can recreate them anytime from the configuration file.
Keeping large image files puts unnessessary stress on the backup system in place for the HPC system.
The definition file, however, should be kept on a file system that is backed up, or even better in a version control system external to the HPC system.


For CLAIX it is recommended to keep container images on the `$WORK` filesystem.

## A basic container configuration

The following basic container definition will create a container that echos back the arguments passed to it while also providing a basic build environment.

```dockerfile
Bootstrap: Docker
From: rockylinux:9

%post
    echo "Installing required packages"
    # configure package manager to work non-interactive and reduce load while downloading packages
    printf '%s\n' 'assumeyes=true' 'max_parallel_downloads=15' >> /etc/dnf/dnf.conf
    # get up-to-date package information
    dnf update
    # install compilers and build environments
    dnf group install "Development Tools"
    # install wget to download files using HTTP.
    dnf install wget
    # Clean up any cached information, build files, etc.
    dnf clean all

    echo "Here you would install your actual application you want to containerized

%runscript
    # set up a default command inside your image to simplify batch scripts
    echo $"These were your arguments:"
    exec echo "$@"
```

The `%` symbol indicates different sections in your container definition.
The `%post` section contains all shell commands you want to be executed during the build of the container.
The `%runscript` section contains all commands executed during an invocation with `apptainer run`.

You can build a container image using the following command.

```sh
$ apptainer build example.sif example.def
```
```output
INFO:    User not listed in /etc/subuid, trying root-mapped namespace
INFO:    The %post section will be run under the fakeroot command
INFO:    Starting build...
INFO:    Fetching OCI image...
61.3MiB / 61.3MiB [=======================================================================================================================================================================================================] 100 % 0.0 b/s 0s
INFO:    Extracting OCI image...
INFO:    Inserting Apptainer configuration...
INFO:    Running post scriptlet
+ echo 'Installing required packages'
Installing required packages
+ printf '%s\n' assumeyes=true max_parallel_downloads=15
+ dnf update
Rocky Linux 9 - BaseOS                                                                                                                                                                                       24 MB/s | 9.9 MB     00:00    
Rocky Linux 9 - AppStream                                                                                                                                                                                    28 MB/s |  14 MB     00:00    
Rocky Linux 9 - Extras                                                                                                                                                                                       73 kB/s |  17 kB     00:00    
Dependencies resolved.
============================================================================================================================================================================================================================================
 Package                                                            Architecture                                  Version                                                            Repository                                        Size
============================================================================================================================================================================================================================================
Upgrading:
 alternatives                                                       x86_64                                        1.24-2.el9                                                         baseos                                            38 k
 audit-libs                                                         x86_64                                        3.1.5-7.el9                                                        baseos                                           121 k
 basesystem                                                         noarch                                        11-13.el9.0.1                                                      baseos                                           6.4 k
 bash                                                               x86_64                                        5.1.8-9.el9                                                        baseos                                           1.7 M
[...]
Complete!
+ dnf clean all
27 files removed
+ echo 'Here you would install your actual application you want to containerized'
Here you would install your actual application you want to containerized
INFO:    Adding runscript
INFO:    Creating SIF file...
[=================================================================================================================================================================================================================================] 100 % 0s
INFO:    Build complete: example.sif

```

Now you can execute this with the following command.

```sh
$ apptainer run example.sif Hello World
```
```output
These were your arguments:
Hello World
```

You can also explore the image with the follwing commands.
Notice how apptainer modifies your prompt to indicate that you are inside a container.
```sh
$ apptainer shell example.sif
Apptainer> gcc --version
```
```output
gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-11)
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

This is the version of the GCC that was installed as part of your image.

## Apptainer and MPI

Amdahl is using MPI for communication.
MPI Libraries are often using libraries very close to the operating system.
Containerizing a complete MPI installation can be tricky.
A fully containerized MPI installation may be dependent on libraries specific to the HPC system it was built on.
This can reduce portability significantly, as creators cannot prepare for all possible future execution scenarios of an image

As a solution, choose between the follwing three container creation models depending on your use case.

### Fully-containerized model

This model includes a fully containerized MPI and PMI (the process manager to start MPI processes) implementations.
The entire MPI stack and all dependent libraries are part of the container image.
Therefore, the container needs to compile in hardware support of the future execution host.

- :white_check_mark: Most portable solution
- :white_check_mark: Full control over used MPI implementation
- :white_check_mark: Images also usable for hybrid model
- :x: Requires proper base images or understanding of MPI

### Bind model

This model does not include an MPI installation.
Therefore the MPI libraries have to be bind-mounted into the image, and the application still needs to be built and linked against the bind-mounted MPI implementation.

- :white_check_mark: Provides MPI support for exotic use cases
- :x: Application is fully hostdependent
- :x: Configuration is convoluted and error-prone

### Hybrid model

This model uses a mix of the previous two models and often serves as the best option.
The application is built and linked agains an MPI that is part of the container.
However, at runtime the host MPI implementation is used via dynamic linking, taking care of all the specific requirements of the host platform.
This requires the MPI libraries on the host and in the container to be ABI-compatible (Application Binary Interface).

- :white_check_mark: Works with foreign containers
- :white_check_mark: Integrates with Slurm
- :x: Requires matching MPIs on host and container

::: callout
While the newest version of the MPI standard specifies ABI compatibility, MPI libraries for older MPI versions are often **not** ABI-compatible.
MPICH and Intel-MPI are often ABI-compatible; Open-MPI, howver, is not compatible with the former two.
:::


::: challenge
Create a container for the Amdahl Python application and run it on the cluster.
Use the following template for a hybrid model container.

:::::: solution
tbd.
::::::
:::

[apptainer]: https://apptainer.org
[amdahl]: https://en.wikipedia.org/wiki/Amdahl\'s_law
[parallel-novice]: https://www.hpc-carpentry.org/hpc-parallel-novice/


:::::::::::::::::::::::::::::::::::::::: keypoints

- Containers provide the means to move software installations between HPC systems.
- Some frameworks are already provided as container images by their vendors.
- Using MPI with containers can be tricky.
- Choose between "Containerized", "Bind", and "Hybrid" Model depending on your needs.

::::::::::::::::::::::::::::::::::::::::::::::::::
