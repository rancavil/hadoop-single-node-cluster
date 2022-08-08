# Hadoop Single Node Cluster on Docker.

Following this steps you can build and use the image to create a Hadoop Single Node Cluster containers.

# Pull the image

     $ docker pull julienlau/hadoop-single-node-cluster:3.3.3

## Creating the hadoop image

     $ git clone https://github.com/rancavil/hadoop-single-node-cluster.git
     $ cd hadoop-single-node-cluster
     $ docker build -t hadoop .

## Creating the container

To run and create a container execute the next command:

     $ docker run --name <container-name> -p 9864:9864 -p 9870:9870 -p 8088:8088 -p 9000:9000 --hostname <your-hostname> hadoop

Then type the standard docker command `Ctrl+p` then `Ctrl+q` if you want to detach from the container and keep it running as a daemon or run directly with option `-d`.

Change **container-name** by your favorite name and set **your-hostname** with by your ip or name machine. You can use **myhdfs** as your-hostname

When you run the container, at the entrypoint you use the docker-entrypoint.sh shell that creates and starts the hadoop environment.

You should get the following prompt:

     hduser@myhdfs:~$ 

To check if hadoop container is working:

- go to the url in your browser: http://localhost:9870
- use hdfs bin from outside the host `docker cp <container-name>:/home/hduser/hadoop-3.3.3/bin/hdfs .` and try a mkdir `./hdfs dfs -mkdir hdfs://localhost:9000/tmp`
  
**Notice**: if you want to change to another port than 9000 you must also adapt the file core-site.xml and rebuild the image... or redirect the port to say 19000 by using at docker run the option `-p 19000:9000`

**Notice:** the hdfs-site.xml configure has the property, so don't use it in a production environment.

     <property>
          <name>dfs.permissions</name>
          <value>false</value>
     </property>

## A first example

Make the HDFS directories required to execute MapReduce jobs:

     hduser@myhdfs:~$ hdfs dfs -mkdir /user
     hduser@myhdfs:~$ hdfs dfs -mkdir /user/hduser

Copy the input files into the distributed filesystem:
      
     hduser@myhdfs:~$ hdfs dfs -mkdir input
     hduser@myhdfs:~$ hdfs dfs -put $HADOOP_HOME/etc/hadoop/*.xml input

Run some of the examples provided:

     hduser@myhdfs:~$ hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.3.jar grep input output 'dfs[a-z.]+'

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

     hduser@myhdfs:~$ hdfs dfs -ls output/
     Found 2 items
     -rw-r--r--   1 hduser supergroup          0 2020-08-08 01:58 output/_SUCCESS
     -rw-r--r--   1 hduser supergroup         47 2020-08-08 01:58 output/part-r-00000

Checking the result using **cat** command on the distributed filesystem:

     hduser@myhdfs:~$ hdfs dfs -cat output/*
     1	dfsadmin
     1	dfs.replication
     1	dfs.permissions


## Stopping and re-starting the container

To stop the container execute the following commands, to gratefully shutdown.

     hduser@myhdfs:~$ stop-dfs.sh
     hduser@myhdfs:~$ stop-yarn.sh

After that.

     hduser@myhdfs:~$ exit

To re-start the container, and go back to our Hadoop environment execute:

     $ docker start -i <container-name>

## Data persistence

This docker does not use volume. 
Data will not be persisted beyond the life of a container instance.
You would clean the data by doing:

     $ docker stop <container-name> && docker rm <container-name>

## Kubernetes 101

To host it on a dev kubernetes cluster:
```
kubectl apply -f hdfs-deployment.yaml
kubectl expose deployment hdfs --type=NodePort --name=hdfs-service
kubectl get svc hdfs-service
# get map for port 9000 of service hdfs-service
kubectl get svc hdfs-service -o=jsonpath='{.spec.ports[?(@.port==9000)].nodePort}'
kubectl get svc --all-namespaces -o go-template='{{range .items}}{{ $save := . }}{{range.spec.ports}}{{if .nodePort}}{{$save.metadata.namespace}}{{"/"}}{{$save.metadata.name}}{{" - "}}{{.name}}{{": "}}{{.targetPort}}{{" -> "}}{{.nodePort}}{{"\n"}}{{end}}{{end}}{{end}}'
```

At this point you should be able to run a tpcx-hs on kubernetes workload using the spark-submit option `--conf "spark.hadoop.fs.defaultFS=hdfs://hdfs-service.default.svc.cluster.local:9000/"` by using for example the following project : https://github.com/julienlau/tpcx-hs