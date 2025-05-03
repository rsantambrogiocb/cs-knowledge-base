# Spark lineage graph
The lineage graph is a directed acyclic graph that **represents the sequence of transformations applied to an RDD and the dependencies between RDDs**.
Each node in the lineage graph represents an RDD resulting from a transformation, and edges represent dependencies between RDDs. *If RDD A is transformed to produce RDD B, there is a dependency edge from A to B.*

The lineage graph **enables Spark to achieve fault tolerance through recomputation**. If a partition of an RDD is lost due to a node failure, Spark can recompute that partition by back tracing the lineage graph to the original data source applying the transformations without recomputing the other partitions.

The lineage graph plays a crucial role in Spark’s caching mechanism. **When you persist an RDD, Spark retains the lineage information so that it can avoid recomputation** by reusing the stored partitions when needed.

The lineage graph allows also Spark to **optimize computational flow by rearranging or combining operations before executing them**. Catalyst, Spark’s optimization engine, uses information from the lineage to apply logical and physical optimization techniques such as predicate pushdown, projection pruning, and physical execution planning which includes pipelining transformations.

Spark provides a way for users to view the lineage graph through the Spark UI. The UI’s DAG visualization shows the sequence of transformations that lead to the creation of an RDD, and the stages of the Spark job. Programatically, users can view the lineage using the `toDebugString` method on an RDD.

## The difference between the lineage graph and Spark's DAG
Spark's DAG is basically the final execution plan on the basis of which the entire set of tasks and transformations is executed. The DAG initially shows the sequence of actions that take place in different stages, as well as the various dependencies between stages. While the lineage graph shows the lineage of RDDs, like which RDD comes from which other RDD after a certain set of transformations.

### In-depth exploration of differences between Lineage Graph and DAG
The lineage graph and Spark's DAG are related concepts but with important differences that are essential to understand in order to optimize Spark applications:

#### RDD Lineage Graph

The lineage graph is essentially the genealogical history of RDDs, which precisely documents how a particular RDD was created:

- **Represents transformations**: Tracks which operations (map, filter, join, etc.) were applied to original RDDs to create new RDDs.
- **Fine granularity**: Operates at the level of individual RDDs, showing direct relationships between an RDD and its "parents."
- **Data-oriented**: Focuses on data and how it is transformed.
- **Resilience purpose**: Serves mainly for fault tolerance, allowing Spark to reconstruct a lost RDD by retracing the transformations that generated it.
- **Continuous existence**: Exists from the moment an RDD is created, regardless of the execution of action operations.

For example, if you have:
```python
rdd1 = sc.textFile("data.txt")  # Original RDD
rdd2 = rdd1.map(lambda x: x.split())  # Transformation 1
rdd3 = rdd2.filter(lambda x: len(x) > 5)  # Transformation 2
```

The lineage graph will show rdd1 → rdd2 → rdd3, with the specific transformations connecting each RDD.

#### Spark's DAG (Directed Acyclic Graph)

Spark's DAG, on the other hand, is an optimized execution plan that is created when an action is called:

- **Represents physical execution**: Shows how Spark intends to actually execute the job, including stages and tasks.
- **Execution-oriented**: Focuses on execution efficiency, organizing operations into optimized stages.
- **Created at action time**: Is generated only when an action is called, not when transformations are defined.
- **Includes stages and shuffles**: Divides execution into stages, separated by operations that require shuffling.
- **Optimized by the scheduler**: Spark analyzes the lineage graph and optimizes the execution DAG to minimize data movement and maximize parallelism.

#### Key differences with concrete examples

1. **Creation time**:
   - Lineage graph: Created incrementally during the definition of transformations
   - DAG: Created only when an action is called

2. **Level of abstraction**:
   - Lineage graph: Represents the logical sequence of transformations
   - DAG: Represents the physical execution plan with stages and tasks

3. **Organization**:
   - Lineage graph: Follows exactly the transformations defined in the code
   - DAG: Can reorganize, combine, or optimize transformations

4. **Practical example**:
   ```python
   # Creating RDDs and transformations (only builds the lineage graph)
   rdd1 = sc.textFile("data.txt")
   rdd2 = rdd1.map(lambda x: x.split())
   rdd3 = rdd2.filter(lambda x: len(x) > 0)
   rdd4 = rdd3.flatMap(lambda x: x)
   rdd5 = rdd4.map(lambda x: (x, 1))
   rdd6 = rdd5.reduceByKey(lambda a, b: a + b)
   
   # At this point only the lineage graph exists
   
   # When we call an action, Spark creates and optimizes the execution DAG
   result = rdd6.collect()
   ```

   In this example, Spark might optimize the DAG by combining some of the map/filter transformations into a single stage, and creating a second stage after the reduceByKey (which requires shuffling).

5. **Visualization**:
   - Lineage graph: Visible via `rdd.toDebugString()` and shows the relationship between RDDs
   - DAG: Visible in the Spark UI in the "Jobs" section and shows stages and tasks

#### Practical implications

This difference has important implications in optimizing Spark applications:

1. **Understanding the lineage** allows you to understand how Spark will reconstruct data in case of failures.

2. **Analyzing the DAG** in the UI allows you to identify bottlenecks in execution, such as expensive shuffle operations or unbalanced stages.

3. **Long lineages** can lead to longer recovery times in case of failures, so strategies such as persistence (`cache()` or `persist()`) can be used to "truncate" the lineage at strategic points.

In summary, the lineage graph is the "recipe" that shows how to create an RDD, while the DAG is the optimized "production plan" that Spark uses to actually execute the computation when requested.
