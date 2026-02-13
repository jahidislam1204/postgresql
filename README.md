# PostgreSQL High Availability Lab (Streaming Replication)

This project demonstrates a real PostgreSQL **Physical Streaming Replication** setup built on an AWS EC2 instance.

Instead of using managed database services, the replication architecture was configured manually to understand how production databases achieve **high availability and disaster recovery**.

---

## Architecture

```
Client
   |
   v
Primary PostgreSQL (5432)
   |
   |  WAL Streaming Replication
   v
Standby PostgreSQL (5433)
```

The standby server continuously receives WAL (Write Ahead Log) records and replays them to maintain an exact copy of the primary database.

---

## What This Project Demonstrates

- Manual PostgreSQL cluster initialization
- WAL configuration for replication
- Base backup cloning using `pg_basebackup`
- Live streaming replication
- Read-only standby database
- Manual failover (standby promotion)
- Debugging real production issues (ports, permissions, cluster conflicts)

---

## Technologies Used

- PostgreSQL 14
- Ubuntu 22.04
- AWS EC2
- Linux system administration

---

## Key Learning Concepts

- Write Ahead Logging (WAL)
- Physical Replication
- Recovery Mode
- Replication Slots & WAL Sender/Receiver
- High Availability Database Design
- Disaster Recovery

---

## How Replication Works

PostgreSQL does not copy tables directly.

Instead:

1. Primary records every change into WAL
2. WAL sender streams transaction logs
3. Standby receives WAL
4. Standby replays WAL to rebuild database state

This ensures near real-time synchronization between servers.

---

## Failover Demonstration

If the primary server crashes:

```
pg_ctl promote
```

The standby instantly becomes the new primary with no data rebuild required.

---

## Project Structure

```
.
├── README.md      # Project overview
├── setup.md       # Full step-by-step setup commands
```

---

## Setup Instructions

Detailed setup steps are available here:

👉 [setup.md]([setup.md](https://github.com/jahidislam1204/postgresql/blob/main/SETUP.md))

---

## Outcome

Successfully built a working PostgreSQL high availability environment capable of:

- Real-time replication
- Data safety
- Manual failover recovery



