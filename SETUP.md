# PostgreSQL Streaming Replication Lab (Primary to Standby)

This document explains how to build PostgreSQL physical streaming replication on a single EC2 instance using two independent database clusters.

---

## Lab Goal

Create a production-like architecture:

```
Client -> Primary DB (5432) -> WAL Streaming -> Standby DB (5433)
```

We will:

- Create a manual PostgreSQL primary server
- Clone it using `pg_basebackup`
- Start a standby replica
- Verify real-time replication
- Simulate failover

---

## Environment

| Component    | Value        |
|------------ |------------ |
| OS           | Ubuntu 22.04 |
| PostgreSQL   | 14           |
| Cloud        | AWS EC2      |
| Primary Port | 5432         |
| Standby Port | 5433         |

---

## 1. Install PostgreSQL

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

Stop default cluster (we will create our own):

```bash
sudo pg_ctlcluster 14 main stop
```

---

## 2. Create Directories

```bash
mkdir -p /var/lib/postgresql/primary
mkdir -p /var/lib/postgresql/replica

chown -R postgres:postgres /var/lib/postgresql
chmod 700 /var/lib/postgresql/primary
chmod 700 /var/lib/postgresql/replica
```

---

## 3. Initialize Primary Database

```bash
sudo -u postgres /usr/lib/postgresql/14/bin/initdb -D /var/lib/postgresql/primary
```

---

## 4. Configure Primary Server

```bash
sudo -u postgres nano /var/lib/postgresql/primary/postgresql.conf
```

Add or modify:

```
listen_addresses = '*'
port = 5432

wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
hot_standby = on
```

---

## 5. Allow Replication Connection

```bash
sudo -u postgres nano /var/lib/postgresql/primary/pg_hba.conf
```

Add at bottom:

```
host replication replicator 127.0.0.1/32 md5
host all all 0.0.0.0/0 md5
```

---

## 6. Start Primary Server

```bash
sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl \
-D /var/lib/postgresql/primary \
-l /var/lib/postgresql/primary/logfile start
```

Verify:

```bash
ss -lntp | grep 5432
```

---

## 7. Create Replication User

```bash
psql -p 5432 -U postgres
```

```sql
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD '123';
CREATE DATABASE testdb;
\q
```

Verify WAL level:

```bash
psql -p 5432 -U postgres -c "SHOW wal_level;"
```

---

## 8. Create Standby Using Base Backup

```bash
sudo -u postgres /usr/lib/postgresql/14/bin/pg_basebackup \
-h 127.0.0.1 \
-D /var/lib/postgresql/replica \
-U replicator \
-Fp -Xs -P -R
```

---

## 9. Configure Standby Port

```bash
sudo -u postgres nano /var/lib/postgresql/replica/postgresql.conf
```

Set:

```
port = 5433
hot_standby = on
```

Comment any other `port = 5432` line.

---

## 10. Start Standby Server

```bash
sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl \
-D /var/lib/postgresql/replica \
-l /var/lib/postgresql/replica/logfile start
```

Verify both running:

```bash
ss -lntp | grep 543
```

---

## 11. Verify Standby Mode

```bash
psql -p 5433 -U postgres -c "SELECT pg_is_in_recovery();"
```

Expected output:

```
t
```

---

## 12. Test Live Replication

### On Primary

```bash
psql -p 5432 -U postgres testdb
```

```sql
CREATE TABLE demo(id INT, name TEXT);
INSERT INTO demo VALUES (1,'Hello from Primary');
INSERT INTO demo VALUES (2,'Replication is working');
SELECT * FROM demo;
```

### On Standby

```bash
psql -p 5433 -U postgres testdb
```

```sql
SELECT * FROM demo;
```

Data should appear instantly.

---

## 13. Simulate Failover

Stop primary:

```bash
sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl -D /var/lib/postgresql/primary stop
```

Promote standby:

```bash
sudo -u postgres /usr/lib/postgresql/14/bin/pg_ctl -D /var/lib/postgresql/replica promote
```

Standby becomes new primary.

---

## Key Concepts Learned

- WAL (Write Ahead Log)
- Physical streaming replication
- Base backup cloning
- Recovery mode
- Failover promotion
- High availability database design

---

## Useful Monitoring Queries

Primary:

```sql
SELECT * FROM pg_stat_replication;
```

Standby:

```sql
SELECT pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn();
```

---

## Conclusion

You now have a working PostgreSQL high availability environment capable of real-time replication and manual failover.
