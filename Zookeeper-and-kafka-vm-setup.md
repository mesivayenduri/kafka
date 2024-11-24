## Create 3 Vms with below settings
-   Make sure to use bridged network

## Install Below packages in all 3 VMs
sudo apt-get update && \
      sudo apt-get -y install wget ca-certificates zip net-tools vim nano tar netcat

## Java Open JDK 8
sudo apt-get -y install openjdk-11-jdk
java -version

## Disable RAM Swap - can set to 0 on certain Linux distro
sudo sysctl vm.swappiness=1
echo 'vm.swappiness=1' | sudo tee --append /etc/sysctl.conf

## Firewall
-   Make sure open firewall for below ports in all 3 machines.
    -   22 for SSH
    -   3888 for Zookeeper
    -   2888 for Zookeeper
### Use below commads to open ports on firewall
    -   sudo ufw allow ssh
    -   sudo ufw enable
    -   sudo ufw allow 2888/tcp
    -   sudo ufw allow 3888/tcp
    -   sudo ufw allow 2181/tcp
    -   sudo ufw reload
    -   sudo ufw status

## Update /etc/hosts
-   Update /etc/hosts as below in all 3 machines
echo "192.168.1.4 zookeeper1
192.168.1.4 kafka1
192.168.1.8 zookeeper2
192.168.1.8 kafka2
192.168.1.10 zookeeper3
192.168.1.10 kafka3" | sudo tee --append /etc/hosts

Note: make sure to update the IPs according to your needs.

## Download the Latest Kafka
-   wget https://archive.apache.org/dist/kafka/kafka_2.13-3.9.0.tgz
-   tar -xvzf kafka_2.13-3.9.0.tgz
-   rm kafka_2.13-3.9.0.tgz
-   mv kafka_2.13-3.9.0 kafka
-   cd kafka

## Set PATH in all machines
-   sudo nano ~/.bashrc
-   Add PATH=/home/vboxuser/kafka/bin:$PATH

## Update config/zookeeper.properties as below
```bash
# the location to store the in-memory database snapshots and, unless specified otherwise, the transaction log of updates to the database.
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
# the basic time unit in milliseconds used by ZooKeeper. It is used to do heartbeats and the minimum session timeout will be twice the tickTime.
tickTime=2000
# The number of ticks that the initial synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# zoo servers
# these hostnames such as `zookeeper-1` come from the /etc/hosts file
server.1=zookeeper1:2888:3888
server.2=zookeeper2:2888:3888
server.3=zookeeper3:2888:3888
```

## Create /data/zookeer directory and make sure to change ownership
-   sudo mkdir -p /data/zookeeper
-   sudo chown vboxuser:vboxuser /data/*  (as my current user is vboxuser)

## Create myid file in each server with respective ID
-   echo "1" > /data/zookeeper/myid ( server 1)
-   echo "2" > /data/zookeeper/myid ( server 2)
-   echo "3" > /data/zookeeper/myid ( server 3)

## Now start zookeeper
-   bin/zookeeper-server-start.sh config/zookeeper.properties


# Testing Zookeeper install
# Start Zookeeper in the background
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
bin/zookeeper-shell.sh localhost:2181
ls /

## demonstrate the use of a 4 letter word
echo "ruok" | nc localhost 2181 ; echo

If you face the following error "ruok is not executed because it is not in the whitelist",
Add the following property in zookeeper.properties.

```bash zookeeper.properties

4lw.commands.whitelist=*

```

# Install Zookeeper boot scripts
sudo nano /etc/init.d/zookeeper
sudo chmod +x /etc/init.d/zookeeper
sudo chown root:root /etc/init.d/zookeeper
# you can safely ignore the warning
sudo update-rc.d zookeeper defaults
# stop zookeeper
sudo service zookeeper stop
# verify it's stopped
nc -vz localhost 2181
# start zookeeper
sudo service zookeeper start
# verify it's started
nc -vz localhost 2181
echo "ruok" | nc localhost 2181 ; echo
# check the logs
cat logs/zookeeper.out



# ENABLING ZOONAVIGATOR

```bash
#!/bin/bash
sudo apt-get update
```

## Install packages to allow apt to use a repository over HTTPS:
```bash
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# set up the stable repository.
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# install docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-compose

# give ubuntu permissions to execute docker
sudo usermod -aG docker $(whoami)
# log out
exit
# log back in

# make sure docker is working
docker run hello-world

# Add hosts entries (mocking DNS) - put relevant IPs here
echo "172.31.9.1 kafka1
172.31.9.1 zookeeper1
172.31.19.230 kafka2
172.31.19.230 zookeeper2
172.31.35.20 kafka3
172.31.35.20 zookeeper3" | sudo tee --append /etc/hosts
```

NOTE:  Make sure to replace the IPs with actual IPs

## Enable firewall
```bash
sudo ufw enable
sudo ufw allow 8001
sudo ufw reload
sudo ufw status
```

## Deploy Zoonavigator

```bash
nano zoonavigator-docker-compose.yml

version: '2'
services:
  # https://github.com/elkozmon/zoonavigator
  zoonavigator:
    image: elkozmon/zoonavigator:latest
    container_name: zoonavigator
    network_mode: host
    environment:
      HTTP_PORT: 8001
    restart: always

# Make sure port 8001 is opened on the instance security group
# copy the zookeeper/zoonavigator-docker-compose.yml file
docker-compose -f zoonavigator-docker-compose.yml up -d
```

NOTE: Now you can access zoonavigator within VM and also from your windows host machine