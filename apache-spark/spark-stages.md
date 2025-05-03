# Spark stages

When an action is called, for example `countByValue()`, an action plan is created from RDDs we defined, so that Spark can figure out how to parallelise the operation. 
To do that, the job is divided into stages, based on how the data needs to be reorganised.
Then each stage is divided into tasks, distributed across the cluster in the way that each task will be executed by an executor.

### An example
**Stage1** : this two operations do not need to reorganise the data, so are in the same step:
1. `sc.textFile()`
2. `map()`

**Stage2** : here the data needs to be shuffled so that a new stage is created:
1. `countByValue()`
