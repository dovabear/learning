NameNode Web Broswer
http://<name-node-ip>:50070
需先安裝Java

安裝 Spark Standalone
<spkma, worker01, worker02>

For Spark 2.3.0 & Hadoop 2.7 later
wget http://ftp.twaren.net/Unix/Web/apache/spark/spark-2.3.0/spark-2.3.0-bin-hadoop2.7.tgz

tar xvfz spark-2.3.0-bin-hadoop2.7.tgz -C /opt

chown -R root:root /opt/spark-2.3.0-bin-hadoop2.7/

vi /opt/myhadoop.sh
<add-lines>
export SPARK_HOME=/opt/spark-2.3.0-bin-hadoop2.7/
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

vi /opt/spark-2.3.0-bin-hadoop2.7/conf/spark-env.sh
# Cluster Mode
export SPARK_MASTER_HOST=spkma
export SPARK_LOG_DIR=/tmp

# Master and Worker Daemon Heap Size
export SPARK_DAEMON_MEMORY=2048m
export SPARK_WORKER_MEMORY=10240m
export SPARK_WORKER_DIR=/root/<spark-hostname>

# pyspark
#export ANACONDA_ROOT=/opt/anaconda3
#export PYSPARK_PYTHON=$ANACONDA_ROOT/bin/python

# Setup Spark Default Conf

cp /opt/spark-2.3.0-bin-hadoop2.7/conf/spark-defaults.conf.template /opt/spark-2.3.0-bin-hadoop2.7/conf/spark-defaults.conf

vi  /opt/spark-2.3.0-bin-hadoop2.7/conf/spark-defaults.conf
<add-lines>
spark.master spark://spkma:7077
spark.cores.max 4
spark.executor.cores 2
spark.executor.memory 4096m

# Expirements
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
<spkma, worker01, worker02>
wget https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh -O /tmp/anaconda3.sh

chmod +x /tmp/anaconda3.sh 

/tmp/anaconda3.sh 
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

/opt/anaconda3/bin/conda update conda

......
The following packages will be UPDATED:

    conda: 4.3.27-py36h2866c0b_0 --> 4.3.30-py36h5d9f9f4_0

Proceed ([y]/n)? y

/opt/anaconda3/bin/conda update anaconda

Fetching package metadata ...........
Solving package specifications: .
# All requested packages already installed.
# packages in environment at /opt/anaconda3:
#
anaconda                  5.0.0.1          py36hfb0b618_1 

/opt/anaconda3/bin/conda upgrade python

檢視 Anaconda 分析平台的 Python 版本
/opt/anaconda3/bin/python -V
Python 3.6.3 :: Anaconda, Inc.

vi /opt/myhadoop.sh
<add-lines>
export ANACONDA_ROOT=/opt/anaconda3
export PATH=$ANACONDA_ROOT/bin:$SPARK_HOME/bin:$SPARK_HOME/sbin:$PATH

設定 Spark 叢集使用 Anaconda 

vi /opt/spark-2.3.0-bin-hadoop2.7/conf/spark-env.sh
<modify>

# pyspark
export ANACONDA_ROOT=/opt/anaconda3
export PYSPARK_PYTHON=$ANACONDA_ROOT/bin/python

設計 Jupyter Server 啟動命令

vi /opt/bin/startjupyter

#!/bin/bash
export ANACONDA_ROOT=/opt/anaconda3
export PYSPARK_DRIVER_PYTHON=$ANACONDA_ROOT/bin/jupyter
export PYSPARK_DRIVER_PYTHON_OPTS="notebook --ip='*'  --no-browser --allow-root"
pyspark --master spark://spkma:7077 --total-executor-cores 2 --executor-cores 1 --executor-memory 2048m 
#Pyspark=jupyter notebook --ip ='*'  --no-browser

安裝本機R
sudo apt-get install r-base -y

安裝 R 分析工具

sudo /opt/anaconda3/bin/conda install -c r r-base r-essentials r-sparklyr

Fetching package metadata .............
Solving package specifications: .

Package plan for installation in environment /opt/anaconda3:

The following NEW packages will be INSTALLED:
    .....          
    r-assertthat:     0.1-r3.3.2_4           r
    r-backports:      1.0.4-r3.3.2_0         r
    .......
The following packages will be UPDATED:
    anaconda:         5.0.0.1-py36hfb0b618_1   --> custom-py36_0        
    conda:            4.3.27-py36h2866c0b_0    --> 4.3.28-py36h620ea6e_0
    ........
The following packages will be DOWNGRADED:
    cairo:            1.14.10-h58b644b_4       --> 1.14.8-0             
    ...............          

Proceed ([y]/n)? y 

設定 SparkR 叢集使用 Anaconda 
vi /opt/spark-2.3.0-bin-hadoop2.7/conf/spark-env.sh
<add-lines>
export SPARKR_DRIVER_R=$ANACONDA_ROOT/bin/R

Restart Spark Standalone

<Master>
/opt/spark-2.2.1-bin-hadoop2.7/sbin/stop-master.sh
/opt/spark-2.2.1-bin-hadoop2.7/sbin/start-master.sh

<Worker>
/opt/spark-2.2.1-bin-hadoop2.7/sbin/stop-slave.sh
/opt/spark-2.2.1-bin-hadoop2.7/sbin/start-slave.sh spark://spma:7077

設計 Jupyter Server 啟動命令
<USE User=root>

vi /opt/bin/startjupyter

#!/bin/bash
export ANACONDA_ROOT=/opt/anaconda3
export PYSPARK_DRIVER_PYTHON=$ANACONDA_ROOT/bin/jupyter
export PYSPARK_DRIVER_PYTHON_OPTS="notebook --ip='*'  --no-browser --allow-root"
pyspark --master spark://spma:7077 --total-executor-cores 1 --executor-cores 1 --executor-memory 512m 
#Pyspark=jupyter notebook --ip ='*'  --no-browser

chmod +x /opt/bin/startjupyter

啟動startjupyter失敗時
conda update jupyter_core jupyter_client

列出jupyter kernel
jupyter kernelspec list

jupyter Spark Properties Debug
sc
spark
sc.getConf().toDebugString()

jupyter SparkR Properties Debug
sparkR.session()
sessionInfo()
sparkR.conf()