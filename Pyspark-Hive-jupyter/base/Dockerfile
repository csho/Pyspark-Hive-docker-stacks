FROM jupyter/scipy-notebook
# tag:  pyspark-hive-base

# Add RUN statements to install packages as the $NB_USER defined in the base images.

# Add a "USER root" statement followed by RUN statements to install system packages using apt-get,
# change file permissions, etc.

# If you do switch to root, always be sure to add a "USER $NB_USER" command at the end of the
# file to ensure the image runs as a unprivileged user by default.

USER root
# arm : archive ;  amd: ports
RUN rm -rf /var/lib/apt/lists/* \ 
 && sed -i -e "s|http://ports.ubuntu.com|http://mirrors.aliyun.com|g" /etc/apt/sources.list \
 && apt-get update  \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      openjdk-8-jdk \
      curl \
      gnupg \
      procps \
      coreutils \
      libc6-dev \
 && rm -rf /var/lib/apt/lists/*

ARG USERNAME=${NB_USER}
ARG GROUPNAME=users
ARG UID=${NB_UID}
ARG GID=${NB_GID}


USER root

ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
# for arm ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-arm64/

# Hadoop
ENV HADOOP_VERSION=3.3.1
ENV HADOOP_URL=https://www.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz
ENV HADOOP_HOME=/opt/hadoop

RUN set -x \
 && curl -fSL https://archive.apache.org/dist/hadoop/common/KEYS -o /tmp/hadoop-KEYS \
 && gpg --import /tmp/hadoop-KEYS \
 && sudo mkdir $HADOOP_HOME  \
 && sudo chown $USERNAME:$GROUPNAME -R $HADOOP_HOME \
 && curl -fSL $HADOOP_URL -o /tmp/hadoop.tar.gz \
 && curl -fSL $HADOOP_URL.asc -o /tmp/hadoop.tar.gz.asc \
 && gpg --verify /tmp/hadoop.tar.gz.asc \
 && tar -xvf /tmp/hadoop.tar.gz -C $HADOOP_HOME --strip-components 1 \
 && mkdir $HADOOP_HOME/logs \
 && rm /tmp/hadoop*

ENV HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
ENV PATH=$HADOOP_HOME/sbin:$HADOOP_HOME/bin:$PATH
ENV LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH

# Spark
ENV SPARK_VERSION=3.2.1
ENV SPARK_URL=https://archive.apache.org/dist/spark/spark-$SPARK_VERSION/spark-$SPARK_VERSION-bin-hadoop3.2.tgz
ENV SPARK_HOME=/opt/spark

RUN set -x \
 && curl -fSL https://archive.apache.org/dist/spark/KEYS -o /tmp/spark-KEYS  \
 && gpg --import /tmp/spark-KEYS \
 && sudo mkdir $SPARK_HOME \
 && sudo chown $USERNAME:$GROUPNAME -R $SPARK_HOME \
 && curl -fSL $SPARK_URL -o /tmp/spark.tgz \
 && curl -fSL $SPARK_URL.asc -o /tmp/spark.tgz.asc \
 && gpg --verify /tmp/spark.tgz.asc \
 && tar -xvf /tmp/spark.tgz -C $SPARK_HOME --strip-components 1 \
 && rm /tmp/spark* \
 && curl -fSL https://jdbc.postgresql.org/download/postgresql-42.3.2.jar -o $SPARK_HOME/jars/postgresql-42.3.2.jar

ENV PYTHONHASHSEED=1
ENV PYSPARK_PYTHON=python3
ENV SPARK_CONF_DIR=$SPARK_HOME/conf
ENV PATH=$SPARK_HOME/sbin:$SPARK_HOME/bin:$PATH

# Hive
ENV HIVE_VERSION=2.3.9
ENV HIVE_URL=https://archive.apache.org/dist/hive/hive-$HIVE_VERSION/apache-hive-$HIVE_VERSION-bin.tar.gz
ENV HIVE_HOME=/opt/hive

RUN set -x \
 && curl -fSL https://archive.apache.org/dist/hive/KEYS -o /tmp/hive-KEYS  \
 && gpg --import /tmp/hive-KEYS \
 && sudo mkdir $HIVE_HOME \
 && sudo chown $USERNAME:$GROUPNAME -R $HIVE_HOME \
 && curl -fSL $HIVE_URL -o /tmp/hive.tar.gz \
 && curl -fSL $HIVE_URL.asc -o /tmp/hive.tar.gz.asc \
 && gpg --verify /tmp/hive.tar.gz.asc \
 && tar -xvf /tmp/hive.tar.gz -C $HIVE_HOME --strip-components 1 \
 && rm /tmp/hive*

#RUN sudo rm $HIVE_HOME/lib/guava-*.jar \
# && sudo cp $HADOOP_HOME/share/hadoop/hdfs/lib/guava-*.jar $HIVE_HOME/lib/
RUN mv /opt/hive/lib/log4j-slf4j-impl-2.6.2.jar /opt/hive/lib/log4j-slf4j-impl-2.6.2.jar.bak

ENV HIVE_CONF_DIR=$HIVE_HOME/conf
ENV PATH=$HIVE_HOME/sbin:$HIVE_HOME/bin:$PATH

# Config
COPY conf/core-site.xml $HADOOP_CONF_DIR/
COPY conf/hdfs-site.xml $HADOOP_CONF_DIR/
COPY conf/yarn-site.xml $HADOOP_CONF_DIR/
COPY conf/mapred-site.xml $HADOOP_CONF_DIR/
COPY conf/workers $HADOOP_CONF_DIR/
COPY conf/spark-defaults.conf $SPARK_CONF_DIR/
COPY conf/log4j.properties $SPARK_CONF_DIR/
COPY conf/hive-site.xml $HIVE_CONF_DIR/

RUN ln $HADOOP_CONF_DIR/workers $SPARK_CONF_DIR/ \
 && ln $HIVE_CONF_DIR/hive-site.xml $SPARK_CONF_DIR/

# Entry point
COPY entrypoint.sh /usr/local/sbin/entrypoint.sh
RUN sudo chmod a+x /usr/local/sbin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

USER $NB_USER