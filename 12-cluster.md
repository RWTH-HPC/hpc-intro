---
title: "Working on a remote HPC system"
teaching: 25
exercises: 10
---



::: questions
- "What is an HPC system?"
- "How does an HPC system work?"
- "How do I log on to a remote HPC system?"
:::

::: objectives
- "Connect to a remote HPC system."
- "Understand the general HPC system architecture."
:::

## What Is an HPC System?

The words "cloud", "cluster", and the phrase "high-performance computing" or
"HPC" are used a lot in different contexts and with various related meanings.
So what do they mean? And more importantly, how do we use them in our work?

The *cloud* is a generic term commonly used to refer to computing resources
that are a) *provisioned* to users on demand or as needed and b) represent real
or *virtual* resources that may be located anywhere on Earth. For example, a
large company with computing resources in Brazil, Zimbabwe and Japan may manage
those resources as its own *internal* cloud and that same company may also
utilize commercial cloud resources provided by Amazon or Google. Cloud
resources may refer to machines performing relatively simple tasks such as
serving websites, providing shared storage, providing web services (such as
e-mail or social media platforms), as well as more traditional compute
intensive tasks such as running a simulation.

The term *HPC system*, on the other hand, describes a stand-alone resource for
computationally intensive workloads. They are typically comprised of a
multitude of integrated processing and storage elements, designed to handle
high volumes of data and/or large numbers of floating-point operations
([FLOPS](https://en.wikipedia.org/wiki/FLOPS)) with the highest possible
performance. For example, all of the machines on the
[Top-500](https://www.top500.org) list are HPC systems. To support these
constraints, an HPC resource must exist in a specific, fixed location:
networking cables can only stretch so far, and electrical and optical signals
can travel only so fast.

The word "cluster" is often used for small to moderate scale HPC resources less
impressive than the [Top-500](https://www.top500.org). Clusters are often
maintained in computing centers that support several such systems, all sharing
common networking and storage to support common compute intensive tasks.

## Logging In

The first step in using a cluster is to establish a connection from our laptop
to the cluster. When we are sitting at a computer (or standing, or holding it
in our hands or on our wrists), we have come to expect a visual display with
icons, widgets, and perhaps some windows or applications: a graphical user
interface, or GUI. Since computer clusters are remote resources that we connect
to over often slow or laggy interfaces (WiFi and VPNs especially), it is more
practical to use a command-line interface, or CLI, in which commands and
results are transmitted via text, only. Anything other than text (images, for
example) must be written to disk and opened with a separate program.

If you have ever opened the Windows Command Prompt or macOS Terminal, you have
seen a CLI. If you have already taken The Carpentries' courses on the UNIX
Shell or Version Control, you have used the CLI on your local machine somewhat
extensively. The only leap to be made here is to open a CLI on a *remote*
machine, while taking some precautions so that other folks on the network can't
see (or change) the commands you're running or the results the remote machine
sends back. We will use the Secure SHell protocol (or SSH) to open an encrypted
network connection between two machines, allowing you to send & receive text
and data without having to worry about prying eyes.

![Connect to cluster](fig/connect-to-remote.svg){alt="Connect to cluster"}

Make sure you have a SSH client installed on your laptop. Refer to the
[setup](../index.md) section for more details. SSH clients are
usually command-line tools, where you provide the remote machine address as the
only required argument. If your username on the remote system differs from what
you use locally, you must provide that as well. If your SSH client has a
graphical front-end, such as PuTTY or MobaXterm, you will set these arguments
before clicking "connect." From the terminal, you'll write something like `ssh
userName@hostname`, where the "@" symbol is used to separate the two parts of a
single argument.

Go ahead and open your terminal or graphical SSH client, then log in to the
cluster using your username and the remote computer you can reach from the
outside world, hpc.itc.rwth-aachen.de.

```bash
you@laptop:~$ ssh ab123456@login23-1.hpc.itc.rwth-aachen.de
```

Remember to replace `ab123456` with your username or the one
supplied by the instructors. You may be asked for your password. Watch out: the
characters you type after the password prompt are not displayed on the screen.
Normal output will resume once you press `Enter`.

## Where Are We?

Very often, many users are tempted to think of a high-performance computing
installation as one giant, magical machine. Sometimes, people will assume that
the computer they've logged onto is the entire computing cluster. So what's
really happening? What computer have we logged on to? The name of the current
computer we are logged onto can be checked with the `hostname` command. (You
may also notice that the current hostname is also part of our prompt!)

```bash
ab123456@login23-1:~$ hostname
```

```output
login23-1
```

::: challenge

## What's in Your Home Directory?

The system administrators may have configured your home directory with some
helpful files, folders, and links (shortcuts) to space reserved for you on
other filesystems. Take a look around and see what you can find.
*Hint:* The shell commands `pwd` and `ls` may come in handy.
Home directory contents vary from user to user. Please discuss any
differences you spot with your neighbors.

:::: solution

## It's a Beautiful Day in the Neighborhood

The deepest layer should differ: `ab123456` is uniquely yours.
Are there differences in the path at higher levels?

If both of you have empty directories, they will look identical. If you
or your neighbor has used the system before, there may be differences. What
are you working on?

Use `pwd` to **p**rint the **w**orking **d**irectory path:

```bash
ab123456@login23-1:~$ pwd
```

You can run `ls` to **l**i**s**t the directory contents, though it's
possible nothing will show up (if no files have been provided). To be sure,
use the `-a` flag to show hidden files, too.

```bash
ab123456@login23-1:~$ ls -a
```

At a minimum, this will show the current directory as `.`, and the parent
directory as `..`.

::::
:::

## Nodes

Individual computers that compose a cluster are typically called *nodes*
(although you will also hear people call them *servers*, *computers* and
*machines*). On a cluster, there are different types of nodes for different
types of tasks. The node where you are right now is called the *head node*,
*login node*, *landing pad*, or *submit node*. A login node serves as an access
point to the cluster.

As a gateway, it is well suited for uploading and downloading files, setting up
software, and running quick tests. Generally speaking, the login node should
not be used for time-consuming or resource-intensive tasks. You should be alert
to this, and check with your site's operators or documentation for details of
what is and isn't allowed. In these lessons, we will avoid running jobs on the
head node.

::: callout

## Dedicated Transfer Nodes

If you want to transfer larger amounts of data to or from the cluster, some
systems offer dedicated nodes for data transfers only. The motivation for
this lies in the fact that larger data transfers should not obstruct
operation of the login node for anybody else. Check with your cluster's
documentation or its support team if such a transfer node is available. As a
rule of thumb, consider all transfers of a volume larger than 500 MB to 1 GB
as large. But these numbers change, e.g., depending on the network connection
of yourself and of your cluster or other factors.
:::


::: callout
For CLAIX-2023 the transfer nodes are as follows:

- copy23-1.hpc.itc.rwth-aachen.de
- copy23-2.hpc.itc.rwth-aachen.de

Please check the presented SSH Key fingerprint against the [fingerprints
published in our documentation](https://help.itc.rwth-aachen.de/service/rhr4fjjutttf/article/0a23d513f31b4cf1849986aaed475789/#fingerprints)
:::

The real work on a cluster gets done by the *worker* (or *compute*) *nodes*.
Worker nodes come in many shapes and sizes, but generally are dedicated to long
or hard tasks that require a lot of computational resources.

All interaction with the worker nodes is handled by a specialized piece of
software called a scheduler (the scheduler used in this lesson is called
**Slurm**). We'll learn more about how to use the
scheduler to submit jobs next, but for now, it can also tell us more
information about the worker nodes.

For example, we can view all of the worker nodes by running the command
`sinfo`.

```bash
ab123456@login23-1:~$ sinfo
```


```output
PARTITION  AVAIL  TIMELIMIT  NODES  STATE NODELIST
[...]
devel         up    1:00:00      2    mix n23m0123,r23m0124
devel_low     up    1:00:00      2    mix n23m0123,r23m0124
c23ms         up 30-00:00:0    123   mix- n23m[0002-0004,0006,0009-0013,0018-0019,0044,0065,0067,0077-0079,0081-0084,0086-0087,0090,0103,0112,0142,0149,0152,0154,0156-0157,0160,0163,0165,0180,0217-0222,0224,0226-0230,0232,0234-0235,0238,0250,0281,0300,0303,0325,0338,0343,0345,0347-0350,0359,0361-0364,0366-0369,0372,0374,0376,0410],r23m[0007,0016,0022,0025,0027,0029,0031,0033-0036,0039-0042,0099,0101,0103-0104,0106,0109,0111,0113-0115,0117-0118,0138,0142,0151,0163-0164,0166,0175,0177,0180-0182,0184-0187,0190-0192,0201]
c23ms         up 30-00:00:0      1  comp* n23m0370
c23ms         up 30-00:00:0      3 drain* n23m0005,r23m[0213,0218]
c23ms         up 30-00:00:0      1   mix* r23m0096
c23ms         up 30-00:00:0      1 alloc* r23m0097
c23ms         up 30-00:00:0      1  down* n23m0213
c23ms         up 30-00:00:0      6  drain n23m[0164,0247,0309],r23m[0001,0011,0132]
c23ms         up 30-00:00:0     98    mix n23m[0031-0032,0037,0040,0045,0063,0068,0093,0095-0096,0100,0106,0108-0109,0114,0118,0122,0126,0138,0169-0170,0173,0176,0178-0179,0182,0190-0191,0198-0199,0211-0212,0214,0239-0240,0242,0244,0248,0251,0254,0257,0260,0269,0273,0283,0286-0287,0294,0297,0302,0307,0311,0313,0316-0318,0329-0330,0336,0341-0342,0351,0395,0397,0406-0407],r23m[0012,0017,0023-0024,0045-0046,0056,0060-0061,0063,0066-0068,0070,0075,0077,0089,0098,0125,0133,0135,0149,0155,0159,0168,0171,0197,0199,0202,0208,0214,0216]
c23ms         up 30-00:00:0    391  alloc n23m[0001,0007-0008,0014-0017,0020-0030,0033-0036,0038-0039,0041-0043,0046-0062,0064,0066,0069-0076,0080,0085,0088-0089,0091-0092,0094,0097-0099,0101-0102,0104-0105,0107,0110-0111,0113,0115-0117,0119-0121,0124-0125,0127-0137,0139-0141,0143-0148,0150-0151,0153,0155,0158-0159,0161-0162,0166-0168,0171-0172,0174-0175,0177,0181,0183-0189,0192-0197,0200-0210,0215-0216,0223,0225,0231,0233,0236-0237,0241,0243,0245-0246,0249,0252-0253,0255-0256,0258-0259,0261-0268,0270-0272,0274-0280,0282,0284-0285,0288-0293,0295-0296,0298-0299,0301,0304-0306,0308,0310,0312,0314-0315,0319-0324,0326-0328,0331-0335,0337,0339-0340,0344,0346,0352-0358,0360,0365,0371,0373,0375,0377-0394,0396,0398-0405,0408-0409],r23m[0002-0006,0008-0010,0013-0015,0018-0021,0026,0028,0030,0032,0037-0038,0043-0044,0047-0055,0057-0059,0062,0064-0065,0069,0071-0074,0076,0078-0088,0090-0095,0100,0102,0105,0107-0108,0110,0112,0116,0119-0123,0126-0131,0134,0136-0137,0139-0141,0143-0148,0150,0152-0154,0156-0158,0160-0162,0165,0167,0169-0170,0172-0174,0176,0178-0179,0183,0188-0189,0195-0196,0198,0200,0203-0207,0209-0212,0215,0217,0219]
c23ms_low     up 30-00:00:0    123   mix- n23m[0002-0004,0006,0009-0013,0018-0019,0044,0065,0067,0077-0079,0081-0084,0086-0087,0090,0103,0112,0142,0149,0152,0154,0156-0157,0160,0163,0165,0180,0217-0222,0224,0226-0230,0232,0234-0235,0238,0250,0281,0300,0303,0325,0338,0343,0345,0347-0350,0359,0361-0364,0366-0369,0372,0374,0376,0410],r23m[0007,0016,0022,0025,0027,0029,0031,0033-0036,0039-0042,0099,0101,0103-0104,0106,0109,0111,0113-0115,0117-0118,0138,0142,0151,0163-0164,0166,0175,0177,0180-0182,0184-0187,0190-0192,0201]
c23ms_low     up 30-00:00:0      1  comp* n23m0370
c23ms_low     up 30-00:00:0      3 drain* n23m0005,r23m[0213,0218]
c23ms_low     up 30-00:00:0      1   mix* r23m0096
c23ms_low     up 30-00:00:0      1 alloc* r23m0097
c23ms_low     up 30-00:00:0      1  down* n23m0213
c23ms_low     up 30-00:00:0      6  drain n23m[0164,0247,0309],r23m[0001,0011,0132]
c23ms_low     up 30-00:00:0     98    mix n23m[0031-0032,0037,0040,0045,0063,0068,0093,0095-0096,0100,0106,0108-0109,0114,0118,0122,0126,0138,0169-0170,0173,0176,0178-0179,0182,0190-0191,0198-0199,0211-0212,0214,0239-0240,0242,0244,0248,0251,0254,0257,0260,0269,0273,0283,0286-0287,0294,0297,0302,0307,0311,0313,0316-0318,0329-0330,0336,0341-0342,0351,0395,0397,0406-0407],r23m[0012,0017,0023-0024,0045-0046,0056,0060-0061,0063,0066-0068,0070,0075,0077,0089,0098,0125,0133,0135,0149,0155,0159,0168,0171,0197,0199,0202,0208,0214,0216]
c23ms_low     up 30-00:00:0    391  alloc n23m[0001,0007-0008,0014-0017,0020-0030,0033-0036,0038-0039,0041-0043,0046-0062,0064,0066,0069-0076,0080,0085,0088-0089,0091-0092,0094,0097-0099,0101-0102,0104-0105,0107,0110-0111,0113,0115-0117,0119-0121,0124-0125,0127-0137,0139-0141,0143-0148,0150-0151,0153,0155,0158-0159,0161-0162,0166-0168,0171-0172,0174-0175,0177,0181,0183-0189,0192-0197,0200-0210,0215-0216,0223,0225,0231,0233,0236-0237,0241,0243,0245-0246,0249,0252-0253,0255-0256,0258-0259,0261-0268,0270-0272,0274-0280,0282,0284-0285,0288-0293,0295-0296,0298-0299,0301,0304-0306,0308,0310,0312,0314-0315,0319-0324,0326-0328,0331-0335,0337,0339-0340,0344,0346,0352-0358,0360,0365,0371,0373,0375,0377-0394,0396,0398-0405,0408-0409],r23m[0002-0006,0008-0010,0013-0015,0018-0021,0026,0028,0030,0032,0037-0038,0043-0044,0047-0055,0057-0059,0062,0064-0065,0069,0071-0074,0076,0078-0088,0090-0095,0100,0102,0105,0107-0108,0110,0112,0116,0119-0123,0126-0131,0134,0136-0137,0139-0141,0143-0148,0150,0152-0154,0156-0158,0160-0162,0165,0167,0169-0170,0172-0174,0176,0178-0179,0183,0188-0189,0195-0196,0198,0200,0203-0207,0209-0212,0215,0217,0219]
c23mm         up 30-00:00:0     97   mix- n23m[0002-0004,0006,0009-0013,0018-0019,0077-0079,0081-0084,0086-0087,0090,0149,0152,0154,0156-0157,0160,0163,0217-0222,0224,0226-0230,0232,0234-0235,0343,0345,0347-0350,0359,0361-0364,0366-0369,0372,0374,0376,0410],r23m[0027,0029,0031,0033-0036,0039-0042,0099,0101,0103-0104,0106,0109,0111,0113-0115,0117-0118,0175,0177,0180-0182,0184-0187,0190-0192]
c23mm         up 30-00:00:0      1  comp* n23m0370
c23mm         up 30-00:00:0      1 drain* n23m0005
c23mm         up 30-00:00:0      1  drain n23m0164
c23mm         up 30-00:00:0     65  alloc n23m[0001,0007-0008,0014-0017,0020,0073-0076,0080,0085,0088-0089,0091-0092,0145-0148,0150-0151,0153,0155,0158-0159,0161-0162,0223,0225,0231,0233,0236,0289-0290,0344,0346,0360,0365,0371,0373,0375,0377-0378,0409],r23m[0028,0030,0037-0038,0100,0102,0105,0107-0108,0110,0112,0116,0176,0178-0179,0183,0188-0189]
c23mm_low     up 30-00:00:0     97   mix- n23m[0002-0004,0006,0009-0013,0018-0019,0077-0079,0081-0084,0086-0087,0090,0149,0152,0154,0156-0157,0160,0163,0217-0222,0224,0226-0230,0232,0234-0235,0343,0345,0347-0350,0359,0361-0364,0366-0369,0372,0374,0376,0410],r23m[0027,0029,0031,0033-0036,0039-0042,0099,0101,0103-0104,0106,0109,0111,0113-0115,0117-0118,0175,0177,0180-0182,0184-0187,0190-0192]
c23mm_low     up 30-00:00:0      1  comp* n23m0370
c23mm_low     up 30-00:00:0      1 drain* n23m0005
c23mm_low     up 30-00:00:0      1  drain n23m0164
c23mm_low     up 30-00:00:0     65  alloc n23m[0001,0007-0008,0014-0017,0020,0073-0076,0080,0085,0088-0089,0091-0092,0145-0148,0150-0151,0153,0155,0158-0159,0161-0162,0223,0225,0231,0233,0236,0289-0290,0344,0346,0360,0365,0371,0373,0375,0377-0378,0409],r23m[0028,0030,0037-0038,0100,0102,0105,0107-0108,0110,0112,0116,0176,0178-0179,0183,0188-0189]
c23ml         up 30-00:00:0      2  alloc n23m[0289-0290]
c23ml_low     up 30-00:00:0      2  alloc n23m[0289-0290]
c23g          up 30-00:00:0     30   mix- n23g[0001,0003-0004,0006,0009,0011-0017,0020-0024,0026-0027],r23g[0002-0003,0005],w23g[0002,0004-0010]
c23g          up 30-00:00:0      2 drain$ n23g[0028-0029]
c23g          up 30-00:00:0      2  drain n23g0030,w23g0001
c23g          up 30-00:00:0      3   resv w23g[0012-0014]
c23g          up 30-00:00:0      2    mix n23g0031,w23g0011
c23g          up 30-00:00:0     11  alloc n23g[0002,0005,0007-0008,0010,0018-0019,0025],r23g[0001,0004],w23g0003
c23g_low      up 30-00:00:0     30   mix- n23g[0001,0003-0004,0006,0009,0011-0017,0020-0024,0026-0027],r23g[0002-0003,0005],w23g[0002,0004-0010]
c23g_low      up 30-00:00:0      2 drain$ n23g[0028-0029]
c23g_low      up 30-00:00:0      2  drain n23g0030,w23g0001
c23g_low      up 30-00:00:0      3   resv w23g[0012-0014]
c23g_low      up 30-00:00:0      2    mix n23g0031,w23g0011
c23g_low      up 30-00:00:0     11  alloc n23g[0002,0005,0007-0008,0010,0018-0019,0025],r23g[0001,0004],w23g0003
c23i          up 30-00:00:0      1   idle n23i0001
[...]
```
::: callout
The list of partitions above is truncated to the most relevant for standard
users.

There are also specialized machines used for managing disk storage, user
authentication, and other infrastructure-related tasks. Although we do not
typically logon to or interact with these machines directly, they enable a
number of key features like ensuring our user account and files are available
throughout the HPC system.

## What\'s in a Node?

All of the nodes in an HPC system have the same components as your own laptop
or desktop: *CPUs* (sometimes also called *processors* or *cores*), *memory*
(or *RAM*), and *disk* space. CPUs are a computer's tool for actually running
programs and calculations. Information about a current task is stored in the
computer's memory. Disk refers to all storage that can be accessed like a file
system. This is generally storage that can hold data permanently, i.e. data is
still there even if the computer has been restarted. While this storage can be
local (a hard drive installed inside of it), it is more common for nodes to
connect to a shared, remote fileserver or cluster of servers.

![Node anatomy](fig/node_anatomy.png){max-width="20%" alt="Node anatomy" caption=""}

::: challenge

## Explore Your Computer

Try to find out the number of CPUs and amount of memory available on your
personal computer.
Note that, if you're logged in to the remote computer cluster, you need to
log out first. To do so, type `Ctrl+d` or `exit`:

```bash
ab123456@login23-1:~$ exit
you@laptop:~$
```

:::: solution

There are several ways to do this. Most operating systems have a graphical
system monitor, like the Windows Task Manager. More detailed information can
sometimes be found on the command line. For example, some of the commands used
on a Linux system are:

Run system utilities

```bash
you@laptop:~$ nproc --all
you@laptop:~$ free -m
```

Read from `/proc`

```bash
you@laptop:~$ cat /proc/cpuinfo
you@laptop:~$ cat /proc/meminfo
```

Use a system monitor

```bash
you@laptop:~$ htop
```

::::
:::

::: challenge

## Explore the login node

Now compare the resources of your computer with those of the head node.

:::: solution

```bash
you@laptop:~$ ssh ab123456@login23-1.hpc.itc.rwth-aachen.de
ab123456@login23-1:~$ nproc --all
ab123456@login23-1:~$ free -m
```

You can get more information about the processors using `lscpu`,
and a lot of detail about the memory by reading the file `/proc/meminfo`:

```bash
ab123456@login23-1:~$ less /proc/meminfo
```

You can also explore the available filesystems using `df` to show **d**isk
**f**ree space. The `-h` flag renders the sizes in a human-friendly format,
i.e., GB instead of B. The **t**ype flag `-T` shows what kind of filesystem
each resource is.

```bash
ab123456@login23-1:~$ df -Th
```
::::
:::

::: discussion
The local filesystems (ext, tmp, xfs, zfs) will depend on whether you're
on the same login node (or compute node, later on). Networked filesystems
(beegfs, cifs, gpfs, nfs, pvfs) will be similar --- but may include
ab123456, depending on how it is [mounted](
https://en.wikipedia.org/wiki/Mount_(computing)).
:::

::: callout
## Shared Filesystems

This is an important point to remember: files saved on one node
(computer) are often available everywhere on the cluster!

:::


::: challenge

## Explore a Worker Node

Finally, let's look at the resources available on the worker nodes
where your jobs will actually run. Try running this command to see
the name, CPUs and memory available on one of the worker nodes:

```bash
ab123456@login23-1:~$ sinfo -o "%n %c %m" | column -t
```
:::

::: discussion
## Compare Your Computer, the login node and the compute node
Compare your laptop's number of processors and memory with the numbers you
see on the cluster head node and worker node. Discuss the differences with
your neighbor.

What implications do you think the differences might have on running your
research work on the different systems and nodes?
:::

::: callout
## Differences Between Nodes

Many HPC clusters have a variety of nodes optimized for particular workloads.
Some nodes may have larger amount of memory, or specialized resources such as
Graphical Processing Units (GPUs).
:::

With all of this in mind, we will now cover how to talk to the cluster's
scheduler, and use it to start running our scripts and programs!

::: keypoints
 - "An HPC system is a set of networked machines."
 - "HPC systems typically provide login nodes and a set of worker nodes."
 - "The resources found on independent (worker) nodes can vary in volume and
   type (amount of RAM, processor architecture, availability of network mounted
   filesystems, etc.)."
 - "Files saved on one node are available on all nodes."
:::
