# Data Skew Issues in Apache Spark

## Table of Contents
- [What is Data Skew?](#what-is-data-skew)
- [Types of Data Skew Issues](#types-of-data-skew-issues)
- [Identifying Data Skew](#identifying-data-skew)
- [Solutions for Data Skew](#solutions-for-data-skew)
  - [General Approaches](#general-approaches)
  - [The Salting Technique](#the-salting-technique)
  - [Split and Join Strategy](#split-and-join-strategy)
  - [Pre-aggregation for Hot Keys](#pre-aggregation-for-hot-keys)
- [Configuration Optimizations](#configuration-optimizations)
- [Case Study: Fixing a Skewed Join](#case-study-fixing-a-skewed-join)

## What is Data Skew?

Data skew occurs when data is unevenly distributed across partitions in Spark, causing some tasks to process significantly more data than others. This imbalance leads to several issues:

- **Stragglers**: Some tasks take much longer to complete, delaying the entire job
- **Resource inefficiency**: Most executors finish quickly and sit idle while a few struggle
- **Failures**: Overloaded tasks may run out of memory or exceed time limits
- **Poor scalability**: Adding more executors doesn't help when bottlenecked by a few tasks

## Types of Data Skew Issues

### 1. Uneven Data Distribution

This occurs when the original data partitioning is unbalanced, often due to:
- Natural data characteristics (e.g., geographic clustering)
- Poor partitioning strategy
- Non-uniform data generation patterns

### 2. Skewed Joins

Join operations between datasets where certain join keys are much more frequent than others. Common examples:
- Customer ID joins where some customers have vastly more records
- Date-based joins where certain dates (e.g., holidays) have more activity
- Category joins where popular categories dominate

### 3. Hot Keys in Aggregations

Similar to skewed joins, certain keys in groupBy operations may have far more records, causing imbalanced workloads during reduction phases.

## Identifying Data Skew

Look for these indicators:

**In Spark UI:**
- Stage timeline shows most tasks completing quickly with a few long-running tasks
- Shuffle read/write sizes vary dramatically between tasks
- Specific tasks have excessive spill to disk

**In logs:**
```
INFO TaskSetManager: Finished task 15.0 in stage 2.0 (TID 42) in 12 seconds
INFO TaskSetManager: Finished task 16.0 in stage 2.0 (TID 43) in 11 seconds
...
INFO TaskSetManager: Finished task 149.0 in stage 2.0 (TID 176) in 735 seconds
```

**Using code to inspect:**

```python
# Check key distribution
key_distribution = df.groupBy("join_key") \
                      .count() \
                      .orderBy("count", ascending=False)
                      
# Show top 20 most frequent keys                    
key_distribution.show(20)

# Show distribution statistics
from pyspark.sql.functions import expr
df.groupBy().agg(
    expr("percentile_approx(count, array(0.5, 0.9, 0.95, 0.99), 10000)").alias("percentiles")
).show()

# Count records per partition
def count_per_partition(df):
    return df.rdd.mapPartitions(lambda it: [sum(1 for _ in it)]).collect()
    
partition_counts = count_per_partition(df)
print(f"Min: {min(partition_counts)}, Max: {max(partition_counts)}, " 
      f"Avg: {sum(partition_counts)/len(partition_counts)}")
print(f"Max/Avg ratio: {max(partition_counts)/(sum(partition_counts)/len(partition_counts))}")
```

## Solutions for Data Skew

### General Approaches

#### 1. Increase Partitions

Repartition data to distribute work more evenly:

```python
# Increase partition count
df = df.repartition(500)  # Adjust number based on data size
```

#### 2. Use Custom Partitioning

```python
# Repartition by specific columns for better distribution
df = df.repartition(col("date"), col("region"))
```

#### 3. Broadcast Smaller Dataframes

```python
from pyspark.sql.functions import broadcast

# Broadcast the smaller dataframe to avoid shuffle
result = large_df.join(broadcast(small_df), "join_key")
```

#### 4. Use Spark's Adaptive Query Execution

In Spark 3.x:

```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
```

### The Salting Technique

The salting technique is a powerful strategy for **handling severely skewed data** by artificially distributing hot keys across multiple partitions.

#### How Salting Works

1. Add random values ("salts") to each record's join key
2. Create composite keys that combine the original key with the salt
3. Ensure both sides of joins use the same salting strategy
4. This distributes processing of hot keys across multiple tasks

#### Implementing Salting

```python
from pyspark.sql.functions import rand, concat, lit, explode, array

def salt_dataframe(df, key_col, num_salts=10):
    """Add random salt to a fact table"""
    return df.withColumn("salt", (rand() * num_salts).cast("int")) \
             .withColumn("salted_key", concat(col(key_col), lit("_"), col("salt")))

def salt_dimension_table(df, key_col, num_salts=10):
    """Create multiple copies of dimension table rows, one for each salt value"""
    # Create array of possible salt values
    salt_values = list(range(num_salts))
    
    # Explode the dataframe to create one row per salt value
    return df.withColumn("salt_values", array([lit(i) for i in salt_values])) \
             .withColumn("salt", explode("salt_values")) \
             .withColumn("salted_key", concat(col(key_col), lit("_"), col("salt"))) \
             .drop("salt_values")

# Usage example for joining a fact table with a dimension table
salted_facts = salt_dataframe(fact_df, "customer_id", 10)
salted_dimension = salt_dimension_table(dim_df, "customer_id", 10)

# Join using salted keys
result = salted_facts.join(salted_dimension, "salted_key")

# Drop the extra columns if needed
final_result = result.drop("salt", "salted_key")
```

#### Visual Example of Salting

**Original Data:**

Fact Table (Orders):
```
customer_id | order_id
-----------+----------
    1      |    101
    2      |    102
    3      |    103
    1      |    104
    1      |    105
```

Dimension Table (Customers):
```
customer_id | name
-----------+-------
    1      | Alice
    2      | Bob
    3      | Carol
```

**After Salting with 2 salts:**

Salted Fact Table:
```
customer_id | order_id | salt | salted_key
-----------+----------+------+-----------
    1      |    101   |   0  |    1_0
    2      |    102   |   1  |    2_1
    3      |    103   |   0  |    3_0
    1      |    104   |   1  |    1_1
    1      |    105   |   0  |    1_0
```

Salted Dimension Table:
```
customer_id | name  | salt | salted_key
-----------+-------+------+-----------
    1      | Alice |   0  |    1_0
    1      | Alice |   1  |    1_1
    2      | Bob   |   0  |    2_0
    2      | Bob   |   1  |    2_1
    3      | Carol |   0  |    3_0
    3      | Carol |   1  |    3_1
```

The hot key (customer_id=1) is now distributed across multiple partitions.

### Split and Join Strategy

For more complex scenarios, use a hybrid approach:

```python
# Identify skewed keys
key_counts = df.groupBy("join_key").count().cache()
skewed_keys = key_counts.filter(col("count") > 10000).select("join_key").collect()
skewed_keys_list = [row.join_key for row in skewed_keys]

# Split dataframes into skewed and non-skewed parts
df1_skewed = df1.filter(col("join_key").isin(skewed_keys_list))
df1_normal = df1.filter(~col("join_key").isin(skewed_keys_list))
df2_skewed = df2.filter(col("join_key").isin(skewed_keys_list))
df2_normal = df2.filter(~col("join_key").isin(skewed_keys_list))

# Handle skewed part with salting
df1_skewed = salt_dataframe(df1_skewed, "join_key", 20)
df2_skewed = salt_dimension_table(df2_skewed, "join_key", 20)

# Process normally distributed data normally
normal_join = df1_normal.join(df2_normal, "join_key")

# Process skewed data with salting
skewed_join = df1_skewed.join(df2_skewed, "salted_key")

# Combine results
result = normal_join.union(skewed_join.drop("salt", "salted_key"))
```

### Pre-aggregation for Hot Keys

For aggregation operations:

```python
from pyspark.sql.functions import sum

# First identify hot keys
key_counts = df.groupBy("key").count()
hot_keys = key_counts.filter(col("count") > 10000).select("key").collect()
hot_keys_list = [row.key for row in hot_keys]

# Split data
hot_data = df.filter(col("key").isin(hot_keys_list))
normal_data = df.filter(~col("key").isin(hot_keys_list))

# Pre-aggregate hot keys locally before global aggregation
hot_agg = hot_data.repartition(100, col("key")) \
                  .groupBy("key") \
                  .agg(sum("value").alias("partial_sum"))

# Normal aggregation for well-distributed keys
normal_agg = normal_data.groupBy("key") \
                        .agg(sum("value").alias("partial_sum"))

# Combine results
final_result = hot_agg.union(normal_agg) \
                      .groupBy("key") \
                      .agg(sum("partial_sum").alias("total_sum"))
```

## Configuration Optimizations

Beyond code solutions, consider these configuration settings:

```python
# Enable adaptive query execution (Spark 3.x)
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")

# Set skew join detection threshold 
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionFactor", "5")  # Default is 5
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes", "256MB")  # Default is 256MB

# Increase shuffle partitions for better distribution
spark.conf.set("spark.sql.shuffle.partitions", "1000")  # Default is 200

# Optimize memory usage
spark.conf.set("spark.memory.fraction", "0.8")  # Default is 0.6
```

## Case Study: Fixing a Skewed Join

Consider a real-world example:

**Problem:** Customer activity analysis joining a 1TB transaction table with a customer dimension table. The join is heavily skewed due to a few major customers having millions of transactions.

**Diagnosis:**
1. Identified through Spark UI that 5% of tasks were taking 95% of the total execution time
2. Analyzed key distribution:
   ```python
   transaction_df.groupBy("customer_id").count().orderBy("count", ascending=False).show(10)
   ```
   Showed that the top customer had 58 million transactions while the median was only 12

**Solution:**
1. Identified the top 100 customers by volume
2. Applied salting with 50 partitions to those customers
3. Processed the remaining customers normally
4. Used the split-and-join pattern

**Results:**
- Job execution time reduced from 4 hours to 17 minutes
- No more executor OOM errors
- Reduced cluster size requirements by 40%

**Key Takeaways:**
1. Identify data skew early in the development process
2. Analyze key distributions before expensive operations
3. Apply targeted solutions to only the problematic keys
4. Monitor improvements with detailed metrics
