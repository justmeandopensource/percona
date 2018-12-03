## Percona-XtraDB-Cluster
### Install instructions for Ubuntu 18

### Assumptions
|Role|machine name|IP address|Memory|Operating System|
|-|-|-|-|-|
|master node 1|ubuntuvm01|172.42.42.101|1G|Ubuntu 18|
|master node 2|ubuntuvm02|172.42.42.102|1G|Ubuntu 18|

### On First node
##### Add Percona Repository
```
wget https://repo.percona.com/apt/percona-release_0.1-6.$(lsb_release -sc)_all.deb
dpkg -i percona-release_0.1-6.$(lsb_release -sc)_all.deb
```
##### Install Percona-XtraDB-Cluster
```
apt-get update
apt-get install -y percona-xtradb-cluster-57
systemctl stop mysql
```
##### Configure Replication Settings
```
cat >>/etc/mysql/my.cnf<<EOF
[mysqld]
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name=democluster
wsrep_cluster_address=gcomm://
wsrep_node_name=centosvm01
wsrep_node_address=172.42.42.101
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=repuser:reppassword
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
EOF
```
##### Bootstrap/Initialize the Cluster
```
systemctl start mysql
```
##### Create Replication User
```
mysql -uroot -p -e "create user repuser@localhost identified by 'reppassword'"
mysql -uroot -p -e "grant reload, replication client, process, lock tables on *.* to repuser@localhost"
mysql -uroot -p -e "flush privileges"
```
##### Update Replication configuration
```
sed -i 's/^wsrep_cluster_address=.*/wsrep_cluster_address=gcomm:\/\/172.42.42.101,172.42.42.102/' /etc/mysql/my.cnf
```

### On Second node
##### Add Percona Repository
```
wget https://repo.percona.com/apt/percona-release_0.1-6.$(lsb_release -sc)_all.deb
dpkg -i percona-release_0.1-6.$(lsb_release -sc)_all.deb
```
##### Install Percona-XtraDB-Cluster
```
apt-get update
apt-get install -y percona-xtradb-cluster-57
systemctl stop mysql
```
##### Configure Replication Settings
```
cat >>/etc/mysql/my.cnf<<EOF
[mysqld]
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name=democluster
wsrep_cluster_address=gcomm://172.42.42.101,172.42.42.102
wsrep_node_name=centosvm02
wsrep_node_address=172.42.42.102
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=repuser:reppassword
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
EOF
```
##### Start mysql to join the cluster
```
systemctl start mysql
```
