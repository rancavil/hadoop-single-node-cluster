# MapReduce Example with Python

We are going to execute an example of MapReduce using Python. This is the typical words count example.

## Loading files into HDFS (Hadoop Distributed FileSystem).

First of all, we have to go to the directory examples.

     hduser@localhost:~$ cd examples

Now, copy the files txt from the local filesystem to HDFS using the following commands.

     hduser@localhost:~/examples$ hdfs dfs -put *.txt input

**Note:** if you aren't created the directory input in the hadoop distributed filesystem you have to execute the following commands:

     hduser@localhost:~/examples$ hdfs dfs -mkdir /user
     hduser@localhost:~/examples$ hdfs dfs -mkdir /user/hduser
     hduser@localhost:~/examples$ hdfs dfs -mkdir input

We can check the files loaded on the distributed file system using.

     hduser@localhost:~/examples$ hdfs dfs -ls input
     Found 4 items
     -rw-r--r--   1 hduser supergroup    1586488 2020-08-09 00:29 input/4300-0.txt
     -rw-r--r--   1 hduser supergroup    1428841 2020-08-09 00:29 input/5000-8.txt
     -rw-r--r--   1 hduser supergroup      15929 2020-08-09 00:29 input/data-text.txt
     -rw-r--r--   1 hduser supergroup     674570 2020-08-09 00:29 input/pg20417.txt

## Checking and understanding the code

### mapper.py

The mapper will read lines from stdin (standard input). Hadoop will send a stream of data read from the HDFS to the mapper using the stdout (standard output). The mapper will read each line sent through the stdin, cleaning all characters non-alphanumerics, and creating a Python list with words (split). Finally, it will create string "word\t1", it is a pair (work,1), the result is sent to the datastream again using the stdout (print).

     #!/usr/bin/env python

     import sys
     import re

     for line in sys.stdin:
          line = re.sub(r'\W+',' ',line.strip())
          words = line.split()

          for word in words:
               print('{}\t{}'.format(word,1))

### reducer.py

The reducer will read every input (line) from the stdin and will count every repeated word (increasing the counter for this word) and will send the result to the stdout. The process will be executed in an iterative way until there aren't more inputs in the stdin.

     #!/usr/bin/env python

     import sys

     current_word = None
     current_count = 0
     word = None

     for line in sys.stdin:
          line = line.strip()
          word, count = line.split('\t',1)

          try:
               count = int(count)
          except ValueError:
               continue

          if current_word == word:
               current_count += count
          else:
               if current_word:
                    print('{}\t{}'.format(current_word,current_count))
               current_word = word
               current_count = count


     if current_word == word:
          print('{}\t{}'.format(current_word,current_count))


## Executing the MapReduce

The following command will execute the MapReduce process using the txt files located in **/user/hduser/input** (HDFS), **mapper.py** and **reducer.py**. The result will be written in the distributed file system **/user/hduser/output**.

     hduser@localhost:~/examples$ hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.0.jar -mapper mapper.py -reducer reducer.py -input /user/hduser/input/*.txt -output /user/hduser/output

To check the results we can execute.

     hduser@localhost:~/examples$ hdfs dfs -ls output
     Found 2 items
     -rw-r--r--   1 hduser supergroup          0 2020-08-09 00:31 output/_SUCCESS
     -rw-r--r--   1 hduser supergroup     530859 2020-08-09 00:31 output/part-00000

     hduser@localhost:~/examples$ hdfs dfs -cat output/*
     0       64
     00      2
     000     116
     001     1
     01      1
     02      4
     .......
     ......
     ....
     Abulafia        1
     Abulfeda        1
     Academie        3
     Academy 4
     Accademia       7
     Accademia_      1
     Accep   1
     .......
     ......
     ....
     zoophyte        2
     zoophytes       2
     zouave  1
     zrads   3
     zum     1
     zur     1
     zvith   1
     zwanzig 1
     zweite  1

  
This is a simple way (with a simple example) to understand how MapReduce works.
