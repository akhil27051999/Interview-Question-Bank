
## Bind Mounts

### Step 1: Create a Host Directory
```sh
bash-5.1$ mkdir -p ~/mysql-data
bash-5.1$ ls
mysql-data
bash-5.1$ docker run -d \
> --name mysql-db \
> -e MYSQL_ROOT_PASSWORD=mypasswd \
> -e MYSQL_DATABASE=testdb \
> -v ~/mysql-data:var/lib/mysql \
> mysql:8
Unable to find image 'mysql:8' locally
8: Pulling from library/mysql
ad9d782f3f87: Pull complete 
3709f9999ba9: Pull complete 
88358ea2a37f: Pull complete 
98f63f165ac1: Pull complete 
100b56c3fd28: Pull complete 
23eb2baa39f3: Pull complete 
08d96bdd8a50: Pull complete 
c68ab04cc1e9: Pull complete 
bec4df3fa85f: Pull complete 
8c32caf90444: Pull complete 
Digest: sha256:90544b3775490579867a30988d48f0215fc3b88d78d8d62b2c0d96ee9226a2b7
Status: Downloaded newer image for mysql:8
docker: Error response from daemon: invalid volume specification: '/home/user/mysql-data:var/lib/mysql': invalid mount config for type "bind": invalid mount path: 'var/lib/mysql' mount path must be absolute.
See 'docker run --help'.
```

### Step 2: Run MySQL Using Bind Mount
```sh
bash-5.1$ docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=mypasswd -e MYSQL_DATABASE=testdb -v ~/mysql-data:/var/lib/mysql mysql:8
6b016784f88acabaae09f27eb9c3473368f1a87cb83b69c487f06938cab9574c
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS          PORTS                 NAMES
6b016784f88a   mysql:8   "docker-entrypoint.s…"   About a minute ago   Up 59 seconds   3306/tcp, 33060/tcp   mysql-db
```

### Step 3: Add Sample Data
```sh
bash-5.1$ docker exec -it mysql-db /bin/sh

sh-5.1# mysql -u root -pmypasswd
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.4.7 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE testdb;
Database changed

mysql> CREATE TABLE test_table (
    ->   id INT AUTO_INCREMENT PRIMARY KEY,
    ->   name VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.03 sec)

mysql> SHOW TABLES;
+------------------+
| Tables_in_testdb |
+------------------+
| test_table       |
+------------------+
1 row in set (0.00 sec)

mysql> INSERT INTO test_table (name)
    -> VALUES ('BindMountUser');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM test_table;
+----+---------------+
| id | name          |
+----+---------------+
|  1 | BindMountUser |
+----+---------------+
1 row in set (0.00 sec)

mysql> 

mysql> exit
Bye
sh-5.1# crtl+d
exit

bash-5.1$ ls
mysql-data
bash-5.1$ ls ~/mysql-data
#ib_16384_0.dblwr      auto.cnf               ca-key.pem             ib_buffer_pool         mysql.ibd              private_key.pem        sys
#ib_16384_1.dblwr      binlog.000001          ca.pem                 ibdata1                mysql.sock             public_key.pem         testdb
#innodb_redo           binlog.000002          client-cert.pem        ibtmp1                 mysql_upgrade_history  server-cert.pem        undo_001
#innodb_temp           binlog.index           client-key.pem         mysql                  performance_schema     server-key.pem         undo_002
```

### Step 4: Stop and Remove Container

```sh
bash-5.1$ docker stop mysql-db
mysql-db
bash-5.1$ docker rm mysql-db
mysql-db
bash-5.1$ ls mysql-data/
#ib_16384_0.dblwr      #innodb_temp           binlog.000002          ca.pem                 ib_buffer_pool         mysql.ibd              performance_schema     server-cert.pem        testdb
#ib_16384_1.dblwr      auto.cnf               binlog.index           client-cert.pem        ibdata1                mysql.sock             private_key.pem        server-key.pem         undo_001
#innodb_redo           binlog.000001          ca-key.pem             client-key.pem         mysql                  mysql_upgrade_history  public_key.pem         sys                    undo_002
bash-5.1$ 
bash-5.1$
```
