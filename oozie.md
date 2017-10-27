---
layout: page
title: Oozie Brief Overview
permalink: /oozie/
---

# Oozie Brief Overview

Here is quick review of main entities and ideas.

Nowadays any calculations and processing made on a Hadoop (and not only) cluster
can be viewed as procesing batches of data. The size of the batches is
significant. As batches go smaller, their arrive time becomes smaller and at
some point we process such a data at almost online speed. So at some place small
portion of data appears, and as it is small it can be transferred to processing
point quickly, thus processing can be held immediately.

The other way to process big data is to wait for the batch to become large
enough and run a heavy processing unit once per each large batch. Since we need
to wait for the batch to become quite large, processing occurs not so
frequently, several times per day for example.

Oozie is a tool to process such large batches of data and stands for batch
processing workflow engine.

Generally when doing batch processing (e.g. calculating daily avergages or sums
of some metrics) you need to run the processing unit at a certain moment every
day but only if the needed data actually appeared in a cluster. Importing the
data into the cluster is itself may be a batch processing job.

Oozie solves this task by introducing coordinators and data dependencies. A
coordinator is a text file in which you describe at what time to run
a processing unit and what data will be needed for it to run. The processing
unit itself is a workflow, which is also described in a text file, and is a
sequence of large steps to take (each step may be a Spark or Hive script for
example).

This is just a brief overview of Oozie, please refer to the official docs to get
into details. It's a great tool to learn.
