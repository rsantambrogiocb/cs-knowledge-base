# Dataset

RDD is e basic (internal) implementation and Spark SQL is built over it (from Spark version 2).

Dataset is build over the RDD and is the most imporant part of Spark SQL APIs.
A **Dataset is an immutable distributed collecion of data organized into named columns**, like a table in a relational database:

- a Dataset is a **collection of strongly-typed JVM objects**, dictated by a case class you define in Scala or a class in Java; 
- it has an **explicitly type structure, a schema known at compile time>>>**;
    - the schema allow to run SQL query over the data in it;
    - the schema leads also to a more efficient data storage;
- Dataset also brings **logical optimization** that comes thanks to the use of SQL and behaviour of a database
    - in adjunction to the optimization done by Spark DAG, it can allow to optimize the query thanks to SQL optimization, just like a relational database (using a Dataset we'll have more optimization that just RDDs)
- a Dataset can be converted easily to an RDD, it's also possible the contrary, just using their respectively APIs.


### Datasets are often way more efficient that RDDs
- Are **serialized more efficiently thanks to Tungsten encoder** that efficiently serialize/deserialize JVM objects as well as it generates compact bytecode that can be executed very fast;
- Have compile time optimization in adjunction with SQL optimization for the execution plan, while RDDs let this optimization to the developer
- SparkML and Spark Streaming have APIs based on Dataset, so it is convenient to not move every time between RDD and Dataset (just from a point of code quality)
- Many times the operations are simplified thanks to SQL APIs, making Spark usable not only by software engineers, but also for data analysts and scientists.

### Spark Session
To work with Datasets, there is the need to create a SparkSession object, just as in SQL we'll have opened a session to a database. When the work is done, the session must be stopped. Using the SparkSession do not mean that you can't have anymore the SparkContext and RDDs, we can just retrieve them by the SparkSession object.

### Dataset vs DataFrame
In Spark, a **DataFrame is essentially an alias for a Dataset of Row (`Dataset<Row>`)**.
- DataFrames are untyped Datasets, where types are checked only at runtime
- Datasets offer compile-time type checking, reducing errors
- In PySpark, only the DataFrame concept exists because Python is a dynamically typed language

Example of creation in Scala:
```scala
// DataFrame (Dataset<Row>)
val df = spark.read.json("people.json")

// Conversion to typed Dataset
case class Person(name: String, age: Long)
val ds = df.as[Person] // Dataset<Person>
```

### Operations Available on Datasets
Datasets offer two types of APIs for manipulating data:

1. **Relational APIs** (similar to SQL):
   ```scala
   // SQL-like operations
   dataset.select("name", "age")
   dataset.filter($"age" > 21)
   dataset.groupBy("department").avg("salary")
   dataset.join(otherDataset, "departmentId")
   ```

2. **Transformation APIs** (similar to RDD):
   ```scala
   // RDD-like functional operations
   dataset.map(person => person.name.toUpperCase())
   dataset.flatMap(person => person.addresses)
   dataset.filter(person => person.age > 21)
   ```

### Supported Data Formats
Datasets can read and write data in various formats:
- CSV
- JSON
- Parquet (high-performance columnar format, recommended)
- ORC
- Avro
- Plain text
- Hive tables
- JDBC/ODBC

### Complete Example in Scala
```scala
import org.apache.spark.sql.SparkSession

// Creating SparkSession
val spark = SparkSession.builder()
  .appName("Dataset Example")
  .master("local[*]")
  .getOrCreate()

// Schema definition
case class Person(name: String, age: Int, city: String)

// Creating a Dataset from in-memory data
import spark.implicits._
val peopleDS = Seq(
  Person("Alice", 25, "New York"),
  Person("Bob", 30, "San Francisco"),
  Person("Charlie", 35, "Boston")
).toDS()

// Operations on the Dataset
val filteredDS = peopleDS.filter(_.age > 25)
  .map(p => (p.name, p.age * 2))

// Converting to DataFrame for SQL operations
peopleDS.createOrReplaceTempView("people")
val results = spark.sql("SELECT name, city FROM people WHERE age > 25")

// Saving results
filteredDS.write.parquet("output/filtered_people")

// Closing the session
spark.stop()
```

### Limitations and Considerations
- Datasets have a higher initial overhead compared to RDDs for planning
- For very small and simple operations, RDDs might be faster
- Dataset APIs are more limited in non-JVM languages like Python

# Spark SQL other than Datasets
Using Spark SQL allow also to talk to JDBC/ODBC servers, also opening a shell (Hive support).
In Sparl SQL we can also define User Defined Functions (UDF) to be applied to the data.




