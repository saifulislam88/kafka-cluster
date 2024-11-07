## 3-Broker Kafka Multi-Node Cluster with Zookeeper on Ubuntu 24

Pre-requisits
1.Kafka latest package
2. Ubuntu VMs(3 nodes)

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
sudo mv kafka_2.13-3.8.1 /opt/kafka
```

### Create a New Directory for Kafka and Zookeeper

```sh
sudo mkdir -p /data/kafka                                                     #A new directory for Kafka message and logs
sudo mkdir -p /data/zookeeper                                                 #It is snapshot and data directory for Zookeeper
```


## 🚀Zookeeper Configuration

### 1. 🟡Create a Zookeeper Uniq one ID on each VM for Zookeeper | Specify Uniq an ID

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

- Change the file to executable, change ownership and start the service

`sudo chmod +x /etc/init.d/zookeeper`
`sudo chown root:root /etc/init.d/zookeeper`
`sudo service zookeeper start` or `systemctl start zookeeper`
`sudo service zookeeper status` or `systemctl status zookeeper`

- 🌟Create `/etc/rc.local` to run `Zookeeper` at boot

`vim  /etc/rc.local`

```sh
#!/bin/bash

# Start Zookeeper first
systemctl start zookeeper

# Wait for 10 seconds to allow Zookeeper to fully start
sleep 10

# Start Kafka after Zookeeper is up
systemctl start kafka

exit 0
```
`chmod +x /etc/rc.local`

- 🌟Now, `reboot` the system, Zookeeper should start automatically. You can check if it's running by using
`systemctl status zookeeper`
