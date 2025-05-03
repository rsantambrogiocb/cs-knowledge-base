# Partitioning

Every RDD has a number of partitions that determine the degree of parallelism to use when executing an operation. Spark will always try to infer a number of partitions based on cluster size and configured parameters (there is a default parallelism property that can be changed).

Sometimes Spark might not know how to distribute the data in the best way. Spark programs can choose to controll RDDs partitioning to reduce comunincation between nodes. Giving some hint to Spark about how to distribute the data makes sure we don’t run out of resources by asking a single executor to do more it can do. 

**Partitioning is available on all RDDs of key/value pairs.**

Spark does not give explicit control of which worker node each key goes: it lets the program to ensure that each key will appear together on the same node.\
For example, you can chose to partition an RDD into 100 partitions so that keys that has the **same hash** modulo 100 appear on the same node. You can also **range-partition** the RDD into sorted ranges of keys so that keys in the same range appear on the same node. 

It is **useful only when a dataset is reused multiple times in key-oriented operations**. A classic example is when you have a big dataset that needs to be joined with other smaller datasets: if you partition the big RDD then, every time it is used for a join the data is already partitioned.

After applying the partition function, spark will know that the RDD is partitioned and how it is partitioned: this makes **joins with the partitioned RDD to take advantage of this, shuffling only the other dataset**. In the end, resulting in a lot less data shared over the network.
The **partitioned RDD must be persisted after the partition transformation**, otherwise subsequent use of the RDD will cause the reevaluation of the RDD lineage (partitioning).

The **partition number specified control how many parallel tasks perform further operations** on the RDD: if I have fewer partitions than the number of executors that gonna leave some executors in idle; too many partitions will results in too much overhead from shuffling all the data around.
As a general rule, we want at least as many partitions as you have cores or executors that fit with your available memory of the cluster.

Is important to note that **transformations like map cause the RDD to forget the partitioning**, because the key for each record can be theoretically modified mapping the RDD elements. For this, the number of partitions does not change after filtering, and if you don't repartition you'll have way too many memory partitions.

There are different **operations that result in an RDD with partitioning information**, range-partitioned or hash-partitioned RDD.
Basically all **operations that work on key-value data:**
- Operations that act on a single RDD, such as reduceByKey(), running on a pre-partitioned RDD will cause all the values, for each key, to be computed locally on a single machine. 
- Binary operations, such as cogroup() and join(), pre-partitioning will cause at least one of the RDDs to not be shuffled; if both RDDs have the same partitioner, and they are cached on the same machine (one RDD is created with mapValues from the other), or one RDD is not yet computed, then no shuffling across network will occur.

```
A list of all operations that benefit from partitioning are cogroup(), groupWith(), join(), leftOuterJoin(), rightOuterJoin(), groupByKey(), reduceByKey(), combineByKey(), lookup().
```

## Manage partition count
So how do I up-scale (increase) or de-scale (decrease) the number of partitions?

For **increase the number of partition we will use repartition**: it will do a shuffle by internal while increasing the partition number; repartition creates new partitions and does a full shuffle, resulting in evenly distributed data (roughly equal sized partitions). 

For **reduce the number of partition we use coalesce**: it will move data in a smaller number of partition, and do not perform reshuffling operations being less costly; coalesce uses existing partitions to minimize the amount of data that's shuffled, but can result in partitions with different amounts of data.

**Can I not reduce using the repartition?** We can reduce using the repartition but it will unnecessarily do the shuffling, so it is more expensive on operation.

# Partition Monitoring in Spark

It's important to monitor the size and distribution of partitions to identify balancing issues. Highly skewed partitions can indicate design problems in the application or partitioning strategy. Here's how to monitor partitions:

## Using the Spark UI

The Spark UI offers several tools for monitoring partitions:

1. **Jobs Tab**:
   - Access the Spark UI (usually available at `http://[driver-node]:4040`)
   - Select a specific job and click on it to view details
   - Examine the stages within the job

2. **Stages Tab**:
   - This is the most useful section for analyzing partitions
   - Click on a specific stage to view details
   - Look for the "Summary Metrics" table showing statistics for tasks
   - Check the columns:
     - **Input Size / Records**: Shows how much data each task processed
     - **Shuffle Read Size / Records**: Shows how much data each task read during shuffle
     - **Shuffle Write Size / Records**: Shows how much data each task wrote during shuffle
     - **Duration**: The execution time for each task

3. **Shuffle Events**:
   - During operations like `join`, `groupByKey`, etc., you can see shuffle metrics
   - Large disparities in shuffle sizes indicate skewed partitions

4. **Execution Graphs**:
   - In the detailed stage view, check the "Event Timeline" graph
   - Tasks that are much longer than others (outliers) often indicate larger partitions

5. **Detailed Task View**:
   - Click on "Task Details" in a stage to see the exact size of each partition
   - Sort by "Input Size" or "Shuffle Write Size" to quickly identify anomalous partitions

## Using Additional Tools

Beyond the Spark UI, you can use:

1. **Programmatic Metrics**:
   ```scala
   // To check partition sizes
   rdd.glom().map(arr => arr.length).collect().foreach(println)
   
   // To count elements per partition
   rdd.mapPartitionsWithIndex((index, iter) => 
     Iterator((index, iter.size))).collect()
   ```

2. **Ganglia or Grafana**: For monitoring cluster metrics in real-time

3. **Flame Graphs**: To identify bottlenecks at the task level

## How to Identify Skewed Partitions

Partitions are considered skewed when:

1. **High variance in size**: Some partitions are significantly larger than others (e.g., 10x or more)
   
2. **Uneven workload distribution**: Check if some partitions take much longer to process
   
3. **Statistical indicators**:
   - Check the ratio between the largest and smallest partition (should be < 3)
   - Verify the standard deviation of partition sizes
   
4. **Visual patterns in the UI**:
   - "Straggler effect" in execution graphs (some tasks much longer)
   - Specific executors using much more memory than others

## Correcting Skewed Partitions

If you identify skewed partitions:

1. Consider **salting techniques** for frequent keys
2. Implement a **custom partitioner** with better distribution
3. Increase the number of partitions to better distribute data
4. Use **pre-aggregation techniques** to reduce data size before shuffle operations


## Resources to integrate
- https://tsaiprabhanj.medium.com/spark-repartition-vs-coalesce-vs-partitionby-f0d50c0c70cf
- https://medium.com/datalex/on-spark-performance-and-partitioning-strategies-72992bbbf150
- https://stackoverflow.com/a/42780452