# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
# 
FROM jupyter/scipy-notebook

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"

USER root

# Spark dependencies
ENV APACHE_SPARK_VERSION 2.3.1
ENV HADOOP_VERSION 2.7.7

RUN apt-get -y update && \
    apt-get install --no-install-recommends -y openjdk-8-jre-headless ca-certificates-java && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Separate out spark and hadoop to get extended library of .jar files with aws support
RUN cd /tmp && \
        wget -q http://mirrors.ukfast.co.uk/sites/ftp.apache.org/spark/spark-${APACHE_SPARK_VERSION}/spark-${APACHE_SPARK_VERSION}-bin-without-hadoop.tgz && \
        tar xzf spark-${APACHE_SPARK_VERSION}-bin-without-hadoop.tgz -C /usr/local --owner root --group root --no-same-owner && \
        rm spark-${APACHE_SPARK_VERSION}-bin-without-hadoop.tgz
RUN cd /usr/local && ln -s spark-${APACHE_SPARK_VERSION}-bin-without-hadoop spark

RUN cd /tmp && \
        wget -q http://mirrors.ukfast.co.uk/sites/ftp.apache.org/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz && \
        tar xzf hadoop-${HADOOP_VERSION}.tar.gz -C /usr/local --owner root --group root --no-same-owner && \
        rm hadoop-${HADOOP_VERSION}.tar.gz
RUN cd /usr/local && ln -s hadoop-${HADOOP_VERSION} hadoop

# Mesos dependencies
# Install from the Xenial Mesosphere repository since there does not (yet)
# exist a Bionic repository and the dependencies seem to be compatible for now.
COPY mesos.key /tmp/
RUN apt-get -y update && \
    apt-get install --no-install-recommends -y gnupg && \
    apt-key add /tmp/mesos.key && \
    echo "deb http://repos.mesosphere.io/ubuntu xenial main" > /etc/apt/sources.list.d/mesosphere.list && \
    apt-get -y update && \
    apt-get --no-install-recommends -y install mesos=1.2\* && \
    apt-get purge --auto-remove -y gnupg && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Spark and Mesos config
ENV SPARK_HOME /usr/local/spark
ENV HADOOP_HOME /usr/local/hadoop
ENV SPARK_DIST_CLASSPATH "$HADOOP_HOME/*:$HADOOP_HOME/share/hadoop/common/lib/*:$HADOOP_HOME/share/hadoop/common/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/hdfs/lib/*:$HADOOP_HOME/share/hadoop/yarn/lib/*:$HADOOP_HOME/share/hadoop/yarn/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*:$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/tools/lib/*"
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip
ENV MESOS_NATIVE_LIBRARY /usr/local/lib/libmesos.so
ENV SPARK_OPTS --driver-java-options=-Xms1024M --driver-java-options=-Xmx4096M --driver-java-options=-Dlog4j.logLevel=info


USER $NB_UID

# Install pyarrow
RUN conda install --quiet -y 'pyarrow' && \
    conda clean -tipsy && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER