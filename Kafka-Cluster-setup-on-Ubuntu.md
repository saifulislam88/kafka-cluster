## Kafka Cluster Setup on Ubuntu with 3 Brokers | Kafka Multi Nodes Cluster Setup




### Add custom hostnames to /etc/hosts file
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


### Install latest Java(OpenJDK) and Disable RAM Swap

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


### Download the latest Kafka binaries

```sh
wget https://downloads.apache.org/kafka/3.8.1/kafka_2.13-3.8.1.tgz
tar -xvf kafka_2.13-3.8.1.tgz
sudo mv kafka_2.13-3.8.1.tgz /opt/kafka
sudo mkdir -p /data/kafka                                                     #create folder to store data
```

### Create a New Directory for Kafka and Zookeeper

```sh
sudo mkdir -p /data/kafka                                                     #A new directory for Kafka message and logs
sudo mkdir -p /data/zookeeper                                                 #It is snapshot and data directory for Zookeeper
```

### Create a Zookeeper Uniq one ID on each VM for Zookeeper | Specify Uniq an ID

#`"1"` to specify Kafka-Zookeeper server #1:
```sh
echo "1" > /data/zookeeper/myid                                                                
```

#`"2"` to specify Kafka-Zookeeper Server #2:
```sh
echo "2" > /data/zookeeper/myid                                                                
```

#`"3"` to specify Kafka-Zookeeper server #3:
```sh
echo "3" > /data/zookeeper/myid                                                                
```




