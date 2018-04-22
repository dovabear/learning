NameNode Web Broswer
http://<name-node-ip>:50070

For Spark 2.3.0 & Hadoop 2.7 later

Download Spark
https://www.apache.org/dyn/closer.lua/spark/spark-2.3.0/spark-2.3.0-bin-hadoop2.7.tgz

設定使用那個  Spark 版本套件
$ sudo nano /opt/bin/dkc.bash
                             
export SPARK_HOME=/opt/spark-2.3.0-bin-hadoop2.7/
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
                         
$ sudo nano /opt/spark-2.3.0-bin-hadoop2.7/conf/spark-env.sh
# Cluster Mode
export SPARK_MASTER_HOST=spkma
export SPARK_LOG_DIR=/tmp

# Master and Worker Daemon Heap Size
export SPARK_DAEMON_MEMORY=1024m
export SPARK_WORKER_MEMORY=10240m
export SPARK_WORKER_DIR=/home/bigred/worker

# pyspark
#export ANACONDA_ROOT=/opt/anaconda3
#export PYSPARK_PYTHON=$ANACONDA_ROOT/bin/python

Add User 'hackerthon' for Spark
sudo adduser hackerthon

Starting a Cluster Manually
You can start a standalone master server by executing:

./sbin/start-master.sh

Once started, the master will print out a spark://HOST:PORT URL for itself, which you can use to connect workers to it, or pass as the “master” argument to SparkContext. You can also find this URL on the master’s web UI, which is http://localhost:8080 by default.

Similarly, you can start one or more workers and connect them to the master via:

./sbin/start-slave.sh <master-spark-URL>

<master-spark-URL> = spark://<master-spark-host>:7077