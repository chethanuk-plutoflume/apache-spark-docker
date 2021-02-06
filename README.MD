# Spark 3 Docker

To start the spark cluster:

```bash
docker-compose up

OR 

docker-compose -f docker-compose-pip-req.yml up
```

To scale the workers

```bash
docker-compose up --scale spark-worker=4
```

## Environment variables

When you start the spark image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the `docker run` command line. If you want to add a new environment variable:

```yaml
spark:
  ...
  environment:
    - SPARK_MODE=master
  ...
```

## Available variables

* SPARK_MODE: Cluster mode starting Spark. Valid values: *master*, *worker*. Default: **master**
* SPARK_MASTER_URL: Url where the worker can find the master. Only needed when spark mode is *worker*. Default: **spark://spark-master:7077**
* SPARK_RPC_AUTHENTICATION_ENABLED: Enable RPC authentication. Default: **no**
* SPARK_RPC_AUTHENTICATION_SECRET: The secret key used for RPC authentication. No defaults.
* SPARK_RPC_ENCRYPTION_ENABLED: Enable RPC encryption. Default: **no**
* SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED: Enable local storage encryption: Default **no**
* SPARK_SSL_ENABLED: Enable SSL configuration. Default: **no**
* SPARK_SSL_KEY_PASSWORD: The password to the private key in the key store. No defaults.
* SPARK_SSL_KEYSTORE_PASSWORD: The password for the key store. No defaults.
* SPARK_SSL_TRUSTSTORE_PASSWORD: The password for the trust store. No defaults.
* SPARK_SSL_NEED_CLIENT_AUTH: Whether to require client authentication. Default: **yes**
* SPARK_SSL_PROTOCOL: TLS protocol to use. Default: **TLSv1.2**
* SPARK_DAEMON_USER: Spark system user when the container is started as root. Default: **spark**
* SPARK_DAEMON_GROUP: Spark system group when the container is started as root. Default: **spark**

More environment variables natively supported by Spark can be found [at the official documentation](https://spark.apache.org/docs/latest/spark-standalone.html#cluster-launch-scripts).
For example, you could still use `SPARK_WORKER_CORES` or `SPARK_WORKER_MEMORY` to configure the number of cores and the amount of memory to be used by a worker machine.

## Installing additional jars

By default, this container bundles a generic set of jar files but the default image can be extended to add as many jars as needed for your specific use case. For instance, the following Dockerfile adds [`aws-java-sdk-bundle-1.11.704.jar`](https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-bundle/1.11.704):

```Dockerfile
FROM bitnami/spark
RUN curl https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.11.704/aws-java-sdk-bundle-1.11.704.jar --output /opt/bitnami/spark/jars/aws-java-sdk-bundle-1.11.704.jar
```


### Using a different version of Hadoop jars

In a similar way that in the previous section, you may want to use a different version of Hadoop jars.

Go to https://spark.apache.org/downloads.html and copy the download url bundling the Hadoop version you want and matching the Spark version of the container. Extend the Bitnami container image as below:

```Dockerfile
FROM bitnami/spark:3.0.0

USER root
RUN rm -r /opt/bitnami/spark/jars && \
    curl --location http://mirror.cc.columbia.edu/pub/software/apache/spark/spark-3.0.0/spark-3.0.0-bin-hadoop2.7.tgz | \
    tar --extract --gzip --strip=1 --directory /opt/bitnami/spark/ spark-3.0.0-bin-hadoop2.7/jars/
USER 1001
```

You can check the Hadoop version by running the following commands in the new container image:

```console
$ pyspark
>>> sc._gateway.jvm.org.apache.hadoop.util.VersionInfo.getVersion()
'2.7.4'
```

# Logging

The Bitnami Spark Docker image sends the container logs to the `stdout`. To view the logs:

```console
$ docker logs spark
```

or using Docker Compose:

```console
$ docker-compose logs spark
```

You can configure the containers [logging driver](https://docs.docker.com/engine/admin/logging/overview/) using the `--log-driver` option if you wish to consume the container logs differently. In the default configuration docker uses the `json-file` driver.

More at bitnami/bitnami-docker-spark
