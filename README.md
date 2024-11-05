Redis Replication and Sentinel Setup on RHEL 8.8
Step 1: Configure Redis Replication
Operating System: RHEL 8.8
Master: 100.110.34.20
Slaves: 100.110.34.21, 100.110.34.30

1.1 Install Redis on All Nodes
Run the following commands on all three nodes (master and slaves):

bash
Copy code
sudo yum install epel-release -y
sudo yum install redis -y
1.2 Configure Redis
On the Master Node (100.110.34.20):
Open the Redis configuration file:
bash
Copy code
sudo nano /etc/redis.conf
Update the bind setting to allow external connections:
conf
Copy code
bind 0.0.0.0
On Each Slave Node (100.110.34.21, 100.110.34.30):
Open the Redis configuration file:
bash
Copy code
sudo nano /etc/redis.conf
Add the following line to point to the master node:
conf
Copy code
replicaof 100.110.34.20 6379
1.3 Start and Enable Redis
On each node, start and enable Redis to run on boot:

bash
Copy code
sudo systemctl restart redis
sudo systemctl enable redis
Step 2: Configure Redis Sentinel
Sentinel Nodes: 100.110.34.20, 100.110.34.21, 100.110.34.30
Operating System: RHEL 8.8

2.1 System Configuration
Open /etc/sysctl.conf and set the following parameter to optimize network connections:
conf
Copy code
net.core.somaxconn = 1024
Create a directory for Redis with appropriate permissions:
bash
Copy code
sudo mkdir -p /home/redis
sudo chown redis:redis /home/redis
2.2 Configure Sentinel
Edit the Sentinel configuration file on each node:

bash
Copy code
sudo nano /etc/redis-sentinel.conf
Add or update the following settings:

conf
Copy code
# Monitor the Redis master instance
sentinel monitor mymaster 100.110.34.30 6379 2

# Authentication password for Redis master
sentinel auth-pass mymaster your_redis_password

# Logging settings
loglevel notice
logfile "/var/log/redis/redis-sentinel.log"

# Sentinel's communication port
port 26379
bind 0.0.0.0

# Directory for Sentinel persistence files
dir "/home/redis"

# Protection settings
protected-mode no

# Quorum and failover settings
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
Note: Each Sentinel node will generate a unique ID automatically on startup.

2.3 Start Sentinel on All Nodes
On each node, start the Redis Sentinel:

bash
Copy code
redis-sentinel /etc/redis-sentinel.conf
Step 3: Verify Sentinel Setup
To check the Sentinel setup and current master, run the following commands:

Check the Current Master:

bash
Copy code
redis-cli -p 26379 sentinel get-master-addr-by-name mymaster
Check Sentinel Nodes:

bash
Copy code
redis-cli -p 26379 sentinel sentinels mymaster
These commands should display the current master and all configured Sentinel nodes, excluding the node from which the command is run.

