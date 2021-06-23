# spark
## spark_basic


### The goal here is to prepare the dev environment by installing Apache Spark and frameworks of Big Data Cluster using Docker and Docker Compose in WSL2.

"Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly."

"Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a Compose file to configure your application's services. Then, using a single command, you create and start all the services from your configuration"

First we might install Docker and Docker-Compose, then use the command below in WSL to download the images, and it will automatically create a directory called Spark:

```git clone https://github.com/rodrigo-reboucas/docker-bigdata.git spark```

The output might be:

![01- docker-pull_images](https://user-images.githubusercontent.com/62483710/123144718-5f76b300-d432-11eb-8d57-021241b4a1ab.PNG)

Then, we go to Spark directory using ```cd spark``` command. Once inside the directory we will start and run the cluster Hadoop using

```docker-compose –f docker-compose-parcial.yml up -d```

The output is shown bellow:

![02-docker-up_creating](https://user-images.githubusercontent.com/62483710/123144817-7cab8180-d432-11eb-977f-913c47294b2c.PNG)

Using ```docker-ps``` to list the containers and the outuput is shown:

![03-docker_ps](https://user-images.githubusercontent.com/62483710/123144951-a8c70280-d432-11eb-968d-6cfc543e2e8f.PNG)

