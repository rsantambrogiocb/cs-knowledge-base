# Apache Spark Introduction

## The genesis of big data and distributed computing
Traditional storage systems (RDBMS) or imperative ways of programming did not satisfied the scale at which Google wanted to build and search internet's indexed documents.\
This necessity leads to Google Filesystem (GFS), MapReduce and Bigtable:
- GFS is a fault-tollerant and distributed file system across many commodity hardware servers in a cluster farm
- Big Table offer scalable storage of structured data across GFS
- MapReduce introduced a new parallel programming paradigm based on functional programming to process data at scale

### MapReduce in brief
MapReduce applications interact with MapReduce systems that sends computation (mappers and reducers) code to where the data resides, favoring data locality rather than bringing data to your application. The workers aggregate and reduce intermediate computations and produce a final append output from reduce function, which is then written to a distributed storage where it is accessible to your application. This keep most of I/O local to disk rather than distributing it over the network.

## Introduction
The idea is to write a script that describes how to transform or analyse a huge amount of data.
Then Spark will figure out how to distribute the computation across an entire cluster.

The key feature is the scalability: you can write one driver program that tells to Spark what you want to do with your data. Is a Spark problem to figure out how to parallelise and make the job scale out on an entire cluster of computer. 
Doing so, you can distribute the processing of a huge amount of data, not just on a single machine but on a fleet of computer machines at the same time. The idea is divide and conquer to process a big dataset.

We need a system that orchestrate the cluster of computers: it thinks about how to spin up the resources, how to distribute the work or also “how do I run the code in the place where the data is most accessibile”.
For example, if we’re using an Hadoop cluster, we’re using Yarn Cluster Manager.
Spark has also its own cluster manager (standalone environment).

On individual machines of the cluster there are different nodes, that are running different executors. Every executor process has its own cache and its own tasks that it’s trying to operate on the data. 

The driver program sands out the commands to the cluster manager (when needed also directly to the executor). Then the cluster manager talks to executors to orchestrate what gets run where and collecting the results back together. The executors talk to each other to synchronise. 

Spark become so popular and substituted Hadoop thanks to it’s faster processing of big data, capacity given by running the computation in memory. But also reading data from disk is approximately 10x faster.

The faster processing it’s because the DAG engine: it’s going to look at the workflow described in the driver program and it optimise the flow automatically.
In contrast, in MapReduce you’re kind of wedged into a single way of thinking of processing the data: you have to explicitly map all the data in parallel and then define some way of reducing that data. 
A DAG engine is more flexible and can organize the flow in a more complex way. A DAG allows Spark to find the most efficient order to run the tasks needed to accomplish your desired output.
Although Spark is in most cases a better alternative to Hadoop's MapReduce component, Spark can run on top of a Hadoop cluster, taking advantage of its distributed file system and YARN cluster manager. Spark also has its own built-in cluster manager allowing you to use it outside of Hadoop, but it is not a complete replacement for Hadoop. Spark and Hadoop can co-exist.

It’s flexible and easy: you can write the code in python, scala or java. 
If you know SQL, Spark provides Datasets and DataFrames that operate very similarly to SQL statements. You can also specify the SQL directly using the SparkSQL API. 
Not everything is a SQL problem or can be defined through SQL commands, Spark makes available also lower level API, that are Resilient Distributed Datasets (using them you have more flexibility and find better performances for particular use cases).