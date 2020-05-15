# Builder
FROM jupyter/pyspark-notebook:3b1f4f5e6cc1 as builder

WORKDIR /tmp

# build geomesa from source
RUN git clone https://github.com/locationtech/geomesa.git

WORKDIR /tmp/geomesa

# デフォルトで2.4.0 (2020/05/13時点での最新) を使用する設定としている。
# docker build 時に、--build-arg GEOMESA_VER="2.x.x とオプションを付けることにより指定できる。
ARG GEOMESA_VER="2.4.0"

RUN git checkout tags/geomesa_2.11-$GEOMESA_VER -b geomesa-$GEOMESA_VER

WORKDIR /tmp/geomesa/geomesa-spark/geomesa_pyspark/src/main/python/

RUN python setup.py sdist

# unzip compressed jars
WORKDIR /tmp

# あらかじめ下記から「FileSystem」のものをDLし、Dockerfileと同じディレクトリに配置しておくこと。
# ※ $GEOMESA_VER に対応したバージョンのものをDLすること。
# https://github.com/locationtech/geomesa/releases
# （DLが遅いので予め用意したファイルを使用するようにした。）
COPY geomesa-fs_2.11-$GEOMESA_VER-bin.tar.gz geomesa-fs_2.11-$GEOMESA_VER-bin.tar.gz

RUN tar xvf geomesa-fs_2.11-$GEOMESA_VER-bin.tar.gz



# Runner
FROM jupyter/pyspark-notebook:3b1f4f5e6cc1 as runner

ARG GEOMESA_VER="2.4.0"

# pip install用ファイルと読み込ませるjarファイル
COPY --from=builder /tmp/geomesa/geomesa-spark/geomesa_pyspark/src/main/python/dist/geomesa_pyspark--python.version-.tar.gz /tmp/geomesa_pyspark.tar.gz
COPY --from=builder /tmp/geomesa-fs_2.11-2.4.0/dist/spark/geomesa-fs-spark-runtime_2.11-$GEOMESA_VER.jar /jars/geomesa-fs-spark-runtime_2.11-$GEOMESA_VER.jar

RUN pip install /tmp/geomesa_pyspark.tar.gz

# jarを読み込むよう設定
USER root

ENV HADOOP_CONF_DIR=$SPARK_HOME/conf

RUN cp $SPARK_HOME/conf/spark-defaults.conf.template $SPARK_HOME/conf/spark-defaults.conf && \
    echo "spark.jars /jars/geomesa-fs-spark-runtime_2.11-$GEOMESA_VER.jar" >> \
        $SPARK_HOME/conf/spark-defaults.conf

USER $NB_USER

EXPOSE 8888

ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]
