# Indexes
An index is a data structure to provide quick lookup in a column (or multiple columns) of a database table. It **enhances the retrieving operation at the cost of additional writes** and memory management to **handle the index data structur**e.

### Unique index
The **unique index ensures that rows of a table have no duplicates for a given key**. Once a unique index has been defined for a table, uniqueness is enforced whenever keys are added or changed within the index.

### Non unique index
Non-unique indexes are **used solely to improve query performance by maintaining a sorted order of data values that are used frequently**.

### Clustered index
**Can exists only one clustered index per table**. They are indexes whose **order of the rows in the database corresponds to the order of the rows in the index**.
These type of index **modifies the way records are stored in a database based on the indexed column**, while a *non-clustered index creates a separate entity within the table which references the original table*.
Clustered index are used mainly to speed up the retrieval of data.