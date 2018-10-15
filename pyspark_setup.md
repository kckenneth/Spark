# pyspark in Jupyter Notebook

Install findspark library for python and launch jupyter-notebook.
```
# pip install findspark
# jupyter-notebook
```
Go to your browser, 
```
50.97.252.101:8123/?tokenxxxxx
```

In a new ipyn notebook, you must begin with the following libraries.
```
import findspark
findspark.init()
import pyspark
import random
from pyspark.sql import SparkSession
spark = SparkSession.builder.getOrCreate()
```

