---
description: >-
  Complete guide for MySQL and PostgreSQL database operations from a DevOps
  perspective.
---

# Database DevOps Commands Reference

***

### Table of Contents

1. Connection & Access
2. User & Permission Management
3. Database Operations
4. Backup & Restore
5. Performance Monitoring
6. Replication
7. Maintenance
8. Troubleshooting

***

### Connection & Access

#### MySQL

**Connect to Local Server**

```bash
mysql -u username -p
mysql -u root -p
```

**Connect to Remote Server**

```bash
mysql -h hostname -u username -p
mysql -h 192.168.1.100 -u devops -p -P 3306
```

**Connect with Socket**

```bash
mysql -u root --socket=/var/run/mysqld/mysqld.sock
```

**Connect to Specific Database**

```bash
mysql -u username -p database_name
```

**Test Connection Without Password Prompt**

```bash
mysql -u root -p'password' -h localhost
```

#### PostgreSQL

**Connect to Local Server**

```bash
psql -U username
psql -U postgres
```

**Connect to Remote Server**

```bash
psql -h hostname -U username -d database_name
psql -h 192.168.1.100 -U postgres -d mydb -p 5432
```

**Connect with Connection String**

```bash
psql postgresql://username:password@hostname:5432/database_name
```

**List Available Databases**

```bash
psql -U postgres -l
```

**Connection via .pgpass (Password File)**

```bash
echo "hostname:port:database:username:password" >> ~/.pgpass
chmod 600 ~/.pgpass
```

***

### User & Permission Management

#### MySQL

**Create User**

```sql
CREATE USER 'devops'@'localhost' IDENTIFIED BY 'password';
CREATE USER 'devops'@'192.168.1.%' IDENTIFIED BY 'password';
CREATE USER 'devops'@'%' IDENTIFIED BY 'password';
```

**Change User Password**

```sql
ALTER USER 'devops'@'localhost' IDENTIFIED BY 'new_password';
SET PASSWORD FOR 'devops'@'localhost' = 'new_password';
```

**Grant Privileges**

```sql
GRANT ALL PRIVILEGES ON database_name.* TO 'devops'@'localhost';
GRANT SELECT, INSERT, UPDATE ON database_name.* TO 'devops'@'localhost';
GRANT ALL PRIVILEGES ON *.* TO 'devops'@'%' WITH GRANT OPTION;
```

**Revoke Privileges**

```sql
REVOKE ALL PRIVILEGES ON database_name.* FROM 'devops'@'localhost';
REVOKE INSERT, UPDATE ON database_name.* FROM 'devops'@'localhost';
```

**List Users**

```sql
SELECT User, Host FROM mysql.user;
```

**Delete User**

```sql
DROP USER 'devops'@'localhost';
```

**Apply Changes**

```sql
FLUSH PRIVILEGES;
```

#### PostgreSQL

**Create User (Role)**

```sql
CREATE USER devops WITH PASSWORD 'password';
CREATE USER devops WITH ENCRYPTED PASSWORD 'password';
```

**Create Superuser**

```sql
CREATE USER devops WITH SUPERUSER PASSWORD 'password';
```

**Change User Password**

```sql
ALTER USER devops WITH PASSWORD 'new_password';
```

**Grant Privileges**

```sql
GRANT ALL PRIVILEGES ON DATABASE mydb TO devops;
GRANT CONNECT ON DATABASE mydb TO devops;
GRANT USAGE ON SCHEMA public TO devops;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO devops;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO devops;
```

**Revoke Privileges**

```sql
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM devops;
REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA public FROM devops;
```

**List Users/Roles**

```sql
\du
SELECT * FROM pg_user;
```

**Delete User**

```sql
DROP USER devops;
DROP USER IF EXISTS devops;
```

**Grant Default Privileges**

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO devops;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO devops;
```

***

### Database Operations

#### MySQL

**Show Databases**

```sql
SHOW DATABASES;
```

**Create Database**

```sql
CREATE DATABASE mydb;
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**Select Database**

```sql
USE mydb;
```

**Drop Database**

```sql
DROP DATABASE mydb;
DROP DATABASE IF EXISTS mydb;
```

**Show Database Size**

```sql
SELECT table_schema AS 'Database', 
       ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables 
GROUP BY table_schema;
```

**Show Table List**

```sql
SHOW TABLES;
SHOW TABLES FROM database_name;
```

**Show Table Structure**

```sql
DESCRIBE table_name;
SHOW CREATE TABLE table_name;
```

**Show Table Size**

```sql
SELECT 
  table_name,
  ROUND(((data_length + index_length) / 1024 / 1024), 2) AS size_mb
FROM information_schema.TABLES
WHERE table_schema = 'database_name'
ORDER BY size_mb DESC;
```

#### PostgreSQL

**List Databases**

```sql
\l
SELECT datname FROM pg_database WHERE datistemplate = false;
```

**Create Database**

```sql
CREATE DATABASE mydb;
CREATE DATABASE mydb OWNER devops;
CREATE DATABASE mydb TEMPLATE template0 ENCODING 'UTF8';
```

**Switch Database**

```sql
\c database_name
```

**Drop Database**

```sql
DROP DATABASE mydb;
DROP DATABASE IF EXISTS mydb;
DROP DATABASE mydb WITH (FORCE);
```

**Show Database Size**

```sql
SELECT datname, pg_size_pretty(pg_database_size(datname)) 
FROM pg_database 
WHERE datistemplate = false;
```

**List Tables**

```sql
\dt
SELECT table_name FROM information_schema.tables 
WHERE table_schema = 'public';
```

**Show Table Structure**

```sql
\d table_name
```

**Show Table Size**

```sql
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname != 'information_schema'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

***

### Backup & Restore

#### MySQL

**Backup Full Database**

```bash
mysqldump -u root -p database_name > backup.sql
mysqldump -u root -p --all-databases > full_backup.sql
```

**Backup with Options**

```bash
# Include routines and triggers
mysqldump -u root -p --routines --triggers database_name > backup.sql

# Single file, no locks
mysqldump -u root -p --single-transaction database_name > backup.sql

# Compress backup
mysqldump -u root -p database_name | gzip > backup.sql.gz

# With timestamp
mysqldump -u root -p database_name > backup_$(date +%Y%m%d_%H%M%S).sql
```

**Backup Specific Tables**

```bash
mysqldump -u root -p database_name table1 table2 > backup.sql
```

**Restore Database**

```bash
mysql -u root -p database_name < backup.sql
```

**Restore Compressed Backup**

```bash
gunzip < backup.sql.gz | mysql -u root -p database_name
```

**Backup Binary Logs**

```bash
mysqlbinlog --start-datetime="2024-01-01 00:00:00" \
            --stop-datetime="2024-01-02 00:00:00" \
            /var/log/mysql/mysql-bin.000001 > binlog_backup.sql
```

#### PostgreSQL

**Backup Database (SQL Format)**

```bash
pg_dump -U postgres database_name > backup.sql
pg_dump -U postgres -h localhost database_name > backup.sql
```

**Backup Database (Custom Format - Recommended)**

```bash
pg_dump -U postgres -Fc database_name > backup.dump
pg_dump -U postgres -Fc -v database_name > backup.dump
```

**Backup All Databases**

```bash
pg_dumpall -U postgres > full_backup.sql
```

**Backup with Compression**

```bash
pg_dump -U postgres -Fc database_name | gzip > backup.dump.gz
```

**Backup Specific Schema**

```bash
pg_dump -U postgres -n schema_name database_name > backup.sql
```

**Backup Specific Tables**

```bash
pg_dump -U postgres -t table1 -t table2 database_name > backup.sql
```

**Restore Database**

```bash
psql -U postgres database_name < backup.sql
```

**Restore Custom Format**

```bash
pg_restore -U postgres -d database_name backup.dump
pg_restore -U postgres -d database_name -v backup.dump
```

**Restore with Specific Options**

```bash
# Disable triggers during restore
pg_restore -U postgres -d database_name --disable-triggers backup.dump

# Create database if not exists
pg_restore -U postgres -C -d postgres backup.dump
```

**Point-in-Time Recovery**

```bash
# Backup WAL files
mkdir -p /backup/wal_archive

# Configure postgresql.conf
# wal_level = replica
# archive_mode = on
# archive_command = 'cp %p /backup/wal_archive/%f'
```

***

### Performance Monitoring

#### MySQL

**Show Server Status**

```sql
SHOW STATUS;
SHOW STATUS LIKE 'Threads%';
SHOW STATUS LIKE '%connection%';
```

**Show Processlist (Active Queries)**

```sql
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;
```

**Kill Slow Query**

```sql
KILL 123;
KILL QUERY 123;
```

**Check Query Performance**

```sql
SET SESSION sql_mode = 'TRADITIONAL';
EXPLAIN SELECT * FROM table_name WHERE condition;
EXPLAIN FORMAT=JSON SELECT * FROM table_name;
```

**Slow Query Log**

```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;
SHOW VARIABLES LIKE 'slow_query_log%';
```

**Check Database Variables**

```sql
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES;
SHOW VARIABLES LIKE '%buffer%';
```

**Monitor Replication Status**

```sql
SHOW SLAVE STATUS\G
SHOW REPLICA STATUS\G
```

**Check Table Statistics**

```sql
SELECT 
  table_name,
  table_rows,
  ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)'
FROM information_schema.TABLES
WHERE table_schema = 'database_name';
```

#### PostgreSQL

**Show Server Status**

```sql
SELECT version();
SHOW server_version;
```

**Active Connections/Queries**

```sql
SELECT pid, usename, application_name, state, query 
FROM pg_stat_activity;

SELECT pid, usename, duration, query
FROM pg_stat_statements
ORDER BY duration DESC LIMIT 10;
```

**Kill Connection**

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity 
WHERE pid <> pg_backend_pid() AND usename = 'username';
```

**Query Execution Plan**

```sql
EXPLAIN SELECT * FROM table_name WHERE condition;
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM table_name;
EXPLAIN (FORMAT JSON) SELECT * FROM table_name;
```

**Cache Hit Ratio**

```sql
SELECT 
  sum(heap_blks_read) as heap_read, 
  sum(heap_blks_hit) as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
```

**Table Statistics**

```sql
SELECT schemaname, tablename, n_live_tup, n_dead_tup, last_vacuum
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;
```

**Database Size**

```sql
SELECT pg_size_pretty(pg_database_size('database_name'));
```

**Index Usage Statistics**

```sql
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

**Long Running Queries**

```sql
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';
```

***

### Replication

#### MySQL - Master/Slave Setup

**On Master Server**

```sql
-- Enable binary logging
SHOW VARIABLES LIKE 'log_bin';
SHOW MASTER STATUS;

-- Create replication user
CREATE USER 'repl'@'slave_ip' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'slave_ip';
```

**On Slave Server**

```sql
-- Configure slave
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=154;

-- Start slave
START SLAVE;

-- Check status
SHOW SLAVE STATUS\G
```

**Monitor Replication**

```sql
SHOW SLAVE STATUS\G
```

#### PostgreSQL - Streaming Replication

**On Primary Server**

```bash
# postgresql.conf settings:
# wal_level = replica
# max_wal_senders = 3
# wal_keep_size = 1GB
# hot_standby = on

# Create replication user
sudo -u postgres createuser replicator -P --replication
```

**On Standby Server**

```bash
# Stop PostgreSQL
sudo systemctl stop postgresql

# Base backup from primary
pg_basebackup -h primary_ip -D /var/lib/postgresql/12/main -U replicator -P -v -W -X stream -C -S slot1

# Create recovery.conf (PostgreSQL < 12)
# Or standby.signal (PostgreSQL 12+)
touch /var/lib/postgresql/12/main/standby.signal

# Configure postgresql.conf for standby
# primary_conninfo = 'host=primary_ip user=replicator password=password'

# Start PostgreSQL
sudo systemctl start postgresql
```

**Monitor Replication**

```sql
-- On Primary
SELECT * FROM pg_stat_replication;
SELECT * FROM pg_replication_slots;

-- On Standby
SELECT pg_last_xlog_receive_location();
SELECT pg_last_xact_replay_timestamp();
```

***

### Maintenance

#### MySQL

**Optimize Tables**

```sql
OPTIMIZE TABLE table_name;
OPTIMIZE TABLE table1, table2, table3;
```

**Repair Tables**

```sql
REPAIR TABLE table_name;
REPAIR TABLE table_name QUICK;
```

**Check Table Integrity**

```sql
CHECK TABLE table_name;
CHECK TABLE table1, table2 QUICK;
```

**Analyze Tables**

```sql
ANALYZE TABLE table_name;
ANALYZE TABLE table1, table2;
```

**Flush Caches**

```sql
FLUSH QUERY CACHE;
FLUSH TABLES;
FLUSH HOSTS;
```

**Update Statistics**

```sql
ANALYZE TABLE database_name;
```

#### PostgreSQL

**Vacuum (Cleanup)**

```sql
VACUUM;
VACUUM table_name;
VACUUM ANALYZE;
VACUUM ANALYZE table_name;
```

**Full Vacuum (Locks Table)**

```sql
VACUUM FULL;
VACUUM FULL ANALYZE;
```

**Reindex**

```sql
REINDEX INDEX index_name;
REINDEX TABLE table_name;
REINDEX DATABASE database_name;
```

**Analyze Statistics**

```sql
ANALYZE;
ANALYZE table_name;
```

**Cluster Table**

```sql
CLUSTER table_name USING index_name;
```

**Maintenance Commands via CLI**

```bash
# Analyze database
vacuumdb -U postgres -d database_name -v

# Full vacuum
vacuumdb -U postgres -d database_name -f -v

# Reindex
reindexdb -U postgres -d database_name -v
```

***

### Troubleshooting

#### MySQL

**Connection Refused**

```bash
# Check if service is running
sudo systemctl status mysql
sudo systemctl start mysql

# Check listening ports
sudo netstat -tlnp | grep mysql
sudo lsof -i :3306

# Check MySQL error log
tail -f /var/log/mysql/error.log
```

**High Memory Usage**

```sql
-- Check buffer pool size
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- Show table cache
SHOW VARIABLES LIKE 'table_open_cache';
```

**Slow Queries**

```bash
# Enable slow query log
mysql -u root -p -e "SET GLOBAL slow_query_log = 'ON';"
mysql -u root -p -e "SET GLOBAL long_query_time = 1;"

# Analyze slow log
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log
```

**Disk Space Issues**

```bash
# Check MySQL data directory size
du -sh /var/lib/mysql

# Find largest tables
SELECT table_name, ROUND(((data_length + index_length) / 1024 / 1024 / 1024), 2) size_gb
FROM information_schema.TABLES
WHERE table_schema = 'database_name'
ORDER BY size_gb DESC;
```

#### PostgreSQL

**Connection Refused**

```bash
# Check if service running
sudo systemctl status postgresql
sudo systemctl start postgresql

# Check listening ports
sudo netstat -tlnp | grep postgres
sudo lsof -i :5432

# Check logs
tail -f /var/log/postgresql/postgresql.log
```

**Authentication Failed**

```bash
# Check pg_hba.conf
sudo cat /etc/postgresql/12/main/pg_hba.conf

# Verify .pgpass file
cat ~/.pgpass
chmod 600 ~/.pgpass
```

**High Memory Usage**

```sql
-- Check shared buffers
SHOW shared_buffers;

-- Check work memory
SHOW work_mem;

-- Cache hit ratio
SELECT 
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
```

**Long Running Transactions**

```sql
-- Find long running transactions
SELECT pid, usename, pg_blocking_pids(pid) as blocked_by, query, query_start
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY query_start;

-- Terminate long running query
SELECT pg_terminate_backend(pid) FROM pg_stat_activity
WHERE query_start < NOW() - interval '1 hour';
```

**Disk Space Full**

```bash
# Check PostgreSQL data directory
du -sh /var/lib/postgresql

# Find largest tables
SELECT schemaname, tablename, 
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
WHERE schemaname != 'information_schema'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

**Replication Lag**

```sql
-- Check if replica is caught up
SELECT NOW() - pg_last_xact_replay_timestamp() AS replication_lag;

-- If lag is high, check network and resources
SELECT * FROM pg_stat_replication;
```

***

### Quick Reference

#### Common Ports

* **MySQL**: 3306
* **PostgreSQL**: 5432

#### Common Paths

**MySQL:**

* Configuration: `/etc/mysql/mysql.conf.d/mysqld.cnf`
* Data: `/var/lib/mysql`
* Logs: `/var/log/mysql`
* Socket: `/var/run/mysqld/mysqld.sock`

**PostgreSQL:**

* Configuration: `/etc/postgresql/12/main/postgresql.conf`
* Data: `/var/lib/postgresql/12/main`
* Logs: `/var/log/postgresql`
* Socket: `/var/run/postgresql`

#### Default Credentials

* **MySQL**: root (no password by default)
* **PostgreSQL**: postgres (no password by default)

***

### Configuration Files

#### MySQL Configuration

```bash
# my.cnf or mysqld.cnf
[mysqld]
max_connections = 1000
innodb_buffer_pool_size = 8G
slow_query_log = 1
long_query_time = 2
log_error = /var/log/mysql/error.log
```

#### PostgreSQL Configuration

```bash
# postgresql.conf
max_connections = 200
shared_buffers = 4GB
effective_cache_size = 12GB
work_mem = 20MB
wal_level = replica
max_wal_senders = 3
```

***

### Related Resources

* [MySQL Documentation](https://dev.mysql.com/doc/)
* [PostgreSQL Documentation](https://www.postgresql.org/docs/)
* [MySQL Best Practices](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)
* [PostgreSQL Performance Tips](https://wiki.postgresql.org/wiki/Performance_Optimization)
