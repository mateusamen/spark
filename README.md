# spark_basic
This is a documentation of Semantix Academy course of Apache Spark. 
___

### 1-Preparing the dev environment for spark project
The goal here is to prepare the dev environment by installing Apache Spark and frameworks of Big Data Cluster using Docker and Docker Compose in WSL2.

>Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly.

>Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a Compose file to configure your application's services. Then, using a single command, you create and start all the services from your configuration

First we might install Docker and Docker-Compose acessing those links:

[https://docs.docker.com/get-docker/]

[https://docs.docker.com/compose/install/]


Use the command below in WSL to download the images, and it will automatically create a directory called Spark:

```git clone https://github.com/rodrigo-reboucas/docker-bigdata.git spark```

The output might be:

![01- docker-pull_images](https://user-images.githubusercontent.com/62483710/123144718-5f76b300-d432-11eb-8d57-021241b4a1ab.PNG)

Then, we go to Spark directory using ```cd spark``` command. Once inside the directory we will start and run the cluster Hadoop using

```docker-compose –f docker-compose-parcial.yml up -d```

The output is shown bellow:

![02-docker-up_creating](https://user-images.githubusercontent.com/62483710/123144817-7cab8180-d432-11eb-977f-913c47294b2c.PNG)

Using ```docker-ps``` to list the containers and the output is shown:

![03-docker_ps](https://user-images.githubusercontent.com/62483710/123144951-a8c70280-d432-11eb-968d-6cfc543e2e8f.PNG)

Config Spark Jar to accept Parquet in Hive table:

```curl -O https://repo1.maven.org/maven2/com/twitter/parquet-hadoop-bundle/1.6.0/parquet-hadoop-bundle-1.6.0.jar```

then, copy file to jupiter-spark container and /opt/spark/jars directory.

```docker cp parquet-hadoop-bundle-1.6.0.jar jupyter-spark:/opt/spark/jars```

Go to input directory by ```cd input```

Download data into input directory(namenode volume):

```sudo git clone https://github.com/rodrigo-reboucas/exercises-data.git```

Verify the data in namenode:

```docker exec -it namenode ls /input```

There might have an exercice data file.

Enter namenode container:

```docker exec -it namenode bash```

Check what's inside HDFS it:

```hdfs dfs -ls /```

We may have a hbase, tmp and user dir.

Creating mateus/data directory inside user dir (the command -p is responsible to create a directory tree):

```hdfs dfs -mkdir -p /user/mateus/data ```

Checking what's inside user dir by:

```hdfs dfs -ls -R /user```

![04- docker dir_tree ls](https://user-images.githubusercontent.com/62483710/123521159-5d5b6100-d68b-11eb-9da2-bf00eed400e1.PNG)

Send files from input to user/mateus/data by:

```hdfs dfs -put input/* user/mateus/data```

---
### 2-Using Jupyter Notebook
The port used is localhost:8889
First create a PySpark file. 

![05-pyspark_jupyter](https://user-images.githubusercontent.com/62483710/123691514-18226500-d82c-11eb-8e8c-b699c7d6fce8.PNG)


Get info from SparkContext and SparkSession:

![05-pyspark_sc_session](https://user-images.githubusercontent.com/62483710/123691534-1e184600-d82c-11eb-9e72-440d8203ed22.PNG)

Set LOG = INFO:

```spark.sparkContext.setLogLevel("INFO")```

List DataBases:

```spark.catalog.listDatabases()```

output:

![07-list_database](https://user-images.githubusercontent.com/62483710/123691783-6899c280-d82c-11eb-83ff-8cd9fdbc7b4d.PNG)

Read json file from HDFS

```leitura_dados = spark.read.json(<"path">)```

in this case, the path is: "hdfs://namenode:8020/user/mateus/data/exercises-data/juros_selic/juros_selic.json"

```leitura_dados = spark.read.json("hdfs://namenode:8020/user/mateus/data/exercises-data/juros_selic/juros_selic.json")```

Show data from JSON file read above:

```leitura_dados.show()```

Save Dataframe as Hive Table named juros:

```leitura_dados.write.saveAsTable("juros")```

Check if data was correctly saved, going to WLS, enter namenode container by ```docker exec -it namenode bash```

then:

```hdfs dfs -ls -R /user/hive```

output:

![08_ls_hdfs](https://user-images.githubusercontent.com/62483710/123695608-21fa9700-d831-11eb-977a-3f0f7d1d88cc.PNG)

or we can do straight from Jupyter using ! as shown below:

![10_info_jurostable](https://user-images.githubusercontent.com/62483710/123696579-3f7c3080-d832-11eb-9de5-ba8e0d723b45.PNG)

we can see the format parquet and snappy compression.

list tables with catalog:

```spark.catalog.listTables()```

output:

![09-list_tables](https://user-images.githubusercontent.com/62483710/123696068-b664f980-d831-11eb-8bfc-78f07937052c.PNG)

read table juros as Hive table using Dataframe

```spark.read.table("juros").show()```

read table juros in parquet format:

```spark.read.parquet("juros").show()```

---

### 3-RDD operations

-Acess external dataset from local file system. Use *sparkContext's textFile* method. 

```logs = sc.textFile("file:///opt/spark/logs/")```

-count the number of lines(output must be 30):

```logs.count()```

-read first line: 

```logs.first()```

-Count number of words in logs file(output must be 268):

```pyspark
words = logs.flatMap( lambda line : line.split(" "))
words.count()
```

- change to lowercase all the words in *logs file*, then check the first 30 elements:

```pyspark
lower_case_words = logs.map( lambda lc : lc.lower())
lower_case_words.take(30)
```

- Remove the words with length less and equal to 2:

```pyspark
size_words = words.filter( lambda palavra : len(palavra)<=2)

```

-Count the number of times each word appears:

```pyspark
p_cv = words.map( lambda palavra : (palavra,1))
p_red = p_cv.reduceByKey( lambda key1 , key2 : key1 + key2)
```

-count number of words repeated:

```pyspark
p_red.count()
```

-Sort by the number of times the word appears:

```pyspark
palavras_ordem = p_red.sortBy( lambda palavra : -palavra[1])
```

-Filter by words that appears more than one time:

```pyspark
palavras_bigger1 = palavras_ordem.filter( lambda palavra : palavra[1]>1)
```

output must be something like that:

![11-PALAVRAS_ORDEMgt1](https://user-images.githubusercontent.com/62483710/124358010-ac0d7b80-dbf4-11eb-99e1-18a8c252ca52.PNG)

-save RDD in directory HDFS /user/mateus/logs_count_word

```pyspark
palavras_bigger1.saveAsTextFile("user/mateus/logs_count_word")
```
---

### 4 - Spark Schema
---
##### DataFrame Schema
   4.1 - Create df name_us_sem_schema to read file in HDFS "/user/mateus/data/exercises-data/names"
 - First check file format:

```! hdfs dfs -ls -R /user/mateus/data/exercises-data/names```

output:

![12- 4 1](https://user-images.githubusercontent.com/62483710/124490466-80bb9580-dd88-11eb-9e46-68323ae04526.PNG)


then create df:

```names_us_sem_schema = spark.read.csv("/user/mateus/data/exercises-data/names")```

   4.2- Visualize Schema and show 5 registers

```
names_us_sem_schema.take(5)
```

output:

![13-4 2](https://user-images.githubusercontent.com/62483710/124490588-a8126280-dd88-11eb-9bb7-15056e73ea1f.PNG)


```
names_us_sem_schema.schema
```
output:

![14-4 3](https://user-images.githubusercontent.com/62483710/124490562-a052be00-dd88-11eb-9256-12b6dfedba6a.PNG)


   4.3- Create df name_us to read file in HDFS "/user/mateus/data/exercises-data/names" with following schema:
 - name: String
 - sexo: String
 - qtd: integer

   4.3.1 - first import pyspark.sql.types module

```
from pyspark.sql.types import *
```
   4.3.2 - Create Struct Field and Struct Type

```
list_structure = [
    StructField("nome", StringType()),
    StructField("sexo", StringType()),
    StructField("qtd", IntegerType())
]

names_schema = StructType(list_structure)
```
   4.3.3 - Associate name_us to structType(name_schema)  

```
name_us = spark.read.csv("/user/mateus/data/exercises-data/names",schema=(names_schema))
```

   4.4- Visualize schema and show 5 registers:

```
name_us.schema
```

output:

![15-4 4](https://user-images.githubusercontent.com/62483710/124490632-b95b6f00-dd88-11eb-85ee-a6aa7a9d946a.PNG)


```
name_us.show(5)
```

output:

![16-4 5](https://user-images.githubusercontent.com/62483710/124490672-c5dfc780-dd88-11eb-8196-624337488e07.PNG)


   4.5- Save df name_us with orc format in HDFS "/user/mateus/data/exercises-data/names_us_orc"

```
name_us.write.orc("/user/mateus/data/exercises-data/names_us_orc")
```

   4.6- Check if file was successfully saved:

```
! hdfs dfs -ls /user/mateus/data/exercises-data/names_us_orc
```

output:

![17-4 6](https://user-images.githubusercontent.com/62483710/124490692-cd06d580-dd88-11eb-94c6-f55504f4e310.PNG)

---
##### DataSet Schema

First, using WSL, enter Jupyter-Spark container by:

```docker exec -it jupyter-spark bash```

then, execute ```spark-shell```

output:

![18-spark-shell_dataset](https://user-images.githubusercontent.com/62483710/124503634-5a075a00-dd9c-11eb-80cf-917e697cea46.PNG)

read file in HDFS /user/mateus/data/exercises-data/names

```scala
scala> val names_us = spark.read.csv("/user/mateus/data/exercises-data/names")
```

print schema

```scala
scala> names_us.printSchema
```

show first 5 registers:

```scala
scala> names_us.show(5)
```

output for the 3 steps shown above (read file, print schema and show registers):

![19-dataset_red-1](https://user-images.githubusercontent.com/62483710/124504469-ed8d5a80-dd9d-11eb-8c8d-db489c30be90.PNG)

Create case class nascimento for data in name_us:

```scala
 scala> case class Nascimento(name:String,sex:String,qtd:Int)
```

import Enconders:

```scala
scala> import org.apache.spark.sql.Encoders
```

create schema:

```scala
scala> val schema = Encoders.product[Nascimento].schema
```

create dataset name_ds

```scala
scala> val names_ds = spark.read.schema(schema).csv("/user/mateus/data/exercises-data/names").as[Nascimento]
```

show first 5 registers:

```scala
scala> names_ds.show(5)
```

output:

![20-ds](https://user-images.githubusercontent.com/62483710/124506418-fc760c00-dda1-11eb-9384-1ee7a7f2577d.PNG)

save dataset names_ds parquet format snappy compression:

```scala 
scala> names_ds.write.parquet("/user/mateus/names_us_parquet")
```

---





