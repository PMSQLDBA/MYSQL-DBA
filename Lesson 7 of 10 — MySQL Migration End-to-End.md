# Lesson 7 of 10 — MySQL Migration End-to-End

Today we cover **migration**, one of the most important real-world DBA skills.

We will cover:

* Windows → Windows
* Linux → Linux
* Windows → Linux
* Linux → Windows
* Same-version migration
* Cross-version migration
* Dump/restore
* MySQL Shell
* Physical vs logical migration
* Character sets
* Users and privileges
* Stored objects
* Validation
* Cutover
* Rollback
* Low-downtime migration strategy

---

# 1. What Is a MySQL Migration?

Migration means moving a MySQL workload from one environment to another.

For example:

```text
OLD SERVER
Windows
MySQL 8.0
    |
    | Migration
    v
NEW SERVER
Linux
MySQL 8.4
```

Or:

```text
OLD
Linux
MySQL 8.4
    |
    v
NEW
Linux
MySQL 8.4
```

Or:

```text
OLD SERVER
On-premises
    |
    v
NEW SERVER
Cloud
```

Migration isn't merely copying database files.

---

# 2. Migration vs Upgrade

These are different operations.

### Upgrade

Same server:

```text
Server A
MySQL 8.0
   ↓
MySQL 8.4
```

### Migration

Different server:

```text
Server A
   ↓
Server B
```

You can combine them:

```text
Old Server
Windows + MySQL 8.0
        |
        v
New Server
Linux + MySQL 8.4
```

That's a **migration + upgrade**.

Such projects require additional testing.

---

# 3. The Migration Lifecycle

Memorize this:

```text
DISCOVER
   ↓
ASSESS
   ↓
DESIGN
   ↓
BUILD
   ↓
MIGRATE
   ↓
VALIDATE
   ↓
CUTOVER
   ↓
MONITOR
   ↓
DECOMMISSION
```

Never begin by immediately running `mysqldump`.

---

# 4. Phase 1 — Discover

Inventory the source:

```text
Hostname
OS
MySQL version
Architecture
CPU
Memory
Storage
Data size
Databases
Tables
Users
Privileges
Plugins
Events
Triggers
Procedures
Functions
Views
Replication
Binary logging
Character sets
Collations
Application connections
```

---

# 5. Source Inventory Commands

Version:

```sql
SELECT VERSION();
```

Databases:

```sql
SHOW DATABASES;
```

Character set:

```sql
SHOW VARIABLES LIKE 'character_set%';
```

Collation:

```sql
SHOW VARIABLES LIKE 'collation%';
```

Data directory:

```sql
SHOW VARIABLES LIKE 'datadir';
```

Binary logging:

```sql
SHOW VARIABLES LIKE 'log_bin';
```

GTID:

```sql
SHOW VARIABLES LIKE 'gtid_mode';
```

---

# 6. Identify Database Size

One useful query:

```sql
SELECT
    table_schema AS database_name,
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS size_mb
FROM information_schema.tables
GROUP BY table_schema
ORDER BY size_mb DESC;
```

This gives you an approximate view of schema sizes.

Why is size important?

Because:

```text
100 MB
```

and:

```text
10 TB
```

require completely different migration strategies.

---

# 7. Identify Largest Tables

```sql
SELECT
    table_schema,
    table_name,
    ROUND(
        (data_length + index_length) / 1024 / 1024,
        2
    ) AS size_mb
FROM information_schema.tables
ORDER BY
    (data_length + index_length) DESC
LIMIT 20;
```

This helps answer:

> Where is most of the data?

---

# 8. Why Data Size Matters

Imagine:

```text
Database = 50 GB
```

A logical dump might be practical.

But:

```text
Database = 20 TB
```

A conventional dump/restore could take a very long time.

Then you might consider:

```text
Replication-based migration
Physical backup
Parallel dump/load
MySQL Shell dump/load
Storage-level migration
```

The correct approach depends on requirements.

---

# 9. Define the Migration Requirements

Before choosing a tool, determine:

```text
Data size:
Downtime allowed:
RPO:
RTO:
Source version:
Target version:
Source OS:
Target OS:
Network bandwidth:
Encryption requirements:
Application compatibility:
Rollback requirement:
```

The most important question:

> **How much downtime can the business tolerate?**

---

# 10. Migration by Downtime

### Option A — Large downtime acceptable

```text
Stop application
    ↓
Dump
    ↓
Transfer
    ↓
Restore
    ↓
Validate
    ↓
Start application
```

Simple.

---

### Option B — Low downtime

```text
Initial copy
      ↓
Replication / change synchronization
      ↓
Catch up
      ↓
Freeze writes
      ↓
Final synchronization
      ↓
Cutover
```

More complex.

---

# 11. Logical vs Physical Migration

There are two major approaches.

## Logical

Export SQL/data:

```text
Source
  ↓
Dump
  ↓
SQL/data files
  ↓
Transfer
  ↓
Import
  ↓
Target
```

Examples:

```text
mysqldump
MySQL Shell dump utilities
```

## Physical

Move/restore database storage structures:

```text
Source
  ↓
Physical backup
  ↓
Transfer
  ↓
Restore
  ↓
Target
```

Physical approaches can be much faster for large datasets but have stricter compatibility requirements.

---

# 12. Logical Migration — `mysqldump`

Traditional tool:

```bash
mysqldump
```

Example:

```bash
mysqldump -u root -p mydb > mydb.sql
```

Restore:

```bash
mysql -u root -p mydb < mydb.sql
```

Conceptually:

```text
Database
   ↓
SQL statements
   ↓
File
   ↓
SQL statements
   ↓
Database
```

---

# 13. What Does a Logical Dump Contain?

Depending on options and objects, it can contain:

```text
CREATE DATABASE
CREATE TABLE
INSERT
ALTER TABLE
Indexes
Triggers
Stored routines
Events
```

But you must understand exactly what your chosen command exports.

Don't assume:

> "Dump = everything."

Always test.

---

# 14. `mysqldump` Example

For a database:

```bash
mysqldump \
  -u root \
  -p \
  --single-transaction \
  mydb > mydb.sql
```

`--single-transaction` is commonly useful for consistent dumps of transactional tables such as InnoDB without taking a broad table lock for the entire dump.

But it doesn't magically make every workload safe.

You need to understand:

* Storage engines
* DDL activity
* Non-transactional tables
* Long transactions
* Replication
* Locking

---

# 15. Dump All Databases

Conceptually:

```bash
mysqldump \
  -u root \
  -p \
  --all-databases > all.sql
```

But be careful.

A migration often requires a more deliberate inventory rather than blindly exporting everything.

You may not want:

```text
Old test database
Temporary schema
Obsolete objects
```

on the new server.

---

# 16. Users Are Special

A common mistake:

```text
Database migrated
   ↓
Application won't connect
```

Why?

Because:

```text
Database objects
      ≠
mysql system/account information
```

Your migration plan must explicitly handle:

```text
Users
Hosts
Authentication
Privileges
Roles
Password policies
```

---

# 17. Inventory Users

Run:

```sql
SELECT
    user,
    host,
    plugin
FROM mysql.user;
```

Then inspect privileges for important accounts:

```sql
SHOW GRANTS FOR 'appuser'@'%';
```

The actual host portion may differ.

---

# 18. Don't Migrate Users Blindly

Suppose the old environment has:

```text
root
old_backup_user
test_user
temporary_admin
application_user
```

Do you really want all of them on the new production server?

Migration is an opportunity to clean up:

```text
Unused accounts
Excess privileges
Old passwords
Temporary access
```

---

# 19. Application Account

Create/validate the application account deliberately.

For example:

```sql
CREATE USER 'appuser'@'10.10.10.%'
IDENTIFIED BY 'strong-password';
```

Then grant only what it needs.

Example:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON application_db.*
TO 'appuser'@'10.10.10.%';
```

Don't blindly copy `GRANT ALL` if the application doesn't require it.

---

# 20. Migration of Stored Objects

Don't forget:

```text
Views
Triggers
Procedures
Functions
Events
```

Your validation checklist must explicitly test them.

For example:

```sql
SHOW TRIGGERS;
```

Events:

```sql
SHOW EVENTS;
```

Procedures/functions can be inspected through the relevant `information_schema` views.

---

# 21. Character Set Migration

This is extremely important when moving between systems.

Check:

```sql
SHOW VARIABLES LIKE 'character_set_server';
```

and:

```sql
SHOW VARIABLES LIKE 'collation_server';
```

Also inspect schemas:

```sql
SELECT
    schema_name,
    default_character_set_name,
    default_collation_name
FROM information_schema.schemata;
```

---

# 22. Why Character Sets Matter

Suppose source uses:

```text
utf8mb4
```

but target is configured differently.

You could encounter:

```text
Incorrect string value
Sorting differences
Comparison differences
Index limitations
Application display problems
```

Always validate real application data.

Especially:

```text
Emoji
Asian languages
Accented characters
Special symbols
```

---

# 23. Windows → Linux

This is a very common migration.

Example:

```text
SOURCE
Windows
C:\MySQLData
MySQL 8.0
       |
       | Logical migration
       v
TARGET
Linux
/var/lib/mysql
MySQL 8.4
```

The logical approach is often attractive because it avoids trying to directly move Windows-specific filesystem structures into Linux.

---

# 24. Why Logical Migration Helps Cross-OS

You export logical database definitions/data:

```text
Windows
   ↓
SQL/data representation
   ↓
Linux
```

The target recreates its own storage structures.

Therefore you're not depending on:

```text
Windows filesystem
Windows paths
Windows service
Linux filesystem
Linux service
```

---

# 25. Linux → Windows

Same concept:

```text
Linux
   ↓
Logical dump
   ↓
Transfer
   ↓
Windows
   ↓
Restore
```

The database contents move independently of OS-specific paths.

---

# 26. Don't Copy `my.cnf` to Windows

This is a classic mistake.

Linux:

```text
/etc/my.cnf
```

might contain:

```text
/var/lib/mysql
```

Windows:

```text
C:\ProgramData\MySQL\...
```

You cannot simply copy the Linux configuration to Windows.

Instead:

```text
Source configuration
       ↓
Understand settings
       ↓
Translate paths/settings
       ↓
Create target configuration
```

---

# 27. Windows Path Problems

Example:

```text
Linux:
datadir=/data/mysql

Windows:
datadir=C:/mysql/data
```

Other examples:

```text
log-error
slow-query-log-file
secure-file-priv
plugin-dir
```

Paths must be appropriate for the target OS.

---

# 28. Linux Permission Problems

On Linux, after migration ensure ownership and permissions are correct.

For example, the MySQL service may need appropriate ownership of its data directory.

Check:

```bash
ls -ld /var/lib/mysql
```

and:

```bash
ls -l /var/lib/mysql
```

Don't blindly run recursive permission changes on production data.

Understand the service account and expected permissions first.

---

# 29. Windows Permissions

Windows also has filesystem permissions.

Check that the account running MySQL has access to:

```text
Data directory
Log directory
Temporary directories
Backup locations
Plugin directories
```

Service startup failures can be caused by permissions.

---

# 30. MySQL Shell

For modern MySQL migrations, you should know **MySQL Shell**.

It provides administrative utilities beyond the traditional `mysql` client.

For dump/load, you should learn:

```text
util.dumpInstance()
util.dumpSchemas()
util.loadDump()
```

These utilities support more modern and scalable dump/load workflows.

---

# 31. MySQL Shell Dump

Conceptually:

```text
Source
  |
  v
MySQL Shell
  |
  +-- Parallel dump
  +-- Chunking
  +-- Metadata
  +-- Data
  |
  v
Dump directory
```

Then:

```text
Dump directory
       |
       v
Target
       |
       v
MySQL Shell
       |
       v
Load
```

This is particularly useful for larger migrations.

---

# 32. MySQL Shell vs `mysqldump`

Think:

| Feature           | `mysqldump`   | MySQL Shell dump |
| ----------------- | ------------- | ---------------- |
| Traditional       | Yes           | Modern           |
| SQL-oriented      | Yes           | Yes/data files   |
| Parallelism       | Limited       | Stronger         |
| Large datasets    | Can be slower | Better suited    |
| Migration tooling | Basic         | More advanced    |

The exact capabilities depend on the MySQL Shell version and utility options.

---

# 33. Example Shell Workflow

Connect:

```bash
mysqlsh
```

Then:

```javascript
util.dumpSchemas(["mydb"], "/backup/mydb")
```

On target:

```javascript
util.loadDump("/backup/mydb")
```

These are examples of the MySQL Shell dump/load API; always consult the target MySQL Shell documentation for the exact options appropriate to your version and migration.

---

# 34. Parallel Dumping

For large databases, parallelism can improve throughput.

Conceptually:

```text
Database
   |
   +---- Worker 1
   +---- Worker 2
   +---- Worker 3
   +---- Worker 4
   |
   v
Dump
```

And loading:

```text
Dump
 |
 +---- Worker 1 → Target
 +---- Worker 2 → Target
 +---- Worker 3 → Target
 +---- Worker 4 → Target
```

But more parallelism isn't always better.

You can saturate:

```text
CPU
Disk
Network
Target MySQL
```

---

# 35. Migration Network Planning

Suppose:

```text
Database = 500 GB
Network = 1 Gbps
```

The theoretical transfer time is not simply:

```text
500 GB / 1 Gbps
```

Real throughput is affected by:

```text
Network overhead
Dump speed
Compression
Encryption
Disk read speed
Target write speed
Parallelism
CPU
```

Always benchmark.

---

# 36. Compression

Migration data can be compressed during transfer.

Conceptually:

```text
Source
   |
   v
Compress
   |
   v
Network
   |
   v
Decompress
   |
   v
Target
```

Compression can reduce network usage but consumes CPU.

So:

```text
Fast network + slow CPU
```

may behave differently from:

```text
Slow network + powerful CPU
```

---

# 37. Validation Is More Than Row Counts

After migration, don't only check:

```sql
SELECT COUNT(*) FROM table;
```

Also validate:

```text
Schemas
Tables
Columns
Indexes
Constraints
Triggers
Views
Procedures
Functions
Events
Users
Privileges
Character sets
Collations
Application behavior
```

---

# 38. Data Validation

For important tables:

```sql
SELECT COUNT(*) FROM customers;
```

Compare:

```text
SOURCE
customers = 10,542,100

TARGET
customers = 10,542,100
```

Good.

But row count alone doesn't guarantee correctness.

---

# 39. Check Aggregates

For numeric data, compare:

```sql
SELECT
    COUNT(*),
    SUM(amount),
    MIN(created_at),
    MAX(created_at)
FROM orders;
```

Run on both source and target.

This can detect some data discrepancies.

---

# 40. Checksums

For stronger validation, use checksums or hashes over carefully selected data.

Conceptually:

```text
Source
   |
   v
Checksum
   |
   X
Target
   |
   v
Checksum
```

If they match, confidence increases.

But generating checksums across huge tables can itself be expensive.

Choose validation methods appropriate to the dataset.

---

# 41. Application Validation

This is the final authority.

Test:

```text
Login
Search
Create record
Update record
Read record
Transactions
Reports
Background jobs
Scheduled events
```

Example:

```text
User creates order
       ↓
DB INSERT
       ↓
Trigger
       ↓
Audit record
       ↓
Report
```

You need to verify the entire chain.

---

# 42. Cutover

Cutover means:

> Switching production traffic from the old environment to the new environment.

Example:

```text
Before:

Application
    |
    v
OLD DB


After:

Application
    |
    v
NEW DB
```

---

# 43. Simple Cutover

For systems where downtime is acceptable:

```text
1. Stop application
2. Final dump
3. Restore target
4. Validate
5. Change application connection
6. Start application
7. Validate
```

This is straightforward.

---

# 44. Low-Downtime Cutover

For more demanding environments:

```text
             OLD PRIMARY
                 |
          Initial migration
                 |
                 v
             NEW SERVER
                 ^
                 |
        Continuous changes
```

Once target catches up:

```text
Stop writes
   ↓
Final sync
   ↓
Verify lag = 0
   ↓
Switch application
   ↓
Start writes on new server
```

This can dramatically reduce downtime.

---

# 45. Replication-Based Migration

One powerful strategy:

```text
OLD
MySQL
  |
  | Replication
  v
NEW
MySQL
```

The old server remains active while the new server catches up.

Then:

```text
Old
 |
 | final changes
 v
New
 |
 v
Cutover
```

This is especially useful for large databases.

---

# 46. Why Replication Helps Migration

Without replication:

```text
500 GB
 ↓
Dump
 ↓
Transfer
 ↓
Restore
 ↓
Application downtime
```

With replication:

```text
Initial copy
      ↓
Replication catches changes
      ↓
Application remains online
      ↓
Short final cutover
```

But replication introduces its own complexity.

We'll cover it deeply in Lesson 8.

---

# 47. Rollback After Cutover

You should have:

```text
OLD SERVER
    |
    | remains available
    |
    v
NEW SERVER
```

Don't immediately destroy the old server.

Keep it available according to your organization's rollback/retention policy.

If:

```text
New server
    ↓
Critical problem
```

you may need:

```text
Application
    ↓
OLD SERVER
```

But rollback becomes complicated once new writes have occurred on the target.

This is why rollback planning must account for **data divergence**, not merely DNS/application configuration.

---

# 48. DNS Cutover

One migration approach is:

```text
Application
   |
   v
db.company.internal
   |
   v
OLD DB
```

Change the endpoint:

```text
db.company.internal
   |
   v
NEW DB
```

But DNS caching means clients may not switch instantly.

Don't assume:

> "I changed DNS, therefore every application is using the new database."

Validate actual connections.

---

# 49. Application Configuration Cutover

Another approach:

```text
Old:

DB_HOST=old-db

New:

DB_HOST=new-db
```

Deploy the configuration change.

Then:

```text
Application restart/rolling deployment
       ↓
New connections
       ↓
New database
```

This can be easier to control than DNS in some environments.

---

# 50. Migration Rollback Decision

Define beforehand:

```text
Rollback if:
- Application errors exceed threshold
- Data validation fails
- Latency exceeds threshold
- Replication/cutover failure
- Critical feature fails
```

Example:

```text
Error rate > 2%
for 5 minutes
        ↓
Rollback decision
```

Your actual thresholds must be defined by your application's SLOs.

---

# 51. Migration Failure Example

Imagine:

```text
Source:
Windows / MySQL 8.0

Target:
Linux / MySQL 8.4
```

Migration succeeds.

But the application fails with:

```text
Authentication plugin error
```

What happened?

Potentially:

```text
Server migration succeeded
       ↓
Application compatibility failed
```

The database itself may be healthy.

This illustrates why you test the complete stack.

---

# 52. Migration Failure Example #2

Target starts successfully.

Application reports:

```text
Incorrect string value
```

You investigate:

```sql
SHOW VARIABLES LIKE 'character_set%';
```

and find different character-set behavior.

Again:

```text
Server startup = success
```

doesn't mean:

```text
Migration = success
```

---

# 53. Migration Failure Example #3

Target has:

```text
Database
Tables
Data
```

But application gets:

```text
Access denied
```

You inspect:

```sql
SHOW GRANTS FOR 'appuser'@'...';
```

and discover the user wasn't recreated correctly.

This is why accounts and privileges are first-class migration objects.

---

# 54. Migration Runbook

Use this structure.

## PRE-MIGRATION

```text
[ ] Inventory source
[ ] Inventory target
[ ] Confirm versions
[ ] Confirm OS
[ ] Measure database size
[ ] Identify largest tables
[ ] Inventory users
[ ] Inventory privileges
[ ] Inventory stored objects
[ ] Check character sets
[ ] Check replication
[ ] Confirm backup
[ ] Test restore
[ ] Test migration
[ ] Define rollback
```

## MIGRATION

```text
[ ] Stop/drain application if required
[ ] Perform final sync
[ ] Load target
[ ] Validate data
[ ] Validate users
[ ] Validate objects
[ ] Validate application
```

## CUTOVER

```text
[ ] Freeze writes
[ ] Confirm synchronization
[ ] Switch application
[ ] Test critical workflows
[ ] Monitor
```

## POST-MIGRATION

```text
[ ] Monitor target
[ ] Confirm backups
[ ] Confirm monitoring
[ ] Confirm replication/HA
[ ] Retain old server
[ ] Document migration
```

---

# 55. Your Migration Decision Matrix

| Requirement                     | Good starting strategy                                   |
| ------------------------------- | -------------------------------------------------------- |
| Small DB + downtime acceptable  | Logical dump/restore                                     |
| Cross-OS                        | Logical migration often simplest                         |
| Large DB                        | Parallel dump/load or physical approach                  |
| Very low downtime               | Replication-based migration                              |
| Need major OS change            | Build new target                                         |
| Need major MySQL version change | Test separately and carefully                            |
| Need clean environment          | Build new server + logical migration                     |
| Need rollback                   | Keep old environment and design data-divergence strategy |

This isn't a universal rulebook—the actual choice depends on your environment.

---

# 56. Hands-On Lab — Windows → Linux Simulation

You don't need two physical servers.

Create:

```text
VM 1
Windows
MySQL
Source

VM 2
Linux
MySQL
Target
```

Then:

### Source

```sql
CREATE DATABASE migration_lab;
```

```sql
USE migration_lab;
```

```sql
CREATE TABLE customers
(
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(200)
);
```

Insert test data.

---

# 57. Export

Run:

```bash
mysqldump \
  -u root \
  -p \
  --single-transaction \
  migration_lab > migration_lab.sql
```

Transfer:

```text
Windows
   |
   | migration_lab.sql
   v
Linux
```

---

# 58. Import

On Linux:

```bash
mysql -u root -p < migration_lab.sql
```

Then:

```sql
SHOW DATABASES;
```

```sql
USE migration_lab;
```

```sql
SELECT * FROM customers;
```

---

# 59. Validate

Compare:

```sql
SELECT COUNT(*) FROM customers;
```

Then:

```sql
SELECT
    MIN(id),
    MAX(id),
    COUNT(*)
FROM customers;
```

Test:

```sql
INSERT INTO customers
VALUES (999, 'Migration Test', 'migration@example.com');
```

Then:

```sql
SELECT *
FROM customers
WHERE id = 999;
```

---

# 60. Migration Lab — Users

On target, create a dedicated application user:

```sql
CREATE USER 'migration_app'@'localhost'
IDENTIFIED BY 'StrongPasswordHere';
```

Grant:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON migration_lab.*
TO 'migration_app'@'localhost';
```

Then:

```sql
SHOW GRANTS FOR 'migration_app'@'localhost';
```

Test login using that account.

This teaches you an important lesson:

> **A migrated database isn't production-ready until its access model works.**

---

# 61. Migration Lab — Cross-OS Configuration

On Linux, inspect:

```sql
SHOW VARIABLES LIKE 'datadir';
```

On Windows, inspect the same variable.

Compare.

You'll see that:

```text
Windows path
```

and:

```text
Linux path
```

are different.

Therefore configuration must be adapted, not copied blindly.

---

# 62. Lesson 7 Interview Questions

You should be able to answer:

1. What is database migration?
2. What's the difference between migration and upgrade?
3. What is logical migration?
4. What is physical migration?
5. When would you use `mysqldump`?
6. When might MySQL Shell dump/load be preferable?
7. Why does database size affect migration strategy?
8. How do you migrate Windows → Linux?
9. How do you migrate Linux → Windows?
10. Why shouldn't you copy `my.cnf` directly to Windows?
11. Why are users and privileges important during migration?
12. How do you validate migrated data?
13. Why isn't row count alone sufficient?
14. What is cutover?
15. What is a low-downtime migration?
16. How can replication help migration?
17. What is a rollback problem after new writes occur on the target?
18. Why do character sets matter?
19. Why should you test application drivers?
20. What should a migration runbook contain?

---

# The Migration Formula

Remember this:

```text
              MIGRATION
                  |
          +-------+-------+
          |               |
       SOURCE           TARGET
          |               |
      Inventory        Build
      Backup           Configure
      Assess           Secure
          |               |
          +-------+-------+
                  |
              MIGRATE
                  |
              VALIDATE
                  |
               CUTOVER
                  |
              MONITOR
                  |
             DECOMMISSION
```

And the golden rule:

> **Never decommission the source immediately after a successful cutover.**

Keep the old environment according to your rollback and retention policy until the migration is proven stable.

---

## Progress: 7 / 10

```text
✅ 1. Architecture & DBA Fundamentals
✅ 2. Installation & Configuration
✅ 3. Security, Users & Privileges
✅ 4. Backup, Restore & DR
✅ 5. Monitoring, Performance & Troubleshooting
✅ 6. Patching & Version Management
✅ 7. Migration — Windows/Linux/Cross-Version
⬜ 8. Replication & High Availability
⬜ 9. Advanced InnoDB, Security & Production Operations
⬜ 10. Complete End-to-End Production DBA
```

**Next: Lesson 8 — MySQL Replication, High Availability & Disaster Recovery.** We'll build the concepts behind **Source/Replica, GTID, binary logs, replication lag, failover, switchover, read-only replicas, replication troubleshooting, and production HA architecture**.
