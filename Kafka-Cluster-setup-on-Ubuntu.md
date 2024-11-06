## Kafka Cluster Setup on Ubuntu with 3 Brokers | Kafka Multi Nodes Cluster Setup





### Install latest Java(OpenJDK) and Disable RAM Swap
    # sudo apt update
    # sudo apt search openjdk
    # sudo apt install default-jdk
    # java -version
    # swapoff -a
    # sudo sed -i '/ swap / s/^/#/' /etc/fstab
    # echo 'vm.swappiness=1' | sudo tee --append /etc/sysctl.conf
    # cat /etc/sysctl.conf |grep 'swappiness'                                        #Output "vm.swappiness=1"



### Download the latest Kafka binaries

$ wget https://downloads.apache.org/kafka/3.8.1/kafka_2.13-3.8.1.tgz
$ tar -xvf kafka_2.13-3.8.1.tgz
$ sudo mv kafka_2.13-3.8.1.tgz /opt/kafka
$ sudo mkdir -p /data/kafka                                                         #create folder to store data


### Create a New Directory for Kafka and Zookeeper

sudo mkdir -p /data/kafka                                                           # A new directory for Kafka message and logs
sudo mkdir -p /data/zookeeper                                                       # It is snapshot and data directory for Zookeeper
