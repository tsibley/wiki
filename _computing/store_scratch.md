---
title: Temporary Storage: Scratch
last_modified_at: 2018-06-20
---

## Why use Scratch Storage for temporary data?

In bioinformatics workflows we are often using pipelines with many execution steps. Each of these steps can create a large amount of temporary data (e.g. extracting information from genomics data (BAM files). This data often needs to be kept for a short period of time to allow for quality assurance.

Informaticians often do not delete this data after this step because they are already off to the next task. Even worse, if temporary data is created in a standard file system such as "Fast File" it will be picked up by the backup system and copied to the cloud the next night. If data is frequently created and deleted the backup data can grow to **5 or even 10 times the size** of the primary data which is an enormous waste. To prevent this waste every Informatician or Data Scientist working with large datasets should use **Scratch** as part of their routine.

For this purpose we have a scratch file systems attached to the Gizmo, Beagle and Koshu clusters. There is a different scratch file system mounted on each cluster.  Using a scratch resource has several advantages:

- The scratch file system is free of charge
- It is the most performant storage system connected to Rhino/Gizmo
- You do not have to clean up your temporary data because the system does it for you
- It reduces Fred Hutch computing expenses because the data is not backed up to tape

## Types of Scratch Storage Available

On Gizmo there are three forms of scratch space available: "node local job scratch", "network job scratch" and "network monthly scratch".  The "network job scratch"  and  "node local"  scratch directories and their contents exist only for the duration of the job- when the job exits, the directory and its contents are removed.  
For more persistent scratch space, ​please see the persistent / monthly Scratch section.


### Node local Job Scratch

There are varying volumes of local storage depending on node configuration and utilization by other jobs.  If you require a large volume of local disk, request it with the "--tmp" argument:
```
sbatch -n 2 -t 1-0 --tmp=4096 # requests 4GB of disk space
```
Note that this only ensures that the disk is available when the job starts.  Other processes may fill up this scratch space, causing problems with your job.
The location of this local scratch space is stored in the environment variable "TMPDIR" and "SCRATCH_LOCAL- use this environment variable if you need local storage on the node- do not use "/tmp" for storage of files or for scratch space.

### Network Job Scratch

Partition global scratch space is a scratch directory is created on storage that is available to all nodes in your job's allocation.  The directory is based on the job ID.  You should access the job scratch directory by using the environment variable "$SCRATCH" in your scripts.

### Persistent / Monthly scratch

Sometimes you need to work with temporary data that is not part of a specific pipeline, for example if you are doing manual QA on data for a few days or even weeks.


## How can I use Scratch?

There are multiple types of scratch but here we focus on monthly scratch. You can reach it via environment variable $DELETE30 (which currently points to `/fh/scratch/delete30`), for example:

in your Shell
```
$ ls -l $DELETE30/
$ ls -l $DELETE30/lastname_f
$ runprog.R > $DELETE30/lastname_f/datafile.dat
```
in Python
```
#! /usr/bin/env python3
import os
MYSCRATCH = os.getenv('DELETE30', '.')
myfile = os.path.join(MYSCRATCH,'lastname_f','datafile.dat')
with open(myfile, 'w') as f:
    f.write('line in file')
```
in R
```
#! /usr/bin/env Rscript
MYSCRATCH <- Sys.getenv('DELETE30')
MYSCRATCH[is.na(MYSCRATCH)] <- '.'​
save('line in file', file=paste0(MYSCRATCH,'/lastname_f/datafile.dat'))
```

**Note:** lastname_f stands for the last name and the first initial of your PI. If you do not see the folder of your PI please ask `Helpdesk` to create it for you.

## How long will my data stay in monthly scratch?

The data will stay on the file system for 30 days after you have stopped accessing it. 3 days before the data is deleted you (the owner of the files created)  will receive an email with a final warning:
```
From: fs-cleaner.py-no-reply@fhcrc.org [mailto:fs-cleaner.py-no-reply@fhcrc.org]
Sent: Tuesday, August 23, 2016 11:32 PM
To: Doe, Jane <jdoe@fredhutch.org>
Subject: WARNING: In 3 days will delete files in /fh/scratch/delete30!

This is a notification message from fs-cleaner.py, Please review the following message:

Please see attached list of files!

The files listed in the attached text file will be deleted in 3 days when they will not have been touched for 30 days:

# of files: 247, total space: 807 GB
You can prevent deletion of these files
by using the command 'touch -a filename'
on each file. This will reset the access time of the file to the current date.
```

As an alternative to the environment variable $DELETE30 you can also reach scratch through the file system at `/fh/scratch/delete30`, however the file system may be subject to change whereas the environment variable will be supported forever.


## Examples
Python example.
```
#! /usr/bin/env python3
import os

print("Network Job Scratch: %s" % (os.environ['SCRATCH']))
print("Node Local Job Scratch: %s" % (os.environ['SCRATCH_LOCAL']))
print("Node Local Job Scratch: %s" % (os.environ['TMPDIR']))
```

Bash shell example.

```
#! /bin/bash
echo -e $TMPDIR
echo -e "Network Job Scratch:​ $SCRATCH"
echo -e "Node Local Job Scratch: $SCRATCH_LOCAL"
echo -e "Node Local Job Scratch: $TMPDIR"

cd $TMPDIR
cp $HOME/some_file.txt .
file ./some_file.txt
wc -l ./some_file.txt > line_count.txt
cp ./line_count.txt $HOME

exit 0   # $TMPDIR/some_file.txt is removed along with $TMPDIR
```