# SQL Constraints

Constraints are used to **specify the rules concerning data in a table**. They are applied to one or more fields in the creation of a table or altering a table. 
- NOT NULL - Restricts NULL value from being inserted into a column;
- CHECK - Verifies that all values in a field satisfy a condition;
- DEFAULT - Automatically assigns a default value if no value has been specified for the field;
- UNIQUE - Ensures unique values to be inserted into the field; helps identifying each row uniquely; there can be multiple unique constraints defined per table; 
- INDEX - Indexes a field providing faster retrieval of records;
- PRIMARY KEY - Uniquely identifies each record in a table; It must contain UNIQUE values and has an implicit NOT NULL constraint; a table is restricted to have only one primary key;
- FOREIGN KEY - Ensures referential integrity for a record in another table; single or a collection of fields in a table that refers to the primary key in another table.
