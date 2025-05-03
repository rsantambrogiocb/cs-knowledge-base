# The laziness of Spark
Spark works with the concept of the lazy evaluation: nothing happens in your driver program until an action is called. 

Until Spark knows what you are trying to achieve with the action you called, it does not know how to optimize the operations you done on the dataset. 
In this way, Spark build the DAG optimized for the action you are trying to do. 
This can be confusing during the debug process: you may need to put a temporary action to debug a specific section of the code. 