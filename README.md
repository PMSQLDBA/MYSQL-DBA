** 10-lesson MySQL DBA course**, taking you from installation through production administration, including **Windows and Linux**, patching, migration, upgrades, backup/recovery, security, performance, replication, and troubleshooting.

## MySQL Database Administration — 10-Lesson End-to-End Course

| Lesson | Main Skill                      | Outcome                            |
| ------ | ------------------------------- | ---------------------------------- |
| **1**  | Architecture & DBA fundamentals | Understand MySQL                   |
| **2**  | Installation & configuration    | Build MySQL servers                |
| **3**  | Security & users                | Secure/manage access               |
| **4**  | Backup & recovery               | Recover production data            |
| **5**  | Monitoring & performance        | Troubleshoot MySQL                 |
| **6**  | Patching & maintenance          | Safely patch servers               |
| **7**  | Migration & upgrades            | Move/upgrade MySQL                 |
| **8**  | Replication & HA                | Build resilient systems            |
| **9**  | Advanced administration         | Handle complex production DBA work |
| **10** | Production + capstone           | Operate as a real DBA              |

### Lesson 1 — MySQL DBA Foundations + Architecture

You’ll learn:

* What a MySQL DBA actually does
* MySQL editions and distributions
* MySQL Server vs Client
* MySQL architecture
* Storage engines: InnoDB, MyISAM, etc.
* Instance, database, schema, table, index
* MySQL data directory
* Configuration files
* Ports and networking
* Processes, threads, connections
* Important system databases:

  * `mysql`
  * `performance_schema`
  * `sys`
  * `information_schema`
* MySQL startup/shutdown lifecycle
* DBA tools and command-line utilities

**Hands-on:** Build a basic MySQL lab and explore the installation.

---

### Lesson 2 — MySQL Installation & Initial Configuration

**Windows**

* MSI installation
* ZIP installation
* MySQL Installer
* Windows Services
* `my.ini`
* PATH configuration
* Starting/stopping MySQL
* Multiple MySQL instances on Windows

**Linux**

* RPM/YUM/DNF installation
* APT installation
* Generic binary installation
* Systemd service
* `/etc/my.cnf`
* `/etc/mysql/`
* Data directory
* File ownership and permissions

**Configuration**

* `port`
* `bind-address`
* `server_id`
* `datadir`
* `socket`
* `log_error`
* `slow_query_log`
* `general_log`
* `binlog`
* `innodb_buffer_pool_size`
* Connection settings

**Hands-on:** Install MySQL on both Windows and Linux and perform initial hardening.

---

### Lesson 3 — Users, Security & Access Management

You’ll learn:

* Creating users
* Authentication
* Password policies
* Roles
* Privileges
* `GRANT`
* `REVOKE`
* `SHOW GRANTS`
* Least privilege
* Administrative accounts
* Application accounts
* Remote access
* Host-based access
* SSL/TLS
* Account locking
* Password expiration
* Auditing concepts
* Common security mistakes

**Hands-on:** Build DBA, developer, application, read-only, and backup accounts.

---

### Lesson 4 — Backup, Restore & Disaster Recovery

This is one of the most important DBA lessons.

You’ll learn:

* Logical vs physical backups
* `mysqldump`
* MySQL Shell dump utilities
* Physical backup concepts
* Full backups
* Incremental backups
* Binary logs
* Point-in-time recovery
* Backup compression
* Backup encryption
* Backup retention
* Backup validation
* Restore testing
* Database-level restore
* Table-level recovery
* Disaster recovery planning
* RPO and RTO

**Hands-on scenarios:**

1. Database accidentally dropped
2. Table accidentally deleted
3. Server failure
4. Restore from full backup
5. Point-in-time recovery

---

### Lesson 5 — Monitoring, Performance & Troubleshooting

You’ll learn how to determine **why MySQL is slow or unhealthy**.

Topics:

* CPU
* Memory
* Disk I/O
* Disk space
* Connections
* Threads
* Locks
* Transactions
* Deadlocks
* Buffer pool
* Cache
* Temporary tables
* Slow queries
* Error logs
* Binary logs
* Performance Schema
* `sys` schema
* `SHOW PROCESSLIST`
* `SHOW ENGINE INNODB STATUS`
* `EXPLAIN`
* `EXPLAIN ANALYZE`

Performance investigation methodology:

**Application → Query → Index → Transaction → Lock → MySQL → OS → Storage**

**Hands-on:** Diagnose intentionally slow queries and database problems.

---

### Lesson 6 — Patching, Maintenance & Version Management

You’ll learn the DBA lifecycle for maintaining MySQL.

Topics:

* MySQL versions
* LTS releases
* Innovation releases
* Patch releases
* Security patches
* Bug fixes
* Release notes
* Compatibility checking
* Pre-patch health checks
* Backup before patching
* Rollback planning
* Maintenance windows

**Windows**

* Service management
* MSI upgrade
* ZIP-based upgrade
* Configuration preservation
* Data directory handling

**Linux**

* RPM/DNF/YUM patching
* APT patching
* Repository management
* Package dependencies
* Systemd
* Configuration files

**Hands-on:** Create a production-style patching runbook.

---

### Lesson 7 — MySQL Upgrades & Migration

This will cover **major-version upgrades**, not just patches.

You’ll learn:

* Upgrade vs migration
* In-place upgrade
* Logical migration
* Physical migration
* Cross-platform migration
* Windows → Linux
* Linux → Windows
* Older MySQL → newer MySQL
* Compatibility checks
* Deprecated features
* Removed features
* Authentication changes
* Character sets and collations
* SQL modes
* Stored procedures
* Triggers
* Events
* Views
* Users and privileges
* Upgrade validation
* Rollback strategies

Example scenarios:

`MySQL 5.7 → MySQL 8.0`

`Windows MySQL → Linux MySQL`

`Server A → Server B`

`On-premises → Cloud`

**Hands-on:** Perform a complete migration while minimizing downtime.

---

### Lesson 8 — Replication & High Availability

You’ll learn how production MySQL environments achieve redundancy.

Topics:

* Binary logging
* GTIDs
* Source/Replica architecture
* Asynchronous replication
* Replication configuration
* Replica lag
* Replication errors
* Relay logs
* Failover
* Switchover
* Read replicas
* Semi-synchronous replication concepts
* MySQL InnoDB Cluster
* Group Replication concepts
* MySQL Router concepts
* High Availability architecture

**Hands-on:**

```text
              Application
                   |
                   v
             MySQL Primary
              /         \
             /           \
            v             v
       Replica 1       Replica 2
```

You’ll intentionally break replication and recover it.

---

### Lesson 9 — Advanced DBA Operations

This is where we move toward **senior DBA responsibilities**.

Topics:

* InnoDB internals
* Buffer pool
* Redo logs
* Undo logs
* Doublewrite buffer
* Checkpoints
* Transactions
* MVCC
* Isolation levels
* Locking
* Deadlocks
* Tablespaces
* Partitioning
* Index management
* Online DDL
* Schema changes
* Large-table operations
* Metadata locks
* Connection management
* Capacity planning
* Disk growth
* Resource planning
* Production change management

You’ll also learn how to safely perform changes on large production databases.

---

### Lesson 10 — Real-World Production DBA + Complete Capstone

The final lesson ties everything together.

You’ll learn:

* Production DBA daily/weekly/monthly activities
* Health checks
* Monitoring strategy
* Incident response
* Root-cause analysis
* Change management
* Backup strategy
* DR strategy
* Security reviews
* Capacity planning
* Upgrade planning
* Migration planning
* HA architecture
* Documentation
* Automation
* DBA scripting
* Production troubleshooting methodology

### Final Capstone Project

You'll be given a realistic company environment:

```text
                Application
                     |
                     v
              MySQL Primary
              /           \
             /             \
            v               v
       Read Replica 1   Read Replica 2
             |
             v
        Backup Server
```

You will perform:

1. Install MySQL
2. Configure it
3. Secure it
4. Create users
5. Create databases
6. Configure logging
7. Configure backups
8. Perform a restore
9. Configure replication
10. Monitor the environment
11. Diagnose performance problems
12. Patch the server
13. Upgrade MySQL
14. Migrate a database
15. Recover from failure
16. Perform failover
17. Validate the environment
18. Produce a production DBA handover document


**Important:** We’ll treat **Windows and Linux separately wherever the administration procedure differs**, rather than pretending the commands are interchangeable.

We can proceed **strictly one lesson at a time**: I’ll teach **Lesson 1 completely**, give you commands/examples and practical exercises, and only then move to Lesson 2.
