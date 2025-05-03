# Apache Spark Resource Managers

## YARN

### Understanding spark.yarn.executor.memoryOverhead
The `spark.yarn.executor.memoryOverhead=<num-of-giga>g` configuration parameter is a critical setting when running Spark on YARN.
It represents additional memory that YARN allocates to each executor container beyond the main JVM heap memory (set with --executor-memory or spark.executor.memory).

This overhead memory accommodates:
- Off-heap memory usage by Spark
- Native libraries loaded by the executor
- JVM overheads (metadata, thread stacks)
- Python processes in PySpark
- Temporary files and buffers created during execution

When you specify `--executor-memory 8g --conf spark.yarn.executor.memoryOverhead=2g` YARN will allocate 10GB total to each executor container: 8GB for the JVM heap and 2GB for overhead. \\
If not specified, Spark sets this value to either: 10% of executor memory or 384MB.

Common scenarios requiring more overhead:
- PySpark applications (Python processes need additional memory)
- Applications using native libraries
- Operations creating many intermediate objects outside the JVM heap
- Large shuffle operations
- UDFs, especially in Python
