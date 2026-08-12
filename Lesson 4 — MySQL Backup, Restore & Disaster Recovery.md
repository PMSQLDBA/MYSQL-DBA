# Lesson 4 — MySQL Backup, Restore & Disaster Recovery

This is **Lesson 4 of 10**.

This lesson is critical because a DBA isn't judged only by whether the database is running.

A real DBA must be able to answer:

> **"If the database disappears right now, how do we get the data back?"**

Our workflow:

```text
BACKUP
  ↓
VERIFY
  ↓
STORE SAFELY
  ↓
MONITOR
  ↓
RECOVER
  ↓
VALIDATE
  ↓
DOCUMENT
```

---

# 1. Backup vs Recovery

These are different concepts.

### Backup

Creating a copy of data that can be used for recovery.

### Recovery

Using backups/logs to return the database to a usable state.

Think:

```text
Production
    |
    v
 Backup
    |
    v
Failure
    |
    v
Restore
    |
    v
Recovery
    |
    v
Validation
```

Having a backup doesn't automatically mean you have a working recovery strategy.

---

# 2. The Most Important DBA Rule

> **A backup that has never been successfully restored is only an assumption.**

For example:

```text
Backup completed successfully
        ≠
Restore will definitely work
```

You must periodically perform test restores.

---

# 3. RPO and RTO

Two critical disaster-recovery concepts.

## RPO — Recovery Point Objective

How much data loss can the business tolerate?

Example:

```text
RPO = 15 minutes
```

This means the organization may accept losing up to approximately 15 minutes of data, depending on the actual recovery design.

## RTO — Recovery Time Objective

How long can the service be unavailable?

Example:

```text
RTO = 1 hour
```

Meaning the organization targets restoring service within roughly one hour.

Conceptually:

```text
             Failure
                |
                v
Time ----------X------------------>
              / \
             /   \
          RPO     RTO
```

Your backup architecture should be designed around business requirements, not just DBA convenience.

---

# 4. Types of MySQL Backups

At a high level:

```text
MySQL Backups
      |
      +-- Logical
      |
      +-- Physical
      |
      +-- Full
      |
      +-- Incremental
      |
      +-- Binary-log based
```

The exact tooling and recovery capabilities depend on your MySQL version, edition, storage architecture, and operational requirements.

---

# 5. Logical Backup

A logical backup exports database objects/data into a representation that can be recreated.

Common tool:

```text
mysqldump
```

Example:

```bash
mysqldump -u root -p companydb > companydb.sql
```

Conceptually:

```text
Database
   |
   v
mysqldump
   |
   v
SQL dump
   |
   v
companydb.sql
```

Restore:

```bash
mysql -u root -p companydb < companydb.sql
```

---

# 6. What Does a Logical Backup Contain?

Depending on options and the objects being dumped, it can contain SQL statements such as:

```sql
CREATE TABLE ...
INSERT INTO ...
CREATE VIEW ...
```

Conceptually:

```text
Dump File
 |
 +-- Database definition
 +-- Table definitions
 +-- Data
 +-- Other selected objects
```

This makes logical backups portable.

---

# 7. Advantages of Logical Backups

Logical backups are useful because they can:

* Be relatively portable
* Allow selective restoration
* Allow object-level recovery
* Be inspected
* Work across some platform differences
* Be useful for migrations
* Be useful for smaller databases

For example:

```text
Production
    |
    v
mysqldump
    |
    v
SQL file
    |
    v
New server
```

This makes logical backup a particularly important tool for **migration**, which we'll use in Lesson 7.

---

# 8. Disadvantages of Logical Backups

Logical backups can be:

* Slow for very large databases
* CPU-intensive
* I/O-intensive
* Larger in some cases
* Slower to restore

Imagine:

```text
Database = 10 TB
```

A logical export/import may be impractical for tight recovery windows.

This is where physical backup approaches become important.

---

# 9. Physical Backup

A physical backup works closer to the database's underlying storage representation.

Conceptually:

```text
MySQL
  |
  v
Data Files
  |
  v
Physical Backup
  |
  v
Backup Storage
```

Physical backup tools can be significantly more appropriate for large production environments.

MySQL's ecosystem includes backup approaches/tools designed for physical backups, depending on your MySQL edition and environment.

---

# 10. Logical vs Physical

Think about them this way:

|                   | Logical          | Physical                       |
| ----------------- | ---------------- | ------------------------------ |
| Representation    | SQL/object-level | Storage-level                  |
| Portability       | Generally high   | More environment-dependent     |
| Selective restore | Strong           | More complex                   |
| Large DB recovery | Can be slower    | Often better suited            |
| Migration         | Very useful      | Useful in compatible scenarios |
| Human-readable    | Often yes        | No                             |

Neither is universally "better."

A production DBA often uses a combination of strategies.

---

# 11. `mysqldump`

Let's start with the classic logical backup utility.

Basic:

```bash
mysqldump -u root -p companydb > companydb.sql
```

You enter the password when prompted.

This creates:

```text
companydb.sql
```

---

# 12. Backup Multiple Databases

You can specify multiple databases or use appropriate `mysqldump` options.

For example:

```bash
mysqldump -u root -p db1 db2 > databases.sql
```

Another important option is:

```bash
mysqldump -u root -p --all-databases > all_databases.sql
```

Be careful with `--all-databases` in production.

A full logical dump can contain:

* Application databases
* System-related data
* Users/privileges depending on version/options
* Other objects

Always understand exactly what you're backing up and how you intend to restore it.

---

# 13. Schema-Only Backup

Sometimes you need the structure without the data.

For example:

```bash
mysqldump -u root -p --no-data companydb > companydb_schema.sql
```

This is useful for:

* Development environments
* Schema migration
* Testing
* Comparing structures
* Recreating empty environments

---

# 14. Data-Only Backup

You can also perform a data-oriented dump without table creation statements.

Conceptually:

```bash
mysqldump -u root -p --no-create-info companydb > companydb_data.sql
```

This is useful in specific migration/recovery scenarios.

---

# 15. Important `mysqldump` Options

You'll encounter options such as:

```text
--single-transaction
--routines
--triggers
--events
--no-data
--no-create-info
--databases
--all-databases
```

You should understand each option before using it in production.

For transactional InnoDB workloads, a commonly used approach is:

```bash
mysqldump -u root -p \
  --single-transaction \
  --routines \
  --triggers \
  --events \
  companydb > companydb.sql
```

This is only an example strategy; exact backup requirements should be validated for your workload and MySQL version.

---

# 16. Why `--single-transaction` Matters

For transactional InnoDB data, `--single-transaction` can provide a consistent logical view without requiring a long blocking table lock in the same way older approaches might.

Conceptually:

```text
Application
    |
    v
MySQL
    |
    +----> Transactions continue
    |
    +----> Dump obtains consistent view
```

However, it isn't a universal solution.

For example, non-transactional tables or certain schema operations can require additional consideration.

---

# 17. Backup Compression

A SQL dump can be compressed.

Linux example:

```bash
mysqldump -u root -p \
  --single-transaction \
  companydb | gzip > companydb.sql.gz
```

This can significantly reduce storage usage.

Restore:

```bash
gunzip < companydb.sql.gz | mysql -u root -p companydb
```

---

# 18. Windows Backup

On Windows, the same MySQL utilities can be used.

Example:

```powershell
mysqldump.exe -u root -p companydb > C:\DBBackup\companydb.sql
```

The important thing is to use an appropriate backup destination.

Don't treat:

```text
C:\DBBackup\
```

on the same disk as the database as your complete disaster-recovery strategy.

If the disk fails:

```text
Database disk
     X
Backup disk
     X
```

You could lose both.

---

# 19. Better Backup Architecture

Instead:

```text
                 MySQL
                   |
                   v
             Backup Process
                   |
             +-----+-----+
             |           |
             v           v
        Local Backup  Remote Backup
                         |
                         v
                  Backup Storage
```

Better still, maintain appropriate off-host/off-site copies according to your organization's requirements.

---

# 20. The 3-2-1 Backup Principle

A classic backup strategy is:

```text
3 copies of data
2 different storage types/media
1 copy off-site
```

For example:

```text
Production
   |
   +--> Local backup
   |
   +--> Backup server
   |
   +--> Off-site/object storage
```

Modern cloud architectures may implement this differently, but the principle remains valuable:

> Don't keep every copy of your data in one failure domain.

---

# 21. Backup Retention

Don't just say:

> "We take backups."

You need a retention policy.

Example:

```text
Daily backups → 30 days
Weekly backups → 12 weeks
Monthly backups → 12 months
```

The actual policy should be driven by:

* Business requirements
* Compliance
* Storage cost
* Recovery requirements

---

# 22. Backup Validation

After a backup:

```text
Backup completed
      |
      v
Check file exists
      |
      v
Check size
      |
      v
Check logs/status
      |
      v
Test restore
```

Don't assume:

```text
exit code = 0
```

means the entire disaster-recovery process is good.

A mature process includes restore testing.

---

# 23. Restore a Logical Backup

Suppose:

```text
companydb.sql
```

exists.

Create the target database if necessary:

```sql
CREATE DATABASE companydb;
```

Then:

```bash
mysql -u root -p companydb < companydb.sql
```

Afterward:

```sql
USE companydb;

SHOW TABLES;
```

Then validate:

```sql
SELECT COUNT(*)
FROM employees;
```

---

# 24. Restore Validation

A restore isn't finished when the command ends.

Validate:

```text
Database exists
       ↓
Tables exist
       ↓
Expected row counts
       ↓
Indexes
       ↓
Constraints
       ↓
Views
       ↓
Stored programs
       ↓
Application connectivity
       ↓
Application functionality
```

This is critical.

---

# 25. Accidental DROP TABLE Scenario

Imagine:

```sql
DROP TABLE customers;
```

The table is gone.

Your response shouldn't be:

> "Restore everything."

Instead:

```text
1. Stop/limit further destructive activity
2. Determine exact incident time
3. Determine recovery requirements
4. Identify backup
5. Identify binary logs
6. Restore to an isolated environment
7. Recover required data
8. Validate
9. Restore/reintegrate safely
```

This is where point-in-time recovery becomes important.

---

# 26. Binary Logs

The binary log records database changes in a form that MySQL can use for replication and recovery.

Conceptually:

```text
Transaction
    |
    +----> Binary Log
              |
              +-- INSERT
              +-- UPDATE
              +-- DELETE
              ...
```

This is why binary logging is so important for production databases.

---

# 27. Point-in-Time Recovery

Suppose:

```text
02:00  Full Backup
       |
       | normal transactions
       |
10:37  Accidental DROP TABLE
```

If you only restore the 02:00 backup:

```text
02:00 backup
    |
    v
Restore
    |
    v
Data as of 02:00
```

You potentially lose everything between 02:00 and 10:37.

But if you have binary logs:

```text
02:00 Full Backup
       |
       +--> Binlog events
       |
       +--> Binlog events
       |
       +--> Binlog events
       |
10:37 DROP TABLE
```

You can potentially recover to a point **just before the destructive event**, assuming your backup/binlog chain is valid and the required logs are available.

---

# 28. Point-in-Time Recovery Architecture

Conceptually:

```text
                Full Backup
                    |
                    v
             Restore Backup
                    |
                    v
              Apply Binlogs
                    |
                    v
          Stop before bad event
                    |
                    v
               Validate
```

This is one of the most important production recovery techniques.

---

# 29. Why Binary Logs Must Be Protected

Imagine:

```text
Server
 |
 +-- Database
 |
 +-- Binary Logs
```

If the server's disk fails:

```text
Database      X
Binary logs   X
```

You may lose the recovery chain.

Therefore, production environments often ship/copy binary logs to another failure domain.

Conceptually:

```text
MySQL
  |
  +--> Binary Logs
          |
          v
      Remote Storage
```

---

# 30. Full Backup + Binary Logs

A common conceptual recovery architecture is:

```text
                Full Backup
                    |
                    v
              Backup Storage

                MySQL
                  |
                  v
              Binary Logs
                  |
                  v
            Remote Storage
```

Recovery:

```text
Full Backup
     ↓
Restore
     ↓
Apply Binary Logs
     ↓
Stop at desired point
     ↓
Validate
```

---

# 31. Backup Schedule Example

Suppose business requirements are:

```text
RPO = 15 minutes
RTO = 1 hour
```

A possible strategy might involve:

```text
Full backup
     +
Continuous binary log preservation
     +
Remote backup storage
     +
Regular restore testing
```

But don't blindly equate a 15-minute RPO with "run a full backup every 15 minutes."

Binary-log-based recovery can provide much finer recovery points.

---

# 32. Backup Security

Backups contain sensitive information.

Protect them.

```text
Production Data
      |
      v
Backup
      |
      +--> Encryption
      |
      +--> Access control
      |
      +--> Retention
      |
      +--> Monitoring
      |
      +--> Off-site copy
```

A database may be securely protected while its backup is sitting in an open directory.

That would defeat much of the security model.

---

# 33. Backup Permissions

Only authorized personnel/processes should access backups.

For example:

```text
DBA
 |
 +-- Backup management
 |
 +-- Restore management

Application
 |
 X-- Direct backup access
```

Application users generally should not have access to database backup files.

---

# 34. Backup Monitoring

A production backup system should alert when:

```text
Backup failed
Backup didn't run
Backup is unexpectedly small
Storage is nearly full
Binary-log shipping stopped
Backup age exceeds threshold
Restore test failed
```

A useful metric is:

> **Age of last successful backup**

Another:

> **Age of last recoverable binary log**

---

# 35. Backup Failure Scenario

Suppose the scheduled backup normally creates:

```text
companydb_2026-08-12.sql.gz
```

but today it didn't.

A weak DBA response:

> "I'll check it tomorrow."

A production DBA response:

```text
Backup failure detected
       ↓
Investigate immediately
       ↓
Check scheduler
       ↓
Check MySQL connectivity
       ↓
Check credentials
       ↓
Check disk space
       ↓
Check permissions
       ↓
Check MySQL logs
       ↓
Rerun/test backup
       ↓
Validate
       ↓
Document
```

---

# 36. Common Backup Problems

### Problem 1 — Disk Full

```text
Backup
  ↓
Disk full
  ↓
Backup fails
```

Check:

Linux:

```bash
df -h
```

Windows:

Check available disk capacity.

---

### Problem 2 — Permission Denied

```text
Backup process
      |
      X
Cannot write destination
```

Check the OS account running the backup.

---

### Problem 3 — Incorrect Credentials

The backup process cannot authenticate to MySQL.

Check:

```text
Username
Host
Authentication
Password/secret
Privileges
```

---

### Problem 4 — Backup File Exists but Is Incomplete

Don't assume:

```text
file exists = valid backup
```

Validate the process and test restores.

---

# 37. Physical Backup Concepts

For large databases, physical backup solutions can dramatically improve recovery speed.

Conceptually:

```text
10 TB database
    |
    +-- Logical backup
    |       |
    |       +-- Potentially very long
    |
    +-- Physical backup
            |
            +-- Potentially much faster recovery
```

But physical backup solutions have more operational complexity.

You need to understand:

* Version compatibility
* Storage engine
* Backup consistency
* Recovery procedure
* Backup metadata
* Encryption
* Incremental chains
* Restore dependencies

---

# 38. MySQL Shell Backup Utilities

Modern MySQL environments may also use **MySQL Shell's dump and load utilities**, which are particularly useful for large datasets and migration workflows.

You'll encounter commands/tools such as:

```text
util.dumpInstance()
util.dumpSchemas()
util.loadDump()
```

These can provide features beyond traditional `mysqldump`, including parallelized dump/load workflows.

We'll use these more when we get to **migration in Lesson 7**.

---

# 39. Backup Architecture — Production Example

A more mature architecture:

```text
                    Production
                        |
                        v
                  MySQL Primary
                        |
            +-----------+-----------+
            |                       |
            v                       v
       Full Backup             Binary Logs
            |                       |
            v                       v
      Backup Server           Remote Storage
            |
            v
      Off-site Storage
            |
            v
       Restore Testing
```

Notice the final component:

```text
Restore Testing
```

That's what turns a backup strategy into a recovery strategy.

---

# 40. Disaster Recovery

Disaster Recovery isn't just:

> "We have backups."

DR considers:

```text
People
Process
Technology
Data
Dependencies
Communication
Recovery procedures
Testing
```

For example:

```text
Primary DC
    |
    X
Major outage
    |
    v
DR Environment
    |
    v
Restore/Failover
    |
    v
Application
```

We'll expand this significantly when we reach replication and HA in Lesson 8.

---

# 41. Your Hands-On Lab

Create a test database:

```sql
CREATE DATABASE backup_lab;
```

Then:

```sql
USE backup_lab;

CREATE TABLE employees
(
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department VARCHAR(100)
);
```

Insert:

```sql
INSERT INTO employees
VALUES
(1, 'John', 'IT'),
(2, 'Sarah', 'Finance'),
(3, 'David', 'HR'),
(4, 'Maria', 'Sales');
```

Verify:

```sql
SELECT *
FROM employees;
```

---

# 42. Perform Your First Backup

Run:

```bash
mysqldump -u root -p backup_lab > backup_lab.sql
```

Check that the file exists.

Then inspect its contents.

You should see SQL statements corresponding to the database objects/data.

---

# 43. Test Your First Restore

Create another database:

```sql
CREATE DATABASE backup_lab_restore;
```

Restore:

```bash
mysql -u root -p backup_lab_restore < backup_lab.sql
```

Then:

```sql
USE backup_lab_restore;

SHOW TABLES;

SELECT *
FROM employees;
```

If you see the expected rows, you've completed your first recovery test.

---

# 44. Now Simulate Disaster

In your **lab only**, delete the test database:

```sql
DROP DATABASE backup_lab;
```

Do not do this on a production server.

Now recreate it:

```sql
CREATE DATABASE backup_lab;
```

Restore:

```bash
mysql -u root -p backup_lab < backup_lab.sql
```

Validate:

```sql
SELECT *
FROM backup_lab.employees;
```

You've just performed a basic backup/recovery cycle.

---

# 45. Your Lesson 4 DBA Exercise

You should now practice these scenarios:

### Exercise 1

Create a database.

Take a logical backup.

Delete the database.

Restore it.

---

### Exercise 2

Create several tables.

Back up the database.

Restore it to a different database name.

Verify:

* Tables
* Rows
* Indexes
* Views if applicable

---

### Exercise 3

Create:

```text
backup_lab
```

and generate:

```text
backup_lab.sql
```

Then compress it where appropriate.

---

### Exercise 4

Find the MySQL binary log configuration:

```sql
SHOW VARIABLES LIKE 'log_bin';
```

Then inspect:

```sql
SHOW VARIABLES LIKE 'log_bin_basename';
```

and:

```sql
SHOW BINARY LOGS;
```

Your exact output depends on how your server is configured.

---

# 46. DBA Interview Questions

Be able to answer:

1. What is the difference between backup and recovery?
2. What is RPO?
3. What is RTO?
4. What is a logical backup?
5. What is a physical backup?
6. What is `mysqldump`?
7. What does `--single-transaction` do conceptually?
8. Why are binary logs important?
9. What is point-in-time recovery?
10. Why should backups be stored off the database server?
11. Why should backups be encrypted?
12. Why should you test restores?
13. What happens if your backup disk fails with your database disk?
14. What is the 3-2-1 backup principle?
15. What would you do after an accidental `DROP TABLE`?
16. Why might a logical backup be unsuitable for a huge database with a very short RTO?
17. Why should backup failures generate alerts?

---

# 47. The Recovery Mindset

Remember this workflow:

```text
              INCIDENT
                  |
                  v
          Stop Further Damage
                  |
                  v
          Identify Exact Time
                  |
                  v
         Identify Recovery Point
                  |
                  v
        Find Valid Backup Chain
                  |
                  v
             Restore
                  |
                  v
         Apply Required Logs
                  |
                  v
             Validate
                  |
                  v
          Resume Application
                  |
                  v
        Root Cause Analysis
```

Never rush straight into restoring production without understanding **what you are restoring, from where, and to what point in time**.

---

# Lesson 4 Complete

You now understand the core recovery toolkit:

```text
Logical Backup
     ↓
mysqldump
     ↓
Physical Backup Concepts
     ↓
Binary Logs
     ↓
Point-in-Time Recovery
     ↓
RPO / RTO
     ↓
Restore Testing
     ↓
Disaster Recovery
```

## Next — Lesson 5: Monitoring, Performance & Troubleshooting

This is where we start thinking like a **production DBA**.

We'll learn:

* CPU/memory/disk investigation
* MySQL connections
* Threads
* Sessions
* Locks
* Deadlocks
* Transactions
* InnoDB Buffer Pool
* Slow Query Log
* `SHOW PROCESSLIST`
* `SHOW ENGINE INNODB STATUS`
* Performance Schema
* `sys` schema
* `EXPLAIN`
* `EXPLAIN ANALYZE`
* Index troubleshooting
* Blocking queries
* Long-running transactions
* Disk I/O problems
* "MySQL is slow" incident methodology
* **Windows + Linux OS-level troubleshooting**
* A complete production-style performance incident lab
