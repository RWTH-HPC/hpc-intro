---
title: "Navigating and selecting filesystems"
teaching: 25
exercises: 10
---



::::::::::::::::::::::::::::::::::::::: objectives

- List different types of filesystems present on HPC systems
- Idenfiy the best filesystem for a given task

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- Where do I put my data on an HPC system?
- Does it impact performance and data security where I place my data?

::::::::::::::::::::::::::::::::::::::::::::::::::

## Aspect of filesystems

In HPC, filesystems play a crucial role in managing data efficiently.
When selecting or working with filesystems in HPC environments the following key aspects should be considered:

- Performance
- Reliability
- Capacity

### Performance

Performance is paramount in HPC systems, where computational tasks often involve processing vast amounts of data.
The filesystem must support high throughput and low latency to ensure that data can be read from and written to storage as quickly as possible.
Factors influencing performance include:

- **I/O Bandwidth**: The amount of data that can be transferred to and from storage per unit time.
- **Latency**: The time it takes for a request to be processed by the filesystem.
- **Concurrency**: The ability of the filesystem to handle multiple simultaneous requests efficiently.

Optimizing these factors is essential for maximizing the overall performance of HPC applications.
Better performance can be achieved by better or more hardware.

### Reliability

Reliability refers to the ability of a filesystem to maintain data integrity and availability over time.
In HPC environments, where failures can have significant consequences, ensuring that data is protected against corruption or loss is critical.
Key aspects of reliability include:

- **Data Redundancy**: Techniques such as RAID (Redundant Array of Independent Disks) or replication help safeguard against hardware failures.
- **Error Detection and Correction**: Mechanisms that identify and correct errors during data transmission or storage.
- **Backup Solutions**: Regular backups are vital for recovering data in case of catastrophic failures.

A reliable filesystem ensures that users can trust their data will remain intact and accessible when needed.
Better reliability can be achieved by adding redundancies to the storage.

### Capacity

Capacity pertains to the amount of data that a filesystem can store.
As datasets continue to grow exponentially, especially in fields like genomics, climate modeling, and simulations, having sufficient capacity becomes increasingly important. Considerations regarding capacity include:

- **Scalability**: The ability of the filesystem to expand its storage capabilities seamlessly as requirements increase.
- **Storage Management**: Efficient management techniques such as tiered storage or archiving solutions help optimize space utilization.
- **Quota Management**: Implementing quotas allows administrators to control how much space each user or project can consume.

Better capacity can be achieved by adding more hardware.

It is usually difficult to optimize for all of these dimensions with a fixed budget.
Therefore, HPC systems provide different filesystems to the users that each excel in one or more dimension, and let the user decide which filesystem to use when.

::: callout
The "home" filesystem is usually considered the most reliable, whereas a "work" filesystem puts more emphasis on performance.
Next to the `$HOME` variable common on UNIX systems, many HPC systems provide a `$WORK` variable for the faster filesystem.
:::


On CLAIX the following variables are available:

- `$HOME` points to the home folder of the user, which is considered to be the most reliable. This filesystem is backed up daily on tape storage off-site the HPC facility.
   Use this for data that are hard to reproduce and most valuable to you.

- `$WORK` points to the "standard" working filesystem. This filesystem is not backed up, but provides a snapshot feature once per day for past seven days and once per week for the past 5 weeks.
   Use this for everyday work items and software installations that you don't need to have backed up long-term and need reasonable performance.

- `$HPCWORK` points to a high-capacity, high-performance filesystem. This filesystem is not backed up and provides no snapshots.
  This filesystem is best suited for **large-file I/O** from **many processes**.
  It is less suited for I/O of many small files.
  Use this if your application does parallel I/O with multiple processes to larger files.

- `$BEEOND` points to a BeeGFS-on-demand filesystem that is set up **on demand**, spanning the SSDs of the nodes of an exclusive, multi-node job.
  To trigger the setup of this filesystem for your job, you need to pass `--beeond` to the `sbatch` command or add `#SBATCH --beeond` to your job file.
  Use this if your job spans multiple nodes and you need I/O to many small files.
  This filesystem is volatile, in that you need to **copy in** data at the beginning of the job, and **copy out** any data you need saved before the end of the job.

- `$TMP` points to the local SSD disk of the node. The filesystem is not shared across nodes, so only practical for single-node jobs.
  The SSD is fast and can handle small files.
  This filesystem is volatile, in that you need to **copy in** data at the beginning of the job, and **copy out** any data you need saved before the end of the job.


## Filesystem quota

Filesystems on HPC systems are a shared resource.
Just like with the available compute power of the HPC system, the filesystem resources need to be managed to keep individual use influencing other users.
Therefore, most HPC systems set quotas, both for total bytes used by files as well as the number of files.

When filesystem quotas are set, the HPC system provides a means to query your current use of the quotas.

```bash
ab123456@login23-1:~$ r_quota
```

```output
                        ------------ Blocks ------------- ------------- Files -------------
Object                    used   soft   hard        grace   used   soft   hard        grace
/home/ab123456            259G      -   300G            -  1458K      -  1512K            -
/work/ab123456            174G      -   250G            -   724K      -  1024K            -
/hpcwork/ab123456         679G  1000G  1100G            -    50K  1024K  1088K            -
/hpcwork/ab123456(ssd)     11M  1024M  1024M            -      -      -      -            -
```

## Best practices

There is no single best filesystem for all use cases.
Here is a list of advice that may guide your selection of filesystems:

- Select a highly reliable filesystems for important data.
- Select the most performant filesystem that fits your input or output characteristics
- Avoid many small files on parallel filesystems


::: caution
Parallel filesystems can be sensitive to metadata query overload.
Concurrent access to many small files may overload the filesystem metadata server and deteriorate overall filesystem performance for **all users** of the HPC system.
:::


The CLAIX system provides the following different filesystems:

| Access     | Type   | Capacity Quota             | File Quota | Backup          | Performance          |
| ---------- | ------ | -------------------------- | ---------- | --------------- | -------------------- |
| `$HOME`    | GPFS   | 250 GB                     | 1 mio.     | Tape (off-site) | :star:               |
| `$WORK`    | GPFS   | 250 GB                     | 1 mio.     | Snapshots       | :star: :star:        |
| `$HPCWORK` | Lustre | 1 TB                       | 1 mio.     | :x:             | :star: :star: :star: |
| `$BEEOND`  | BeeGFS | 1.5 TB (HPC) / 682 GB (ML) |            | :x:             | :star: :star: :star: |
| `$TMP`     | XFS    | 1.5 TB (HPC) / 682 GB (ML) |            | :x:             | :star: :star: :star: |

:::::::::::::::::::::::::::::::::::::::: keypoints

- HPC systems can employ many different filesystems
- Filesystems can vary in performance, capacity, and reliance
- Balancing capacity with performance and reliability is essential for effective resource management in HPC systems.
- Have data that can not be recreated easily reside on the most reliable file system

::::::::::::::::::::::::::::::::::::::::::::::::::

