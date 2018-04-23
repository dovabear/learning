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

安裝 Anaconda

wget https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh -O /tmp/anaconda3.sh

chmod +x /tmp/anaconda3.sh 

sudo /tmp/anaconda3.sh 
.........
Do you approve the license terms? [yes|no]
>>> yes

Anaconda3 will now be installed into this location:
/home/bigred/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/home/bigred/anaconda3] >>> /opt/anaconda3
........
Do you wish the installer to prepend the Anaconda3 install location
to PATH in your /home/bigred/.bashrc ? [yes|no]
[no] >>> no

更新 Anaconda

sudo /opt/anaconda3/bin/conda update conda

sudo /opt/anaconda3/bin/conda update anaconda

sudo /opt/anaconda3/bin/conda upgrade python

/opt/anaconda3/bin/python -V
Python 3.6.3 :: Anaconda, Inc.

設定 Anaconda

sudo nano /opt/myhadoop.sh
<add-lines>
export ANACONDA_ROOT=/opt/anaconda3
export PATH=$ANACONDA_ROOT/bin:$SPARK_HOME/bin:$SPARK_HOME/sbin:$PATH

設定 Spark 叢集使用 Anaconda

sudo nano /opt/spark-2.2.1-bin-hadoop2.7/conf/spark-env.sh
<add-lines>
# pyspark
export ANACONDA_ROOT=/opt/anaconda3
export PYSPARK_PYTHON=$ANACONDA_ROOT/bin/python

Restart Spark Standalone

<Master>
/opt/spark-2.2.1-bin-hadoop2.7/sbin/stop-master.sh
/opt/spark-2.2.1-bin-hadoop2.7/sbin/start-master.sh

<Worker>
/opt/spark-2.2.1-bin-hadoop2.7/sbin/stop-slave.sh
/opt/spark-2.2.1-bin-hadoop2.7/sbin/start-slave.sh spark://spma:7077

設計 Jupyter Server 啟動命令
<USE User=root>

sudo nano /opt/bin/startjupyter

#!/bin/bash
export ANACONDA_ROOT=/opt/anaconda3
export PYSPARK_DRIVER_PYTHON=$ANACONDA_ROOT/bin/jupyter
export PYSPARK_DRIVER_PYTHON_OPTS="notebook --ip='*'  --no-browser --allow-root"
pyspark --master spark://spma:7077 --total-executor-cores 1 --executor-cores 1 --executor-memory 512m 
#Pyspark=jupyter notebook --ip ='*'  --no-browser

sudo chmod +x /opt/bin/startjupyter