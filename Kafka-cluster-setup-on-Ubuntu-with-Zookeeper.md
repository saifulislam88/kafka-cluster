## 3-Broker Kafka Multi-Node Cluster with Zookeeper on Ubuntu 24

<img src=https://github.com/user-attachments/assets/a11a8870-c157-466e-adcf-7ce0707c63b6 width="800" height="350"/>

---

<img src=https://github.com/user-attachments/assets/4edb6f66-b516-4e43-9ea4-535b8cb1c45e width="800" height="250"/>


### **Pre-requisites**
- Kafka latest package
- Ubuntu VMs(3 nodes)
- Root Access

### Step🌐1 — Add custom hostnames to /etc/hosts file
Since we are creating `3` Ubuntu Servers with custom hostnames for kafka and zookeeper, we are required to add this ip to hostname mapping in `/etc/hosts` file

`sudo vim /etc/hosts`

```sh
192.168.100.101  kafka-1.tallykhata.com                    kafka-1
192.168.100.101  zookeeper-1.tallykhata.com                zookeeper-1
192.168.100.102  kafka-2.tallykhata.com                    kafka-2
192.168.100.102  zookeeper-2.tallykhata.com                zookeeper-2
192.168.100.103  kafka-3.tallykhata.com                    kafka-3
192.168.100.103  zookeeper-3.tallykhata.com                zookeeper-3
```

<img src=https://github.com/user-attachments/assets/eee57da9-1491-4e42-931d-3c8a9b54722b width="800" height="250"/>


### Step🌐2 — Install latest Java(OpenJDK) and Disable RAM Swap

```sh
sudo apt update -y
sudo apt search openjdk
sudo apt install default-jdk
java -version
swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
echo 'vm.swappiness=1' | sudo tee --append /etc/sysctl.conf
cat /etc/sysctl.conf |grep 'swappiness'                                       #Output "vm.swappiness=1"
```


### Step🌐3 — Download the latest Kafka binaries

```sh
wget https://downloads.apache.org/kafka/3.8.1/kafka_2.13-3.8.1.tgz
tar -xvf kafka_2.13-3.8.1.tgz
sudo mv kafka_2.13-3.8.1 /opt/kafka
```

### Step🌐4 — Create a New Directory for Kafka and Zookeeper

```sh
sudo mkdir -p /data/kafka                                                     #A new directory for Kafka message and logs
sudo mkdir -p /data/zookeeper                                                 #It is snapshot and data directory for Zookeeper
```


## Step🌐5 — Zookeeper Configuration

### 1. 🟡Create a Zookeeper Uniq one ID on each VM for Zookeeper | Specify Uniq an ID

#🎯`"1"` to specify Kafka-Zookeeper server #1:
```sh
echo "1" > /data/zookeeper/myid                                                                
```

#🎯`"2"` to specify Kafka-Zookeeper Server #2:
```sh
echo "2" > /data/zookeeper/myid                                                                
```

#🎯`"3"` to specify Kafka-Zookeeper server #3:
```sh
echo "3" > /data/zookeeper/myid                                                                
```


### 2. 🟡Edit Zookeeper Configuration Files

Use the following command to backup the existing `zookeeper.properties` file (in the config directory) and create a new `zookeeper.properties` file:

`mv /opt/kafka/config/zookeeper.properties /opt/kafka/config/zookeeper.properties_ori-backup-msi`\
`vim /opt/kafka/config/zookeeper.properties`\
Copy and paste the following into the contents of the `zookeeper.properties` file (don't change this file):

```sh
# the directory where the snapshot is stored.
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181
# setting number of connections to unlimited
maxClientCnxns=0
# keeps a heartbeat of zookeeper in milliseconds
tickTime=2000
# time for initial synchronization
initLimit=10
# how many ticks can pass before timeout
syncLimit=5
# define servers ip or hostname and internal ports to zookeeper
server.1=zookeeper-1:2888:3888
server.2=zookeeper-2:2888:3888
server.3=zookeeper-3:2888:3888
```

### 3. 🟡Create the Zookeeper Service | The `init.d` scripts to start and stop Zookeeper service

`vim /etc/init.d/zookeeper`

```sh
#!/bin/bash
#/etc/init.d/zookeeper
DAEMON_PATH=/opt/kafka/bin
DAEMON_NAME=zookeeper
# Check that networking is up.
#[ ${NETWORKING} = "no" ] && exit 0

PATH=$PATH:$DAEMON_PATH

case "$1" in
  start)
        # Start daemon.
        pid=`ps ax | grep -i 'org.apache.zookeeper' | grep -v grep | awk '{print $1}'`
        if [ -n "$pid" ]
          then
            echo "Zookeeper is already running";
        else
          echo "Starting $DAEMON_NAME";
          $DAEMON_PATH/zookeeper-server-start.sh -daemon /opt/kafka/config/zookeeper.properties
        fi
        ;;
  stop)
        echo "Shutting down $DAEMON_NAME";
        $DAEMON_PATH/zookeeper-server-stop.sh
        ;;
  restart)
        $0 stop
        sleep 2
        $0 start
        ;;
  status)
        pid=`ps ax | grep -i 'org.apache.zookeeper' | grep -v grep | awk '{print $1}'`
        if [ -n "$pid" ]
          then
          echo "Zookeeper is Running as PID: $pid"
        else
          echo "Zookeeper is not Running"
        fi
        ;;
  *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
esac

exit 0
```

🌟**Change the file to executable, change ownership and start the service**

```sh
sudo chmod +x /etc/init.d/zookeeper
sudo chown root:root /etc/init.d/zookeeper
sudo systemctl daemon-reload
systemctl start zookeeper
systemctl status zookeeper --no-pager
```

## Step🌐6 — Kafka Configuration

### 1. 🟡Edit Kafka Configuration Files | All Nodes

Use the following command to backup the existing `server.properties` file (in the config directory) and create a new `server.properties` file:\
`mv /opt/kafka/config/server.properties /opt/kafka/config/server.properties_ori-backup-msi`

#### **⚠️Server:** `1` | `broker.id=1` | `advertised.listeners=PLAINTEXT://kafka-1:9092` 
Copy and paste the following into the contents of the `server.properties` and change the `broker.id` and the advertised.listeners\
`vim /opt/kafka/config/server.properties`

```sh
# change this for each broker
# example, broker.id=1 for server1, broker.id=2 for server2 and broker.id=3 for server 3
broker.id=1    
# change this to the hostname of each broker
# example advertised.listeners=PLAINTEXT://kafka-1:9092
# example, hostname -> kafka-1 for server1, hostname -> kafka-2 for server2 and hostname -> kafka-3 for server 3 
advertised.listeners=PLAINTEXT://kafka-1:9092  
# The ability to delete topics
delete.topic.enable=true
# Where logs are stored
log.dirs=/data/kafka
# default number of partitions
num.partitions=8
# default replica count based on the number of brokers
default.replication.factor=3
# to protect yourself against broker failure
min.insync.replicas=2
# logs will be deleted after how many hours
log.retention.hours=168
# size of the log files 
log.segment.bytes=1073741824
# check to see if any data needs to be deleted
log.retention.check.interval.ms=300000
# location of all zookeeper instances and kafka directory
zookeeper.connect=zookeeper-1:2181,zookeeper-2:2181,zookeeper-3:2181
# timeout for connecting with zookeeper
zookeeper.connection.timeout.ms=6000
# automatically create topics
auto.create.topics.enable=true
```

#### **⚠️Server:** `2` | `broker.id=2` | `advertised.listeners=PLAINTEXT://kafka-2:9092`
Copy and paste the following into the contents of the `server.properties` and change the `broker.id` and the advertised.listeners\
`vim /opt/kafka/config/server.properties`

```sh
# change this for each broker
# example, broker.id=1 for server1, broker.id=2 for server2 and broker.id=3 for server 3
broker.id=2    
# change this to the hostname of each broker
# example advertised.listeners=PLAINTEXT://kafka-2:9092
# example, hostname -> kafka-1 for server1, hostname -> kafka-2 for server2 and hostname -> kafka-3 for server 3 
advertised.listeners=PLAINTEXT://kafka-2:9092  
# The ability to delete topics
delete.topic.enable=true
# Where logs are stored
log.dirs=/data/kafka
# default number of partitions
num.partitions=8
# default replica count based on the number of brokers
default.replication.factor=3
# to protect yourself against broker failure
min.insync.replicas=2
# logs will be deleted after how many hours
log.retention.hours=168
# size of the log files 
log.segment.bytes=1073741824
# check to see if any data needs to be deleted
log.retention.check.interval.ms=300000
# location of all zookeeper instances and kafka directory
zookeeper.connect=zookeeper-1:2181,zookeeper-2:2181,zookeeper-3:2181
# timeout for connecting with zookeeper
zookeeper.connection.timeout.ms=6000
# automatically create topics
auto.create.topics.enable=true
```

#### **⚠️Server:** `3` | `broker.id=3` | `advertised.listeners=PLAINTEXT://kafka-3:9092`
Copy and paste the following into the contents of the `server.properties` and change the `broker.id` and the advertised.listeners\
`vim /opt/kafka/config/server.properties`

```sh
# change this for each broker
# example, broker.id=1 for server1, broker.id=2 for server2 and broker.id=3 for server 3
broker.id=3    
# change this to the hostname of each broker
# example advertised.listeners=PLAINTEXT://kafka-3:9092
# example, hostname -> kafka1 for server1, hostname -> kafka2 for server2 and hostname -> kafka3 for server 3 
advertised.listeners=PLAINTEXT://kafka-3:9092  
# The ability to delete topics
delete.topic.enable=true
# Where logs are stored
log.dirs=/data/kafka
# default number of partitions
num.partitions=8
# default replica count based on the number of brokers
default.replication.factor=3
# to protect yourself against broker failure
min.insync.replicas=2
# logs will be deleted after how many hours
log.retention.hours=168
# size of the log files 
log.segment.bytes=1073741824
# check to see if any data needs to be deleted
log.retention.check.interval.ms=300000
# location of all zookeeper instances and kafka directory
zookeeper.connect=zookeeper-1:2181,zookeeper-2:2181,zookeeper-3:2181
# timeout for connecting with zookeeper
zookeeper.connection.timeout.ms=6000
# automatically create topics
auto.create.topics.enable=true
```

### 2. 🟡Create the Kafka Service | The `init.d` scripts to start and stop Kafka service

`vim /etc/init.d/kafka`

```sh
#!/bin/bash
#/etc/init.d/kafka
DAEMON_PATH=/opt/kafka/bin
DAEMON_NAME=kafka
# Check that networking is up.
#[ ${NETWORKING} = "no" ] && exit 0

PATH=$PATH:$DAEMON_PATH

# See how we were called.
case "$1" in
  start)
        # Start daemon.
        pid=`ps ax | grep -i 'kafka.Kafka' | grep -v grep | awk '{print $1}'`
        if [ -n "$pid" ]
          then
            echo "Kafka is already running"
        else
          echo "Starting $DAEMON_NAME"
          $DAEMON_PATH/kafka-server-start.sh -daemon /opt/kafka/config/server.properties
        fi
        ;;
  stop)
        echo "Shutting down $DAEMON_NAME"
        $DAEMON_PATH/kafka-server-stop.sh
        ;;
  restart)
        $0 stop
        sleep 2
        $0 start
        ;;
  status)
        pid=`ps ax | grep -i 'kafka.Kafka' | grep -v grep | awk '{print $1}'`
        if [ -n "$pid" ]
          then
          echo "Kafka is Running as PID: $pid"
        else
          echo "Kafka is not Running"
        fi
        ;;
  *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
esac

exit 0
```

🌟**Change the file to executable, change ownership and start the service**

```sh
sudo chmod +x /etc/init.d/kafka
sudo chown root:root /etc/init.d/kafka
sudo systemctl daemon-reload
systemctl start kafka
systemctl status kafka --no-pager
```


### Step🌐7 — **Create `/etc/rc.local` to run `Kafka` and `Zookeeper` daemon services at boot time**

`vim  /etc/rc.local`

```sh
#!/bin/bash

# Start Kafka first
systemctl start zookeeper

# Wait for 10 seconds to allow Zookeeper to fully start
sleep 10

# Start Kafka after Zookeeper is up
systemctl start kafka

exit 0
```

`chmod +x /etc/rc.local`

🌟**Now, `reboot` the system, Zookeeper and Kafka should start automatically. You can check if it's running by using after booting**

`systemctl status kafka --no-pager`\
`systemctl status zookeeper --no-pager`


### Step🌐8 — Testing the Kafka Installation

🌟**Create a Topic**

```sh
/opt/kafka/bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

🌟**Verify that the topic has been created**

```sh
/opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

🌟**Produce a Message to the Topic**

```sh
/opt/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test-topic
```
Once this command is executed, you should be able to type messages into the console. Each line you type will be sent as a message to the Kafka topic.\

> **`Hello, Kafka!`**\
> **`This is a test message.`**

**`Ctrl C for Exit`**

🌟**Consume the Message from the Topic**

```sh
/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```

You should see the messages you produced, including:

**Hello, Kafka!**\
**This is a test message.**

🌟**Verify Logs for Errors (if any)**

```sh
tail -f /opt/kafka/logs/server.log
tail -f /opt/kafka/logs/zookeeper.out
tail -f /opt/kafka/logs/zookeeper-gc.log
```

---

### Step🌐9 — Kafka integration with Applications

- Key Information You Need to Provide to the Developers

  **1. Kafka Broker URL(s)**\
  Kafka clients (producers and consumers) need to know the broker URL(s) to connect to the Kafka cluster. This typically includes:

  - The hostname or IP address of one or more Kafka broker nodes.
  - The port that Kafka is listening on (default is 9092).

    **Example Kafka Broker URL:**
    
    ```sh
    kafka-1:9092,kafka-2:9092,kafka-3:9092
    ```
  **Note:** The above method for integrating Kafka with applications should be used if the Kafka cluster is set up **without** `Kafka REST Proxy` and `Kafka Schema Registry`.


  **2. Topic Names**\
  Kafka clients need to know the topic(s) they’ll be producing or consuming messages to/from. If your developers have specific topics in mind, provide them with the list of topics that have been created, or allow them to 
  create new topics.

  **View running kafka topic**
  ```sh
  /opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
  ```
  **Example:**
  
  ```
  test-topic
  user-events
  payment-transactions
  ```

  **3. Security and Authentication Details (if applicable)**\
  SSL certificates, SASL credentials, and ACL permissions

  **Example security info:**
  ```
  SSL enabled: true
  SASL mechanism: SCRAM-SHA-256
  User: dev_user
  Password: dev_password
  ```

  **4. Kafka Producer Connection Configuration | Java Application**

  Provide developers with the necessary configuration for the Kafka client. Depending on the programming language or Kafka client library they're using, the config will differ slightly. Here's an example in a generic   
  format:

  - **For Java** (using org.apache.kafka.clients.producer.KafkaProducer):
 
      ```
      Properties props = new Properties();
      props.put("bootstrap.servers", "kafka-1:9092,kafka-2:9092,kafka-3:9092");
      props.put("acks", "all");
      props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
      props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
      
      // Create a producer
      KafkaProducer<String, String> producer = new KafkaProducer<>(props);
      ```
   
  **5. Kafka Consumer Configuration**
  
  - **For Python**

      ```
      from confluent_kafka import Consumer
      
      conf = {
          'bootstrap.servers': 'kafka-broker1.example.com:9092',
          'group.id': 'user-events-consumer',
          'auto.offset.reset': 'earliest',
          'security.protocol': 'SASL_SSL',
          'sasl.mechanism': 'SCRAM-SHA-256',
          'sasl.username': 'dev_user',
          'sasl.password': 'dev_password',
      }
      
      consumer = Consumer(conf)
      consumer.subscribe(['user-events'])
      ```


https://github.com/saifulislam88/KafkaClusterSetupAndMonitoring\
https://www.cloudduggu.com/kafka/introduction/
https://amsayed.medium.com/apache-kafka-architecture-real-time-cdc-and-python-integration-1846f5e49b39
