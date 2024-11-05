Step1: Configure replication

OS RHEL 8.8

master  100.110.34.20
Slave   100.110.34.21 & 100.110.34.30

===  on all 3 nodes ====
sudo yum install epel-release -y
sudo yum install redis -y


===================================
on master in /etc/redis.conf
replace "bind 127.0.0.1" to "bind  0.0.0.0"

on slave in /etc/redis.conf add following Line
replicaof 100.110.34.20 6379

========================================
sudo systemctl restart redis
sudo systemctl enable redis
===============================

Step2: Configure sentinel

Sentinel nodes  100.110.34.20, 100.110.34.21 & 100.110.34.30
OS RHEL 8.8

set net.core.somaxconn = 1024  in /etc/sysctl.conf
Ensure a create a directory /home/redis with permission  redis:redis

edit /etc/redis-sentinel.conf file

# Unique Sentinel ID - This will be generated automatically for each instance on start
#sentinel myid <unique_id_for_each_node>  # This line can be removed; Sentinel will assign an ID

# Monitor the Redis master instance
# Replace "100.110.34.30" with the actual IP address of the Redis master
sentinel monitor mymaster 100.110.34.30 6379 2

# Authentication password for Redis master
# Replace "your_redis_password" with the actual Redis password
sentinel auth-pass mymaster your_redis_password

# Logging settings
loglevel notice
logfile "/var/log/redis/redis-sentinel.log"

# Sentinel's communication port
port 26379
bind 0.0.0.0

# Directory for Sentinel persistence files
dir "/home/redis"

# Protection settings - "protected-mode" should be off for accessibility
protected-mode no

# Quorum setting - set to 2 for this setup
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000


start sentinel in all hosts

$redis-sentinel /etc/redis-sentinel in all nodes

Check following

redis-cli -p 26379 sentinel get-master-addr-by-name mymaster  (will give you current master)
redis-cli -p 26379 sentinel sentinels mymaster  (will give all sentinels, this will show all nodes but itself)
