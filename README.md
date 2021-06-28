# spark_basic
This is a documentation of Semantix Academy course of Apache Spark. 

### 1-Preparing the dev environment for spark project
### 2-Projects with Jupyter Notebooks with Python
### 3-RDD operations
### 4-Dataframe operations
### 5-Dataset operations
### 6-IDE Python and Scala
### 7-Struct Streaming KAFKA
### 8-Spark Streaming KAFKA
### 9-Optimizations and Tuning
___

## 1-Preparing the dev environment for spark project
#### -The goal here is to prepare the dev environment by installing Apache Spark and frameworks of Big Data Cluster using Docker and Docker Compose in WSL2.

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
### Using Jupyter Notebook
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

read table juros as Hive table using Dataframe:

```spark.read.table("juros").show()```
---
read table juros in parquet format:

```spark.read.parquet("juros").show()```












