# RDD (Resilient Distributed Dataset)
An RDD is an **immutable collection of rows of data** and can be **distributed on the computers of a cluster**, allowing then parallel computing over this distributed dataset. 
The **resilient** part is because Spark makes sure that **the processing of an RDD is done even if node failures occur** and it’s up to Spark to figure out how to do that (through the RDD lineage).

RDD are at the core of the Apache Spark functionalities. With **RDDs you are at low level APIs**, you **specify how to do the computation** rather then specify what to do. This in contrast with the Dataset from SQL APIs, wich are based on RDDs.

A driver program create the spark context that is necessary to create and operate with RDDs. The spark context is in charge to make RDDs distributed and resilient. 
Thanks to the spark context, we do not need to write code to handle the distribution of data or the resilience of the dataset. 

Consider these scenarios or common use cases for using RDDs when:
- **you want low-level transformation and actions and control on your dataset**;
- your data is unstructured, such as media streams or streams of text;
- you want to manipulate your data with functional programming constructs than domain specific expressions;
- you can forgot some optimization and performance benefits available with DataFrames and Datasets for structured and semi-structured data

## RDD Operations
There are two types of operations possible on RDDs:
- **Transformation** (map, filter, etc…): apply a function to every row of a dataset; this type of operations work in parallel on distributed data, **applying a function on every row in different chunks of data**, in different nodes of the cluster;
    - **narrow transformations** (e.g., map, filter) have one-to-tone dependencies between partitions;
    - **wide transformations** (e.g., groupByKey, reduceByKey, join) may **involve shuffling** and have dependencies across multiple partitions, that is exchange of data between partitions; the **number of wide transformations defines the number of stages that we have in our DAG**.
- **Action** (count, write, etc…): calling an action on the data trigger the actual computation and return the result to the driver program or write data to an external storage system; we can say that **an action cause the RDD to collapse and give back a result to the driver program** (or writing to a storage).

## groupByKy VS reduceByKey
One of the most common operations in Spark is grouping data by a key and performing some aggregate function on the grouped data. Two commonly used methods for this operation in Spark are reduceByKey() and groupByKey().
groupByKey().

The **groupByKey()** method groups the data by key and returns a PairRDD where each key is associated with a list of values. This operation is useful when we want to group data based on the key, but do not want to perform any aggregate function on the values.

Here is an example of using groupByKey() to count the number of occurrences of each word in a text file:

``` python
text_file = sc.textFile("file.txt")
word_counts = text_file.flatMap(lambda line: line.split(" ")).map(lambda word:word, 1)).groupByKey().mapValues(lambda values: sum(values))
```

In this example, we first split the lines of the text file into words using flatMap(). Then, we map each word to a tuple (word, 1). Finally, we use groupByKey() to group the data by word and mapValues() to compute the sum of the values (i.e., count the number of occurrences of each word).

**Pros and Cons of groupByKey()** \
The advantage of groupByKey() is that it is a simple operation that does not require any complex logic. However, it can be inefficient for large datasets, because all the values associated with each key are shuffled across the network and stored in memory on the worker nodes. This can lead to high memory usage and slow performance.

The **reduceByKey()** method is similar to groupByKey(), but it combines the values for each key using a reduce function. This operation is useful when we want to perform some aggregate function on the values associated with each key.

Here is an example of using reduceByKey() to count the number of occurrences of each word in a text file:

```python
text_file = sc.textFile("file.txt")
word_counts = text_file.flatMap(lambda line: line.split(" ")).map(lambda word: (word, 1)).reduceByKey(lambda x, y: x + y)
```

In this example, we use flatMap() and map() to transform the text file into a PairRDD where each word is associated with the value 1. Then, we use reduceByKey() with a reduce function that adds the values for each key.

**Pros and Cons of reduceByKey()** \
The advantage of reduceByKey() is that it is more efficient than groupByKey() for large datasets, because **it reduces the amount of data that needs to be shuffled and stored in memory computing firsts locally**. However, it requires a **reduce function that is associative and commutative**, which may not always be the case for all aggregate functions.

So, if reduceByKey is so efficient then why groupByKey still exists in Spark? Below are the reason:
- **Simplicity**: groupByKey() is a simple operation that groups data by key and returns a PairRDD where each key is associated with a list of values. It does not require a reduce function and is easy to understand and use.
- **Flexibility**: groupByKey() can be used in situations where we do not want to perform any aggregate function on the values associated with each key. For example, if we want to group a dataset by customer ID and then analyze the purchases made by each customer, we can use groupByKey() to group the data by customer ID and then perform further operations on the resulting PairRDD.
- **Small data**: For small datasets, the difference in performance between reduceByKey() and groupByKey() may not be significant. In fact, for datasets where the number of keys is small and the values associated with each key are relatively small, groupByKey() may even be faster than reduceByKey() due to the overhead of serialization and deserialization in the reduce function.
- **Non-associative operations**: groupByKey() can be used for non-associative operations, where the order of application of the operation matters. For example, if we want to calculate the median of a set of values for each key, we cannot use reduceByKey(), since median is not an associative operation. In this case, we can use groupByKey() to group the data by key and then apply a custom function to calculate the median for each key.

In summary, while reduceByKey() is generally more efficient than groupByKey(), there are still situations where groupByKey() may be a better choice due to its simplicity, flexibility, and applicability to non-associative operations. It is important to understand the characteristics of the dataset and the requirements of the operation when choosing between these two methods.

https://www.linkedin.com/pulse/apache-spark-difference-between-reducebykey-abhijit-sarkhel