# -*- mode: ruby -*-
# # vi: set ft=ruby :
## RESOURCES
# http://stackoverflow.com/questions/2366018/how-to-re-sync-the-mysql-db-if-master-and-slave-have-different-database-incase-o

VAGRANTFILE_API_VERSION = "2"

def configure_server(cluster_address)
setup = <<-SCRIPT
export DEBIAN_FRONTEND=noninteractive
debconf-set-selections <<< 'mariadb-server-5.5 mysql-server/root_password password password'
debconf-set-selections <<< 'mariadb-server-5.5 mysql-server/root_password_again password password'
#apt-get update
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db
sudo add-apt-repository 'deb http://mirror.jmu.edu/pub/mariadb/repo/5.5/ubuntu precise main'
sudo apt-get update
sudo apt-get install mariadb-galera-server galera rsync -y

echo "[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
innodb_locks_unsafe_for_binlog=1
query_cache_size=0
query_cache_type=0
bind-address=0.0.0.0
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_name=\"my_wsrep_cluster\"
wsrep_cluster_address=\"gcomm://#{cluster_address.join(',')}\"
wsrep_slave_threads=1
wsrep_certify_nonPK=1
wsrep_max_ws_rows=131072
wsrep_max_ws_size=1073741824
wsrep_debug=0
wsrep_convert_LOCK_to_trx=0
wsrep_retry_autocommit=1
wsrep_auto_increment_control=1
wsrep_drupal_282555_workaround=0
wsrep_causal_reads=0
wsrep_notify_cmd=
wsrep_sst_method=rsync
wsrep_sst_auth=root:
" > /etc/mysql/conf.d/cluster.cnf
/etc/init.d/mysql restart
SCRIPT
end

def configure_node(address)
node = <<-SCRIPT
echo "[mysqld]
# Galera Node Configuration
wsrep_node_address=#{address}" > /etc/mysql/conf.d/node.cnf
SCRIPT
end


CLUSTER_NODES = {
    "mm01" => "192.168.56.141",
    "mm02" => "192.168.56.142",
    "mm03" => "192.168.56.143"
}


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
#  config.cache.enable :apt #cachier
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  CLUSTER_NODES.each do |name, address|
    config.vm.define name do |node|
      node_address =   address
      node.vm.hostname = name
      node.vm.network :private_network, ip: address
      node.vm.provision "shell", inline: configure_server(CLUSTER_NODES.values)
      node.vm.provision "shell", inline: configure_node(address)
    end
  end
end
