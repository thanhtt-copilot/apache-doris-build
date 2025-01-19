# Install FoundationDB
1. Download and unzip: https://doris.apache.org/download apache-doris-3.0.3-rc04-bin-x64
2. Create data dir

mkdir /mnt/foundationdb
mkdir /fdbhome
sudo chown -R ec2-user:ec2-user /fdbhome
sudo chown -R ec2-user:ec2-user  /mnt/foundationdb

3. Edit file fdb_vars.sh

cd /apache-doris-build/apache-doris-3.0.3-rc04-bin-x64/tools/fdb

./fdb_ctl.sh deploy
./fdb_ctl.sh start

Output:
Run FDB monitor ...
Try create database in fdb single
Database created
Start fdb success, and you can set conf for MetaService:
fdb_cluster = mycluster:ra7eOp7x@10.9.18.199:4500

# start ms:

cd /apache-doris-build/apache-doris-3.0.3-rc04-bin-x64/ms
edit doris_cloud.conf:
fdb_cluster = mycluster:ra7eOp7x@10.9.18.199:4500

export JAVA_HOME=${path_to_jdk_17}
bin/start.sh --daemon

Output:
pid file existed but process not alive, remove it, pid=9727
LIBHDFS3_CONF=
starts doris_cloud with args: 
wait and check doris_cloud start successfully
successfully started brpc listening on port=5000 time_elapsed_ms=58
doris_cloud start successfully

# start fe with the master role

cd /be/

edit below: (Modify arrow_flight_sql_port in fe/conf/fe.conf to an available port, such as 9090.
)
arrow_flight_sql_port = 9090
deploy_mode = cloud
cluster_id = 17261811
meta_service_endpoint = 127.0.0.1:5000

# add be to cluster


# add s3 vault
CREATE STORAGE VAULT IF NOT EXISTS s3_vault
    PROPERTIES (
    "type"="S3",
    "s3.endpoint"="s3.ap-southeast-1.amazonaws.com",
    "s3.access_key" = "AKIAYFAE77M6IBACIKND",
    "s3.secret_key" = "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",
    "s3.region" = "ap-southeast-1",
    "s3.root.path" = "ssb_sf1_p2_s3",
    "s3.bucket" = "demo-apache-doris",
    "provider" = "S3"
    );

SET s3_vault AS DEFAULT STORAGE VAULT


DROP DATABASE IF EXISTS demo_db;
create database demo_db;
use demo_db;

CREATE TABLE IF NOT EXISTS thanhttt (
  s_suppkey int(11) NOT NULL COMMENT ""
)
UNIQUE KEY (s_suppkey)
DISTRIBUTED BY HASH(s_suppkey) BUCKETS 1
PROPERTIES (
"replication_num" = "1",
"storage_vault_name" = "s3_vault"
);

insert into thanhttt values(1),(2);
