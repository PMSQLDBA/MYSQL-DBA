# Lesson 5 — MySQL Monitoring, Performance & Troubleshooting

This is **Lesson 5 of 10**.

From this lesson onward, we're moving from **"I can administer MySQL"** toward **"I can diagnose production MySQL problems."**

The most important skill today is learning **how to investigate before changing anything**.

Our production troubleshooting model:

```text
Problem
   ↓
Observe
   ↓
Collect Evidence
   ↓
Identify Bottleneck
   ↓
Form Hypothesis
   ↓
Test
   ↓
Fix
   ↓
Validate
   ↓
Monitor
```

---

# 1. What Does "MySQL Is Slow" Actually Mean?

"MySQL is slow" is not a diagnosis.

It could mean:

```text
Application
    |
    +-- Network latency
    |
    +-- Connection problems
    |
    +-- Slow SQL
    |
    +-- Missing index
    |
    +-- Lock contention
    |
    +-- Long transaction
    |
    +-- CPU saturation
    |
    +-- Memory pressure
    |
    +-- Disk I/O
    |
    +-- Storage capacity
    |
    +-- MySQL configuration
    |
    +-- Application behavior
```

Your job is to determine **which one**.

---

# 2. The DBA Performance Stack

Think about MySQL as several layers:

```text
+----------------------------+
| Application                |
+----------------------------+
| SQL / Queries              |
+----------------------------+
| Transactions / Locks       |
+----------------------------+
| MySQL Server               |
+----------------------------+
| InnoDB                     |
+----------------------------+
| Memory                     |
+----------------------------+
| CPU                        |
+----------------------------+
| Disk / Storage             |
+----------------------------+
| Operating System           |
+----------------------------+
| Hardware / VM / Cloud      |
+----------------------------+
```

A performance problem at any layer can appear as a MySQL problem.

---

# 3. First Rule: Establish Baseline

Before troubleshooting, know what "normal" looks like.

Useful baseline metrics:

```text
CPU
Memory
Disk utilization
Disk latency
Connections
Queries/sec
Transactions/sec
Threads
Buffer pool
Lock waits
Replication lag
Disk space
Error rate
Slow queries
```

Without a baseline:

```text
Current = 500 connections
```

doesn't tell you whether that's normal.

Maybe the normal value is:

```text
450
```

or:

```text
20
```

Context matters.

---

# 4. First MySQL Health Check

Connect:

```bash
mysql -u root -p
```

Then:

```sql
SELECT VERSION();
```

Check uptime:

```sql
SHOW GLOBAL STATUS LIKE 'Uptime';
```

Check connections:

```sql
SHOW GLOBAL STATUS LIKE 'Threads_connected';
```

Check running threads:

```sql
SHOW GLOBAL STATUS LIKE 'Threads_running';
```

Check connection limit:

```sql
SHOW VARIABLES LIKE 'max_connections';
```

---

# 5. `Threads_connected` vs `Threads_running`

This distinction is important.

### Threads_connected

How many client connections are currently connected.

### Threads_running

How many threads are actively running work.

Imagine:

```text
Connected = 500
Running = 5
```

That doesn't necessarily mean 500 queries are executing.

It could mean:

```text
500 connections
     |
     +-- 495 idle
     |
     +-- 5 active
```

This is why you shouldn't look at one metric in isolation.

---

# 6. `SHOW PROCESSLIST`

One of the most useful commands:

```sql
SHOW PROCESSLIST;
```

For more detail:

```sql
SHOW FULL PROCESSLIST;
```

You'll see things such as:

```text
Id
User
Host
db
Command
Time
State
Info
```

Example conceptually:

```text
+------+---------+------+-------+-------+------+----------------+
| Id   | User    | db   | Cmd   | Time  |State | Info           |
+------+---------+------+-------+-------+------+----------------+
| 101  | app     | shop | Query | 120   | ...  | SELECT ...     |
| 102  | app     | shop | Sleep | 50    |      | NULL           |
+------+---------+------+-------+-------+------+----------------+
```

---

# 7. What Is a Long-Running Query?

Suppose:

```text
Query time = 900 seconds
```

That doesn't automatically mean:

> "Kill it."

First determine:

* What query?
* Who started it?
* Why is it running?
* Is it blocked?
* Is it doing useful work?
* Is it holding locks?
* Is it part of a critical transaction?
* Is the application waiting for it?

Never kill production queries blindly.

---

# 8. Finding Active Queries

You can query Performance Schema or use:

```sql
SHOW FULL PROCESSLIST;
```

You may also inspect statement activity through Performance Schema.

For example:

```sql
SELECT *
FROM performance_schema.threads;
```

and relevant statement tables.

We'll build more practical queries as we progress.

---

# 9. Performance Schema

`performance_schema` is a major MySQL instrumentation framework.

It helps collect information about:

* Statements
* Connections
* Waits
* Locks
* Memory
* Stages
* File I/O
* Metadata

Think:

```text
                 MySQL
                   |
          +--------+--------+
          |                 |
       Workload         Instrumentation
                             |
                             v
                    performance_schema
                             |
                             v
                        DBA analysis
```

---

# 10. The `sys` Schema

The `sys` schema makes performance information easier to consume.

It provides useful views for:

* Statements
* Connections
* I/O
* Memory
* Schema statistics
* Index usage
* Locks

For example, you may use:

```sql
SELECT *
FROM sys.schema_table_statistics;
```

or:

```sql
SELECT *
FROM sys.user_summary;
```

Exact available views depend on your MySQL version.

---

# 11. Why `sys` Is Useful

Raw Performance Schema data can be complicated.

Think:

```text
Performance Schema
       |
       v
Large amount of instrumentation
       |
       v
sys schema
       |
       v
DBA-friendly views
```

For everyday troubleshooting, `sys` can significantly reduce the amount of SQL you need to write.

---

# 12. Slow Query Log

The **slow query log** is one of the DBA's best performance tools.

It identifies queries that exceed configured criteria.

Conceptually:

```text
Query
  |
  v
Execution
  |
  +----> Fast → Normal
  |
  +----> Slow → Slow Query Log
```

Check whether it is enabled:

```sql
SHOW VARIABLES LIKE 'slow_query_log';
```

Check its location:

```sql
SHOW VARIABLES LIKE 'slow_query_log_file';
```

Check threshold:

```sql
SHOW VARIABLES LIKE 'long_query_time';
```

---

# 13. `long_query_time`

This controls the threshold used by the slow query log.

For example:

```text
long_query_time = 10
```

could mean queries exceeding approximately 10 seconds are eligible for logging, depending on other settings.

A production DBA might choose a much smaller threshold based on workload.

For example:

```text
0.5 seconds
1 second
2 seconds
```

There is no universal correct value.

---

# 14. Don't Enable General Logging Without a Reason

The general query log can record a very large amount of activity.

It can create:

```text
High I/O
Large log volume
Performance overhead
Storage pressure
```

Use it deliberately.

For normal query-performance investigation, the slow query log is often more appropriate.

---

# 15. `SHOW STATUS`

MySQL maintains many status counters.

For example:

```sql
SHOW GLOBAL STATUS LIKE 'Questions';
```

Connections:

```sql
SHOW GLOBAL STATUS LIKE 'Threads%';
```

Queries:

```sql
SHOW GLOBAL STATUS LIKE 'Queries';
```

Temporary tables:

```sql
SHOW GLOBAL STATUS LIKE 'Created_tmp%';
```

Table scans:

```sql
SHOW GLOBAL STATUS LIKE 'Handler_read%';
```

The key skill isn't memorizing every variable.

It's understanding:

> **What does this metric represent, and what does it tell me about the workload?**

---

# 16. CPU Troubleshooting — Linux

Suppose MySQL is slow.

Check CPU:

```bash
top
```

or:

```bash
htop
```

You can also inspect CPU-related statistics using tools such as:

```bash
vmstat 1
```

Questions:

* Is CPU saturated?
* Is MySQL consuming most CPU?
* Is another process consuming CPU?
* Is the system experiencing CPU steal time in a VM?
* Is load high because of CPU or blocked I/O?

---

# 17. CPU Troubleshooting — Windows

Use:

```text
Task Manager
```

and:

```text
Performance Monitor
```

Look at:

* CPU utilization
* MySQL process
* Memory
* Disk
* Network

For deeper investigation, Windows Performance Monitor provides counters that can be used for longer-term monitoring.

---

# 18. Memory Troubleshooting — Linux

Useful commands:

```bash
free -h
```

and:

```bash
vmstat 1
```

Look for:

* Available memory
* Swap usage
* Memory pressure
* Cache
* Paging activity

A server can have high memory utilization without necessarily being unhealthy because Linux uses memory for caching.

Don't panic based on one percentage.

---

# 19. Memory Troubleshooting — Windows

Use:

```text
Task Manager
```

or:

```text
Performance Monitor
```

Check:

* Available memory
* Committed memory
* Paging
* MySQL process memory
* Other major consumers

Again, context matters.

---

# 20. Disk Troubleshooting — Linux

First:

```bash
df -h
```

Then:

```bash
df -i
```

For I/O:

```bash
iostat -xz 1
```

You may need the appropriate system package for `iostat`.

Look at:

* Utilization
* Latency
* IOPS
* Throughput
* Queue depth

---

# 21. Disk Troubleshooting — Windows

Use:

* Task Manager
* Resource Monitor
* Performance Monitor

Look at:

* Disk active time
* Disk latency
* Read/write throughput
* Queue length
* Available capacity

A database can be CPU-light but extremely slow because storage is saturated.

---

# 22. InnoDB Buffer Pool

Now we're getting into MySQL internals.

Check:

```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
```

The buffer pool caches InnoDB data and indexes.

Conceptually:

```text
                RAM
        +----------------+
        |  Buffer Pool   |
        |                |
        | Data           |
        | Indexes        |
        | Pages          |
        +-------+--------+
                |
                v
               Disk
```

The goal isn't:

> "Make buffer pool as large as physically possible."

You need to leave memory for:

* MySQL connections
* Sort buffers
* Temporary memory
* Other MySQL structures
* OS
* Monitoring
* Backup software
* Other services

---

# 23. Buffer Pool Hit Ratio

You may encounter buffer pool efficiency metrics.

The basic idea:

```text
Requested data
      |
      +-- Found in memory → fast
      |
      +-- Not found → disk access
```

But don't obsess over a single "hit ratio" number.

Modern performance analysis should consider:

* Actual disk I/O
* Working set
* Query behavior
* Memory pressure
* Buffer pool activity

---

# 24. Transactions

Long-running transactions can cause serious problems.

Example:

```text
Transaction starts
      |
      v
UPDATE ...
      |
      |
      | 30 minutes
      |
      v
COMMIT
```

That transaction may retain:

* Locks
* Undo information
* Old row versions

and can affect other workloads.

---

# 25. Find Long-Running Transactions

You can inspect transaction information through `information_schema` and Performance Schema.

For example:

```sql
SELECT *
FROM information_schema.innodb_trx;
```

Important fields include transaction age and transaction state.

The exact columns depend on the MySQL version.

---

# 26. Locking

Suppose:

```text
Transaction A
     |
     +-- UPDATE customer 100
     |
     v
   Lock

Transaction B
     |
     +-- UPDATE customer 100
     |
     v
   WAITING
```

The application may report:

> "MySQL is slow."

But MySQL isn't necessarily slow.

Transaction B may simply be **waiting for Transaction A**.

---

# 27. Lock Waits

This distinction is critical:

```text
Slow query
    ≠
Blocked query
```

A query can be perfectly efficient but still take 60 seconds because it is waiting for a lock.

So your troubleshooting must distinguish:

```text
CPU-bound
I/O-bound
Lock-bound
Network-bound
```

---

# 28. Deadlocks

A deadlock occurs when transactions wait for one another in a cycle.

Conceptually:

```text
Transaction A
     |
     | holds Lock 1
     |
     v
 waits for Lock 2
     ^
     |
     | holds Lock 2
     |
Transaction B
```

Neither can proceed.

MySQL/InnoDB detects deadlocks and rolls back one transaction to break the cycle.

---

# 29. Deadlock Investigation

A useful command:

```sql
SHOW ENGINE INNODB STATUS;
```

Look for:

```text
LATEST DETECTED DEADLOCK
```

This can provide information about:

* Transactions
* Locks
* SQL statements
* Which transaction was rolled back

Deadlocks aren't automatically evidence of a broken database.

They can occur in legitimate concurrent workloads.

The goal is to understand **why the application workload creates the deadlock**.

---

# 30. Metadata Locks

Another important problem is a metadata lock.

Imagine:

```text
Transaction A
     |
     +-- Long transaction
     |
     +-- Holds metadata-related lock

DBA
 |
 +-- ALTER TABLE
```

The DDL may appear to be:

```text
Waiting
```

while applications appear to be:

```text
Slow
```

This is why large production schema changes require careful planning.

We'll cover online DDL in Lesson 9.

---

# 31. Query Optimization

Suppose:

```sql
SELECT *
FROM employees
WHERE department_id = 10;
```

The query may be slow because MySQL must examine many rows.

You need to ask:

```text
Is there a useful index?
Is the index selective?
How many rows are examined?
How many rows are returned?
Is the query written efficiently?
```

---

# 32. `EXPLAIN`

Use:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department_id = 10;
```

It provides information about the optimizer's planned execution.

Important columns include:

```text
type
possible_keys
key
rows
filtered
Extra
```

Don't memorize them yet.

You need to understand what the optimizer is attempting to do.

---

# 33. `EXPLAIN ANALYZE`

In supported MySQL versions, you can use:

```sql
EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE department_id = 10;
```

This provides actual execution information in addition to the optimizer's plan.

This distinction is valuable:

```text
EXPLAIN
   ↓
What optimizer expects

EXPLAIN ANALYZE
   ↓
What execution actually did
```

---

# 34. Full Table Scan

Suppose you have:

```text
employees = 50 million rows
```

and run:

```sql
SELECT *
FROM employees
WHERE email = 'john@example.com';
```

If there is no useful index on `email`, MySQL may need to examine a huge number of rows.

Conceptually:

```text
50,000,000 rows
       |
       v
Search many/all rows
       |
       v
Find one row
```

A suitable index can change the access pattern dramatically.

---

# 35. Index Trade-Off

Indexes improve some reads but increase write overhead.

Without index:

```text
SELECT
   ↓
Potentially expensive search
```

With index:

```text
INSERT
UPDATE
DELETE
   ↓
Index maintenance
```

So:

> **Don't create an index for every column just because an index might help one query.**

Indexes need to be designed around workload.

---

# 36. A Production Query Investigation

Suppose the application team reports:

> "The orders page is taking 20 seconds."

Your process:

### Step 1 — Confirm

Is the problem real?

```text
Application metrics
Database metrics
Query latency
```

### Step 2 — Identify Query

Find the SQL.

### Step 3 — Run EXPLAIN

```sql
EXPLAIN ...
```

### Step 4 — Inspect indexes

```sql
SHOW INDEX FROM orders;
```

### Step 5 — Check locks

Is the query actually waiting?

### Step 6 — Check system resources

CPU?

Memory?

Disk?

### Step 7 — Test

Run the query in a safe environment.

### Step 8 — Fix

Potentially:

* Index
* Query rewrite
* Schema change
* Application change
* Configuration
* Resource scaling

### Step 9 — Validate

Did latency improve?

---

# 37. Never Jump Straight to Configuration

Bad DBA troubleshooting:

```text
Database slow
     ↓
Increase max_connections
```

Why?

Because the actual problem might be:

```text
Missing index
```

Increasing connections can make an overloaded database **worse**.

Similarly:

```text
Database slow
     ↓
Increase buffer pool
```

may be wrong if:

```text
Disk is saturated
```

or:

```text
Query is doing a huge table scan
```

---

# 38. `max_connections`

Check:

```sql
SHOW VARIABLES LIKE 'max_connections';
```

This controls the maximum number of client connections MySQL permits.

But:

```text
More connections ≠ More performance
```

Imagine:

```text
CPU capacity = 8 cores

Connections = 2000
```

If thousands of queries compete for CPU, memory, locks, and I/O, simply increasing the connection limit won't solve the underlying issue.

---

# 39. Connection Storm

A common production problem:

```text
Application
    |
    +-- 50 connections
    +-- 50 connections
    +-- 50 connections
    +-- ...
    |
    v
MySQL
    |
    v
Connection exhaustion
```

Possible causes:

* Application bug
* Connection pool misconfiguration
* Traffic spike
* Network failures
* Retry storm
* Database latency causing connections to remain occupied

This is why you must investigate the application and database together.

---

# 40. Temporary Tables

Check:

```sql
SHOW GLOBAL STATUS LIKE 'Created_tmp%';
```

High temporary-table activity may indicate certain query patterns requiring memory/disk temporary work.

But again:

> A high counter by itself isn't necessarily a problem.

Compare:

```text
Rate
+
Workload
+
Configuration
+
Latency
```

---

# 41. Disk Space vs Disk Performance

These are different.

### Capacity

```bash
df -h
```

asks:

> How much space remains?

### Performance

```bash
iostat
```

helps answer:

> Is storage keeping up with workload?

You can have:

```text
Disk capacity = 80% free
```

but:

```text
Disk latency = terrible
```

The database can still be slow.

---

# 42. Error Logs

Always check the MySQL error log during unusual behavior.

Find location:

```sql
SHOW VARIABLES LIKE 'log_error';
```

Look for:

* Crashes
* InnoDB errors
* Disk errors
* Authentication problems
* Configuration problems
* Startup issues
* Replication errors
* Resource-related failures

The error log is evidence.

---

# 43. Monitoring Categories

A production monitoring solution should cover:

```text
                    Monitoring
                        |
      +-----------------+-----------------+
      |                 |                 |
   Database            OS             Application
      |                 |                 |
 Connections          CPU             Latency
 Queries              Memory          Errors
 Locks                Disk            Throughput
 Transactions         Network         Requests
```

Monitoring only MySQL is insufficient.

---

# 44. What Should You Alert On?

Good alerting might include:

```text
MySQL unavailable
Disk nearly full
Replication stopped
Replication lag too high
Backup failed
Connection exhaustion
High query latency
Long-running transaction
Deadlock rate abnormal
CPU saturation
Memory pressure
Storage latency
Error rate
```

Avoid creating alerts for every possible metric.

Too many alerts produce:

```text
Alert fatigue
```

---

# 45. Windows Performance Troubleshooting Workflow

Suppose MySQL on Windows is slow.

Check:

```text
1. Task Manager
2. Resource Monitor
3. Performance Monitor
4. Windows Event Viewer
5. MySQL error log
6. MySQL status
7. Processlist
8. Performance Schema
9. Slow Query Log
```

Architecture:

```text
Windows
 |
 +-- CPU
 +-- Memory
 +-- Disk
 +-- Network
 |
 +-- mysqld
       |
       +-- Queries
       +-- Locks
       +-- Transactions
       +-- InnoDB
```

---

# 46. Linux Performance Troubleshooting Workflow

For Linux:

```bash
top
```

```bash
free -h
```

```bash
vmstat 1
```

```bash
iostat -xz 1
```

```bash
df -h
```

```bash
df -i
```

Then inspect MySQL:

```sql
SHOW FULL PROCESSLIST;
```

```sql
SHOW ENGINE INNODB STATUS;
```

```sql
SHOW GLOBAL STATUS;
```

```sql
SHOW VARIABLES;
```

and Performance Schema/sys schema.

---

# 47. Production Incident Example

### Situation

Users report:

> "The application is extremely slow."

You discover:

```text
CPU: 35%
Memory: normal
Disk capacity: normal
Disk latency: normal
Connections: normal
```

So you investigate SQL.

You find:

```sql
SELECT *
FROM orders
WHERE customer_id = 12345;
```

`EXPLAIN` shows a very expensive access path.

You inspect:

```sql
SHOW INDEX FROM orders;
```

There is no suitable index.

Potential solution:

```text
Query
  ↓
Missing index
  ↓
Large scan
  ↓
High query latency
```

You test an appropriate index in a safe environment, validate the execution plan, assess write overhead/storage implications, then plan the production change.

This is the correct DBA thought process.

---

# 48. Production Incident Example #2 — Lock

Application reports:

> "UPDATE statements are hanging."

You inspect:

```sql
SHOW FULL PROCESSLIST;
```

You discover:

```text
Query A → running
Query B → waiting
Query C → waiting
Query D → waiting
```

Then you investigate transactions and locks.

You discover:

```text
Transaction A
     |
     +-- Long-running transaction
     |
     +-- Blocking others
```

Now you have a very different problem:

```text
Not primarily CPU
Not primarily disk
Not necessarily bad SQL
       ↓
Transaction/locking problem
```

This distinction is essential.

---

# 49. Performance Troubleshooting Decision Tree

Use this:

```text
             MYSQL SLOW
                 |
        +--------+--------+
        |                 |
      Queries?           System?
        |                 |
        v                 v
   Slow Query Log     CPU / Memory
   EXPLAIN             Disk / Network
   Indexes             OS
        |
        v
     Waiting?
        |
    +---+---+
    |       |
   Yes      No
    |       |
    v       v
 Locks    Query Plan
    |       |
    v       v
Transaction Index
    |
    v
Deadlock?
```

This is a good starting framework.

---

# 50. Your Lesson 5 Lab

Create:

```sql
CREATE DATABASE performance_lab;
```

Then:

```sql
USE performance_lab;

CREATE TABLE employees
(
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department_id INT,
    email VARCHAR(200)
);
```

Populate it with enough rows to make query behavior meaningful.

Then test:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE email = 'someone@example.com';
```

Create an appropriate index:

```sql
CREATE INDEX idx_employees_email
ON employees(email);
```

Run:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE email = 'someone@example.com';
```

Compare the plans.

The important lesson isn't:

> "Indexes make everything faster."

The lesson is:

> **The execution plan and workload determine whether an index is useful.**

---

# 51. Performance Lab — Processlist

Open two MySQL sessions.

### Session 1

Start a transaction and perform a controlled operation in your lab.

### Session 2

Run:

```sql
SHOW FULL PROCESSLIST;
```

Observe:

* Connection IDs
* Users
* Commands
* Time
* State
* SQL

This is how you'll begin understanding real concurrent activity.

---

# 52. Performance Lab — InnoDB

Run:

```sql
SHOW ENGINE INNODB STATUS;
```

Don't worry if it looks overwhelming.

Find:

```text
TRANSACTIONS
```

and:

```text
LATEST DETECTED DEADLOCK
```

when applicable.

We'll learn how to interpret these sections progressively.

---

# 53. Lesson 5 Interview Questions

You should be able to answer:

1. What does `SHOW PROCESSLIST` show?
2. What's the difference between `Threads_connected` and `Threads_running`?
3. What is Performance Schema?
4. What is the `sys` schema?
5. What is the slow query log?
6. What is `long_query_time`?
7. What is the InnoDB Buffer Pool?
8. What is a long-running transaction?
9. What is a lock wait?
10. What is a deadlock?
11. What does `SHOW ENGINE INNODB STATUS` provide?
12. What does `EXPLAIN` do?
13. What does `EXPLAIN ANALYZE` add?
14. Why isn't increasing `max_connections` always a performance solution?
15. How do you troubleshoot high CPU?
16. How do you troubleshoot disk I/O?
17. How do you distinguish a slow query from a blocked query?
18. Why should you establish a baseline?
19. Why should you investigate before changing configuration?
20. What would you check if users say "MySQL is slow"?

---

# 54. Your Production DBA Troubleshooting Formula

Memorize this:

```text
                 INCIDENT
                    ↓
              VERIFY PROBLEM
                    ↓
              CHECK BASELINE
                    ↓
          +---------+---------+
          |                   |
       MySQL                 OS
          |                   |
    Queries/Locks         CPU/Memory
    Transactions          Disk/Network
    Performance Schema
          |
          v
     FIND BOTTLENECK
          |
          v
     FORM HYPOTHESIS
          |
          v
        TEST
          |
          v
         FIX
          |
          v
       VALIDATE
          |
          v
       MONITOR
          |
          v
     DOCUMENT / RCA
```

That workflow is far more valuable than memorizing hundreds of MySQL variables.

---

# Lesson 5 Complete

You now have the foundation for **production performance troubleshooting**:

```text
OS
 ↓
MySQL
 ↓
InnoDB
 ↓
Transactions
 ↓
Locks
 ↓
Queries
 ↓
Indexes
 ↓
Execution Plans
 ↓
Monitoring
 ↓
Incident Response
```

## Next — Lesson 6: Patching, Maintenance & Version Management

We'll cover **Windows + Linux** in detail:

* MySQL release types
* LTS vs Innovation releases
* Patch vs minor/major upgrade
* Reading release notes
* Security patches
* Pre-patch checks
* Backup requirements
* Configuration backup
* Windows MSI patching
* Windows ZIP upgrade considerations
* Linux APT/DNF/YUM patching
* Repository management
* Service management
* Compatibility checks
* Rollback planning
* Post-patch validation
* Production maintenance windows
* **A complete production patching runbook and lab**
