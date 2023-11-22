# PyFlink: From Beginner to Proficient

## ⚠️ I've just tried to translate main repo to english with chat gpt and make some minor improvements.

This guide is based on the PyFlink learning documentation and provides a series of small exercises to help users quickly get started with PyFlink.

1. [Local Development Environment Setup](#Local-Development-Environment-Setup)
   1. [Install Flink](#Install-Flink)
      1. [Mac](#Mac)
      2. [Other Systems](#Other-Systems)
   2. [Install Other Components](#Install-Other-Components)
   3. [Install Python3](#Install-Python3)
2. [Run](#Run)

## Local Development Environment Setup
### Install Flink
#### Mac

First, upgrade the local Java version to 8 or 11:

```bash
java -version
# You may see java version "1.8.0_111"
```

Then use Homebrew to install Flink. The latest version of Flink is currently 1.11.2:

```bash
brew switch apache-flink 1.11.2
```

cd to `/usr/local/Cellar/apache-flink/1.11.2/libexec/bin/start-cluster.sh` and start Flink:

```bash
cd /usr/local/Cellar/apache-flink/1.11.2/libexec/bin
sh start-cluster.sh
```

After startup, run the `jps` command to see all local Java processes. If Flink is installed correctly, you should see the processes `TaskManagerRunner` and `StandaloneSessionClusterEntrypoint`, indicating that both the job manager and task manager have started normally.

You can also open the web page [http://localhost:8081/](http://localhost:8081/) to view the Flink job management panel. The "Available Task Slots" should display as 1 (indicating one task manager with one task slot and parallelism degree 1), and "Running Jobs" should be 0 (indicating no Flink job is executing).

To shutdown Flink, use the following command:

```bash
sh stop-cluster.sh
```

For convenience, you can modify the local `~/.bash_profile` file, insert the following 3 lines, and then run `source ~/.bash_profile` to activate the modification:

```bash
alias start-flink='/usr/local/Cellar/apache-flink/1.11.2/libexec/bin/start-cluster.sh'
alias stop-flink='/usr/local/Cellar/apache-flink/1.11.2/libexec/bin/stop-cluster.sh'
alias flink='/usr/local/Cellar/apache-flink/1.11.2/libexec/bin/flink'
```

#### Other Systems

Please refer to the [Official Documentation](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/dataset/local_execution/).

### Install Other Components

This tutorial uses databases or big data components such as MySQL, Kafka, Zookeeper, etc. For unified deployment and management, Docker is chosen.

From a development perspective, Docker is essential for the following reasons:
1. Docker ensures consistency between development and production environments.
1. Docker can simulate multi-node clusters using the docker-compose tool, making it easy to start multiple Kafka and Zookeeper containers on a single development machine, simulating a distributed environment.
1. Docker installation and startup are fast.

First, install [docker](https://www.docker.com/).

Then, in the project's root directory, start the Docker orchestration service:

```bash
# For Windows systems, add the following line first
# set COMPOSE_CONVERT_WINDOWS_PATHS=1
docker-compose up -d
```

After startup, run `docker ps` to see the 5 containers:

```bash
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS                                                  NAMES
32d6b6cdf30b        mysql:8.0.22                    "docker-entrypoint.s…"   5 days ago          Up 3 seconds        0.0.0.0:3306->3306/tcp, 33060/tcp                      mysql1
cc8246824903        mysql:8.0.22                    "docker-entrypoint.s…"   5 days ago          Up 3 seconds        33060/tcp, 0.0.0.0:3307->3306/tcp                      mysql2
f732effb7559        redis:6.0.9                     "docker-entrypoint.s…"   5 days ago          Up 5 seconds        0.0.0.0:6379->6379/tcp                                 redis
b62b8d8363c3        wurstmeister/kafka:2.13-2.6.0   "start-kafka.sh"         5 days ago          Up 3 seconds        0.0.0.0:9092->9092/tcp                                 kafka
fe2ad0230ffa        adminer                         "entrypoint.sh docke…"   5 days ago          Up 12 seconds       0.0.0.0:8080->8080/tcp                                 adminer
df80ca04755d        zookeeper:3.6.2                 "/docker-entrypoint.…"   5 days ago          Up 3 seconds        2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp, 8080/tcp   zookeeper
```

Explanation of each container's role:
* mysql + admin: Used in case 3. The mysql1 container is used as the data source to be synchronized, mysql2 as the backup data warehouse, and admin for web-based viewing and operation.
* kafka + zookeeper: Used in case 4. Kafka is a high-throughput, low-latency message middleware. Zookeeper manages Kafka's brokers for load balancing.
* redis: Used in case 5 for caching trained models in real-time calculations.

For security, passwords are set in the `docker-compose.yml` file:
1. MySQL account and password are both root.
1. Redis password is redis_password.

很简单地，我们完成了环境的搭建。

To stop:

```bash
# Stop
docker-compose stop

# Stop and delete
docker-compose down
```

If a container fails to start, delete and rebuild it. For example, for Kafka:

```bash
docker rm kafka
docker-compose up -d --build
```

### Install Python3

PyFlink requires Python version 3.5, 3.6, or 3.7. It is recommended to use Miniconda for creating a Python environment due to its small size, isolation from the system environment, and easy management of multiple virtual environments.

Find a Python3 installation tutorial on the internet for guidance.

## Run

Ensure the following before running:
1. Python environment is set up.
1. Docker is started, and containers are running.
1. Flink is installed correctly.

After preparation, the local PyFlink development and testing environment setup is complete. Proceed to the [PyFlink from Beginner to Master](examples/README.md) tutorial for detailed instructions and code in the `examples` directory.

The tutorial covers 5 cases. If you are a beginner, it is recommended to study in order:
1. Batch Processing Word Count:
    - Teaches how to use PyFlink for batch processing.
    - Demonstrates Table API and SQL API for implementing groupby processing logic.
    - Explains how to read files from a file system and store them after processing.
2. Custom Function UDF:
    - Teaches how to import third-party dependencies in PyFlink.
    - Demonstrates combining UDF (user-defined functions) for complex calculations.
3. Real-time CDC:
    - Teaches how to use PyFlink to build a real-time data warehouse.
    - Explains capturing data changes in the binlog and upserting to a backup data warehouse.
4. Real-time Ranking：
    - Teaches how to use PyFlink for stateful stream processing.
    - Demonstrates importing and using Java-written aggregate function JAR packages in the Python environment.
    - Explains using sliding windows for ranking within a specified time range.
5. Online Machine Learning:
    - Teaches how to use PyFlink for online machine learning.
    - Demonstrates connecting Redis in UDF to load and save models.
    - Explains training a model in UDF, registering indicators, calculating indicators, and viewing indicators in real-time on a web page.
    - Demonstrates developing Flask applications and providing prediction services based on the latest models in Redis.

To run each case, navigate to the case directory and execute the following script (replace xx with the corresponding script name):

```
flink run -m localhost:8081 -py xxx.py
```

For more details, refer to [PyFlink from Beginner to Master](examples/README.md).
