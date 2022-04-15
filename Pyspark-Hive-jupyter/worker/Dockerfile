FROM pyspark-hive-base
#tag: pyspark-hive-worker

USER root

ENV DATADIR=/opt/hadoop/dfs/data
RUN mkdir -p /opt/hadoop/dfs/data
VOLUME /opt/hadoop/dfs/data

ENV SPARK_PUBLIC_DNS=localhost
ENV SPARK_MASTER_ADDRESS=spark://master:7077

COPY run.sh /usr/local/sbin/run.sh
RUN chmod a+x /usr/local/sbin/run.sh
CMD ["run.sh"]

USER $NB_USER