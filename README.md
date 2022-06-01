# Hadoop Single Node Cluster on Docker.

Following this steps you can build and use the image to create a Hadoop Single Node Cluster containers.

## Creating the hadoop image

     $ git clone https://github.com/rancavil/hadoop-single-node-cluster.git
     $ cd hadoop-single-node-cluster
     $ docker build -t hadoop .

## Creating the container

To run and create a container execute the next command:

     $ docker run -it --name <container-name> -p 9864:9864 -p 9870:9870 -p 8088:8088 --hostname <your-hostname> hadoop

Change **container-name** by your favorite name and set **your-hostname** with by your ip or name machine. You can use **localhost** as your-hostname

When you run the container, at the entrypoint you use the docker-entrypoint.sh shell that creates and starts the hadoop environment.

You should get the following prompt:

     hduser@localhost:~$ 

To check if hadoop container is working go to the url in your browser.

     http://localhost:9870

**Notice:** the hdfs-site.xml configure has the property, so don't use it in a production environment.

     <property>
          <name>dfs.permissions</name>
          <value>false</value>
     </property>

## A first example

Make the HDFS directories required to execute MapReduce jobs:

     hduser@localhost:~$ hdfs dfs -mkdir /user
     hduser@localhost:~$ hdfs dfs -mkdir /user/hduser

Copy the input files into the distributed filesystem:
      
     hduser@localhost:~$ hdfs dfs -mkdir input
     hduser@localhost:~$ hdfs dfs -put $HADOOP_HOME/etc/hadoop/*.xml input

Run some of the examples provided:

     hduser@localhost:~$ hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.3.jar grep input output 'dfs[a-z.]+'

     2020-08-08 01:57:02,411 INFO impl.MetricsConfig: Loaded properties from hadoop-metrics2.properties
     2020-08-08 01:57:04,754 INFO impl.MetricsSystemImpl: Scheduled Metric snapshot period at 10 second(s).
     2020-08-08 01:57:04,754 INFO impl.MetricsSystemImpl: JobTracker metrics system started
     2020-08-08 01:57:08,843 INFO input.FileInputFormat: Total input files to process : 10
     ..............
     .............
     ............
     File Input Format Counters 
          Bytes Read=175
     File Output Format Counters 
          Bytes Written=47

Examine the output files: check the output files from the distributed filesystem and examine them:

     hduser@localhost:~$ hdfs dfs -ls output/
     Found 2 items
     -rw-r--r--   1 hduser supergroup          0 2020-08-08 01:58 output/_SUCCESS
     -rw-r--r--   1 hduser supergroup         47 2020-08-08 01:58 output/part-r-00000

Checking the result using **cat** command on the distributed filesystem:

     hduser@localhost:~$ hdfs dfs -cat output/*
     1	dfsadmin
     1	dfs.replication
     1	dfs.permissions


## Stopping and re-starting the container

To stop the container execute the following commands, to gratefully shutdown.

     hduser@localhost:~$ stop-dfs.sh
     hduser@localhost:~$ stop-yarn.sh

After that.

     hduser@localhost:~$ exit

To re-start the container, and go back to our Hadoop environment execute:

     $ docker start -i <container-name>


