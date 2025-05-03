# Dataframe
**A Dataframe is just a Dataset of Row objects**, it's literally an alias. **A Row is a generic untyped JVM object**.

## Dataset VS Dataframe
A Row can contains anything, it has **not a specific schema or a structure known at compile-time**: with a Dataframe type errors or wrong column names are known only at run-time; while a Dataset knows the schema at compile-time understanding the type of a column and what can contain.
Therefore, using a Dataset allow also to operate optimization at compile-time instead to wait until it's running. 

Datasets allow us to use compiled programming languages, such as Java and Scala. Python is limited to use Dataframe because it is typed at runtime, losing compile tinme optimization.

## Schema Inference
When reading a Dataframe the **schema can be infered (inferSchema option)** or enforce the schema by expliciting and using it during the read operation.
When inferSchema is set to `true` during the read operation of particular data it has to scan the records to infer the data types out of it.
Whereas, **explicitly defining the schema** it does not have to handle the schema inference taking less time. By explicinting the schema, for example if there is a column orderId you specify its type using IntegerType.
Obviously expliciting the schema has better performance instead of making Spark infer the schema.

# Benefits of Using Explicit Schema with Spark DataFrames

Creating a DataFrame with an explicit schema offers several significant advantages in Apache Spark. Here are the main reasons why it's recommended:

## 1. Better Performance

When you use automatic schema inference (`inferSchema=true`), Spark must:
- Read a sample of the data (by default, it examines the first rows)
- Analyze each value to determine its type
- Build the schema based on this analysis

This process requires time and resources, especially on large datasets. With an explicit schema, Spark knows exactly what to expect and can skip this analysis phase.

## 2. Precise Control Over Data Types

Automatic inference might not always choose the optimal data type:
- It might infer a more generic type than necessary (e.g., `double` instead of `integer`)
- It might misclassify a column (e.g., interpret a numeric ID as a numeric value to calculate with)
- If extreme values are missing in the sample data, it might choose a type not suitable for all the data

With an explicit schema, you can precisely define the appropriate data type for each column.

## 3. Better Handling of Null and Missing Values

By specifying a schema, you can define which columns can contain null values (`nullable=true/false`). This is particularly important when:
- You're loading data that might have missing values
- Some columns are mandatory for your application

## 4. Consistency Across Different Data Reads

If you read the same or similar data multiple times:
- Automatic inference might produce slightly different schemas based on the examined sample
- An explicit schema ensures that data is always interpreted in the same way

## 5. Documentation and Code Readability

The explicit schema definition also serves as documentation:
- It makes clear which columns are expected
- It documents the expected data types
- It helps other developers understand the data structure

## 6. Validation of Input Data

An explicit schema acts as a form of validation:
- If the data doesn't comply with the schema, Spark will generate clear errors
- It helps identify problems in input data quickly

## 7. Better Performance in Joins and Complex Operations

Having the correct data types is essential for operations like joins and aggregations:
- Joins on columns with incompatible types can fail or produce unexpected results
- Numeric aggregation operations work better with appropriate data types

## Practical Example

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DateType, DoubleType

# Define an explicit schema
schema = StructType([
    StructField("order_id", StringType(), False),  # ID as string, not nullable
    StructField("customer_id", IntegerType(), False),  # Numeric ID, not nullable
    StructField("product_id", StringType(), False),  # Product ID as string
    StructField("quantity", IntegerType(), True),  # Quantity as integer, nullable
    StructField("price", DoubleType(), True),  # Price as double, nullable
    StructField("order_date", DateType(), True)  # Date as date, nullable
])

# Read data with explicit schema
df = spark.read.format("csv") \
    .option("header", "true") \
    .schema(schema) \
    .load("path_to_csv_file")
```

In summary, explicitly defining the schema is a best practice in Spark that improves performance, code robustness, and readability, especially for production applications and large datasets.