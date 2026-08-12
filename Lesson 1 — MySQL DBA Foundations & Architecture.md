# Lesson 1 — MySQL DBA Foundations & Architecture

Welcome to **Lesson 1 of 10**.

Our goal today is to understand **what you are administering before you start administering it**. A good DBA needs to know what is happening underneath every command.

---

## 1. What is a MySQL DBA?

A **MySQL Database Administrator (DBA)** is responsible for keeping MySQL databases:

* Available
* Secure
* Fast
* Recoverable
* Consistent
* Properly configured
* Properly backed up
* Up to date
* Scalable
* Monitored

A DBA typically works across five areas:

```text
                MySQL DBA
                   |
     +-------------+-------------+
     |             |             |
 Availability   Performance   Security
     |             |             |
 Backup/DR      Monitoring    Access
     |
 Patching / Upgrades / Migration
```

In production, a DBA may receive requests such as:

> "The application is slow."

> "The database server is 95% full."

> "We need to create a read-only user."

> "Upgrade MySQL."

> "Move this database from Windows to Linux."

> "Someone accidentally deleted a table."

> "Replication has stopped."

> "We need a backup strategy."

Your job is to determine **what happened, why it happened, and how to fix it safely**.

---

# 2. What is MySQL?

MySQL is a **relational database management system (RDBMS)**.

Applications send SQL statements to MySQL:

```text
Application
     |
     | SQL
     v
 MySQL Server
     |
     v
Database
     |
     v
Tables / Indexes / Data
```

For example:

```sql
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

The application doesn't directly manipulate the database files.

Instead:

```text
Application
     |
     v
MySQL Server
     |
     +--> SQL processing
     |
     +--> Authentication
     |
     +--> Optimization
     |
     +--> Transaction management
     |
     +--> Storage engine
     |
     v
Disk
```

Understanding this architecture is fundamental to DBA work.

---

# 3. MySQL Server vs MySQL Client

This distinction is extremely important.

### MySQL Server

The **server** is the actual database engine.

It:

* Accepts connections
* Authenticates users
* Executes SQL
* Manages transactions
* Manages locks
* Reads/writes data
* Maintains logs
* Manages memory
* Manages storage engines

The main server process is commonly:

```text
mysqld
```

### MySQL Client

The client is a program used to connect to MySQL.

For example:

```bash
mysql -u root -p
```

Architecture:

```text
+-------------------+
|   MySQL Client    |
|       mysql       |
+---------+---------+
          |
          | TCP/IP / Socket
          |
          v
+-------------------+
|   MySQL Server    |
|      mysqld       |
+-------------------+
          |
          v
       Storage
```

The client and server can exist on the **same machine or different machines**.

---

# 4. What is a MySQL Instance?

This is a very important DBA concept.

A **MySQL instance** generally refers to one running MySQL server environment.

For example:

```text
Server
 |
 +-- MySQL Instance
       |
       +-- Port 3306
       +-- Data directory
       +-- Configuration
       +-- Logs
       +-- Databases
```

You can also run multiple MySQL instances on one operating system.

For example:

```text
Windows Server
 |
 +-- MySQL Instance 1
 |      Port 3306
 |      Data directory A
 |
 +-- MySQL Instance 2
        Port 3307
        Data directory B
```

Likewise on Linux.

This becomes useful when you need isolated MySQL environments on the same physical/virtual server.

---

# 5. What is a Database?

A database is a logical container for objects such as:

* Tables
* Views
* Stored procedures
* Functions
* Triggers
* Events

For example:

```text
CompanyDB
 |
 +-- employees
 +-- departments
 +-- salaries
 +-- locations
```

You can create a database with:

```sql
CREATE DATABASE CompanyDB;
```

Then:

```sql
USE CompanyDB;
```

---

# 6. Database vs Schema

In MySQL, **database and schema are effectively synonymous**.

These are equivalent:

```sql
CREATE DATABASE sales;
```

and:

```sql
CREATE SCHEMA sales;
```

You'll commonly see both terms in documentation and conversations.

So when someone says:

> "Create a schema."

in MySQL, they generally mean:

> "Create a database."

---

# 7. Tables

Tables hold structured data.

Example:

```text
employees
------------------------------------------------
employee_id | employee_name | department_id
------------------------------------------------
1           | John           | 10
2           | Sarah          | 20
3           | David          | 10
```

SQL:

```sql
CREATE TABLE employees
(
    employee_id INT,
    employee_name VARCHAR(100),
    department_id INT
);
```

Insert data:

```sql
INSERT INTO employees
VALUES (1, 'John', 10);
```

Retrieve it:

```sql
SELECT *
FROM employees;
```

---

# 8. Primary Keys

A **primary key** uniquely identifies a row.

Example:

```sql
CREATE TABLE employees
(
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department_id INT
);
```

Here:

```text
employee_id
     |
     +-- uniquely identifies each employee
```

Good database design and good indexing are important to DBA work because poor schema design can cause major performance problems.

---

# 9. Indexes

An index helps MySQL find data efficiently.

Without an appropriate index, MySQL may need to examine many rows.

Example:

```sql
SELECT *
FROM employees
WHERE employee_id = 10000;
```

If `employee_id` has an appropriate index, MySQL can locate the row efficiently.

Conceptually:

```text
Without index:

1 -> 2 -> 3 -> 4 -> 5 -> ... -> 10000


With index:

              Index
                |
                v
              10000
                |
                v
               Row
```

But indexes aren't free.

They consume:

* Disk space
* Memory
* CPU
* Write resources

Therefore, DBAs must balance indexing with workload requirements.

---

# 10. Storage Engines

One of the most important MySQL concepts is the **storage engine**.

The storage engine is responsible for how data is stored and managed.

Common engines include:

### InnoDB

This is the primary storage engine you should know for modern MySQL administration.

It supports:

* Transactions
* ACID properties
* Row-level locking
* Foreign keys
* Crash recovery
* MVCC

Typical production databases use InnoDB.

### MyISAM

Older storage engine.

It does not provide the same transaction capabilities as InnoDB and is generally not the default choice for modern transactional applications.

For this course, **InnoDB will be our primary focus**.

---

# 11. MySQL Architecture

Now let's put the major pieces together.

```text
                     Client
                       |
                       v
              +----------------+
              | Connection     |
              | Management     |
              +-------+--------+
                      |
                      v
              +----------------+
              | SQL Layer      |
              | Parser         |
              | Optimizer      |
              +-------+--------+
                      |
                      v
              +----------------+
              | Storage Engine |
              |    InnoDB      |
              +-------+--------+
                      |
          +-----------+-----------+
          |                       |
          v                       v
       Memory                   Disk
          |                       |
          |                  Data Files
          |                  Redo Logs
          |                  Undo
          |                  Tablespaces
          |
          v
      Buffer Pool
```

You don't need to memorize every component today.

But understand the flow:

**Client → MySQL → SQL processing → Storage engine → Memory/Disk**

---

# 12. The InnoDB Buffer Pool

This is one of the most important performance concepts you'll encounter.

The **InnoDB Buffer Pool** is an area of memory used to cache frequently accessed data and indexes.

Conceptually:

```text
                RAM
        +-------------------+
        |   Buffer Pool     |
        |                   |
        | Data Pages        |
        | Index Pages       |
        | Cached Information|
        +---------+---------+
                  |
                  v
                 Disk
```

Suppose the application requests a row.

If the required page is already in memory:

```text
Application
     |
     v
MySQL
     |
     v
Buffer Pool
     |
     v
Return data
```

If it isn't available:

```text
Application
     |
     v
MySQL
     |
     v
Buffer Pool
     |
     | cache miss
     v
Disk
     |
     v
Buffer Pool
     |
     v
Application
```

Disk access is generally much slower than RAM.

Therefore, buffer pool configuration becomes extremely important for database performance.

We'll go much deeper into this in **Lesson 5**.

---

# 13. Transactions

A transaction is a logical unit of work.

Example:

```sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 100
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE account_id = 2;

COMMIT;
```

The idea is:

```text
Account A: -100
Account B: +100

       ↓

Both operations succeed
       ↓
     COMMIT
```

If something goes wrong:

```sql
ROLLBACK;
```

The changes can be undone according to transaction semantics.

---

# 14. ACID

For database administration, you need to understand **ACID**.

### A — Atomicity

A transaction is treated as a unit.

### C — Consistency

The database moves from one valid state to another.

### I — Isolation

Concurrent transactions should not improperly interfere with one another.

### D — Durability

Committed data should survive failures according to the database's durability guarantees.

Think:

```text
A = All or nothing
C = Valid state
I = Transactions don't improperly interfere
D = Committed data survives
```

---

# 15. MySQL Data Directory

The **data directory** is where MySQL maintains its database-related files.

Conceptually:

```text
MySQL Data Directory
 |
 +-- Database A
 |     |
 |     +-- tables/data
 |
 +-- Database B
 |
 +-- mysql
 |
 +-- performance_schema
 |
 +-- sys
 |
 +-- undo / redo-related files
 |
 +-- other MySQL metadata
```

The exact file layout depends on the MySQL version, configuration, storage engine, and platform.

**Never manually manipulate MySQL data files unless you specifically understand the operation and the supported procedure.**

A DBA should generally use MySQL's administrative tools rather than casually copying/deleting files.

---

# 16. MySQL System Databases

You will encounter several important schemas.

### `mysql`

Contains important server metadata such as:

* User accounts
* Privileges
* Roles
* Other system information

### `information_schema`

Provides metadata about databases and objects.

For example:

```sql
SELECT *
FROM information_schema.tables;
```

### `performance_schema`

Provides instrumentation for monitoring MySQL internals and performance.

### `sys`

Provides easier-to-use views built around performance information.

As a DBA, you'll use these extensively.

---

# 17. MySQL Port

The default MySQL TCP port is:

```text
3306
```

For example:

```text
Application
    |
    | TCP 3306
    v
MySQL Server
```

But you can configure another port:

```text
3307
3308
...
```

A DBA needs to understand:

* Firewall rules
* Listening ports
* Network interfaces
* DNS
* Client connectivity
* Authentication

A database can be perfectly healthy internally while applications cannot connect because of a firewall or network problem.

---

# 18. Configuration Files

MySQL configuration varies by operating system.

Typical Linux locations include:

```text
/etc/my.cnf
/etc/mysql/my.cnf
/etc/mysql/mysql.conf.d/
```

Windows commonly uses:

```text
my.ini
```

The exact location depends on how MySQL was installed.

Important configuration parameters include:

```text
port
datadir
bind-address
server_id
log_error
slow_query_log
general_log
innodb_buffer_pool_size
max_connections
```

We'll learn these properly during Lesson 2.

---

# 19. MySQL Logs

Logs are essential to a DBA.

Important log types include:

### Error Log

Contains important server errors, warnings, startup/shutdown information, etc.

### General Query Log

Can record client activity.

Useful for troubleshooting in specific situations, but potentially expensive/noisy.

### Slow Query Log

Records queries that meet configured criteria for being considered slow.

Extremely useful for performance analysis.

### Binary Log

Extremely important for:

* Replication
* Point-in-time recovery
* Data-change history

Conceptually:

```text
Application
     |
     v
MySQL
     |
     +----> Error Log
     |
     +----> Slow Query Log
     |
     +----> Binary Log
```

We'll use these extensively in later lessons.

---

# 20. MySQL Startup and Shutdown

A DBA must understand the lifecycle of the server.

### Startup

```text
Operating System
      |
      v
Start MySQL
      |
      v
Read configuration
      |
      v
Initialize server
      |
      v
Initialize storage
      |
      v
Open for connections
```

### Shutdown

```text
Stop request
     |
     v
Stop accepting work
     |
     v
Finish/handle active work
     |
     v
Flush required data
     |
     v
Shutdown
```

There are safe and unsafe ways to stop a database.

We will cover proper procedures on **Windows and Linux in Lesson 2**.

---

# 21. Windows vs Linux — DBA Perspective

You will administer both.

### Windows

You'll work with:

```text
Windows Services
PowerShell
Command Prompt
my.ini
Windows Event Viewer
Windows Firewall
Task Manager
Performance Monitor
```

You may see a MySQL service such as:

```text
MySQL80
```

### Linux

You'll commonly work with:

```text
systemd
systemctl
journalctl
my.cnf
Linux permissions
firewalld
SELinux/AppArmor
top
iostat
vmstat
df
free
```

For example:

```bash
systemctl status mysqld
```

or on some distributions:

```bash
systemctl status mysql
```

Don't memorize the exact service name yet—we'll determine that from the installation in Lesson 2.

---

# 22. DBA's Mental Model

When something goes wrong, don't immediately start changing configuration.

Use this model:

```text
             USER REPORT
                  |
                  v
          What is actually wrong?
                  |
        +---------+---------+
        |                   |
       DB?               Outside DB?
        |                   |
        v                   v
     MySQL             OS / Network /
     internals          Application /
                          Storage
        |
        v
    Find evidence
        |
        v
    Identify cause
        |
        v
    Fix safely
        |
        v
    Validate
        |
        v
    Document
```

This is the mindset I want you to develop throughout the course.

---

# 23. Essential DBA Commands — Preview

Don't worry about memorizing these yet.

Once connected to MySQL:

```sql
SELECT VERSION();
```

Check the current database:

```sql
SELECT DATABASE();
```

List databases:

```sql
SHOW DATABASES;
```

Show tables:

```sql
SHOW TABLES;
```

Show server variables:

```sql
SHOW VARIABLES;
```

Show running processes:

```sql
SHOW PROCESSLIST;
```

Check status:

```sql
SHOW STATUS;
```

Exit:

```sql
EXIT;
```

These commands will become second nature.

---

# 24. Your First DBA Lab

For this course, I recommend building a small lab containing:

```text
                    DBA LAB

       +---------------------------+
       | Windows MySQL             |
       |                           |
       | MySQL Server              |
       +---------------------------+

                    +

       +---------------------------+
       | Linux MySQL               |
       |                           |
       | MySQL Server              |
       +---------------------------+
```

You can use:

* A Windows machine/VM
* A Linux VM such as Ubuntu
* VirtualBox/VMware/Hyper-V
* A cloud VM if available

**Do not practice destructive DBA operations on a production database.**

We'll deliberately break things later—but only in the lab.

---

# 25. Lesson 1 Practical Exercise

Before moving to Lesson 2, make sure you understand these questions.

### Exercise A — Concepts

Answer these yourself:

1. What is MySQL Server?
2. What is the MySQL client?
3. What is a MySQL instance?
4. What is a database/schema?
5. What is a table?
6. What is a primary key?
7. What is an index?
8. What is InnoDB?
9. What is the InnoDB Buffer Pool?
10. What is the MySQL data directory?
11. What is port 3306?
12. What is the error log?
13. What is the slow query log?
14. What is the binary log?
15. What are ACID properties?
16. What is the difference between Windows `my.ini` and Linux MySQL configuration?
17. Why shouldn't you manually delete MySQL data files?
18. What is the difference between a database and a table?

---

# 26. DBA Interview Questions From Lesson 1

You should eventually be able to answer these confidently:

**Q1. What is MySQL?**

An RDBMS that stores and manages relational data using SQL.

**Q2. What is InnoDB?**

MySQL's primary transactional storage engine, supporting features such as transactions, row-level locking, foreign keys, crash recovery, and MVCC.

**Q3. What is the default MySQL port?**

`3306`.

**Q4. What is the Buffer Pool?**

A major InnoDB memory area used to cache data and indexes, reducing disk I/O.

**Q5. What is the binary log used for?**

Among other things, replication and point-in-time recovery.

**Q6. What is the difference between a client and server?**

The client connects to MySQL and submits requests; the server processes those requests and manages the database.

**Q7. What is a transaction?**

A logical unit of database work that can be committed or rolled back.

---

# 27. What You Should Know Before Lesson 2

You should now have this mental picture:

```text
                    MySQL DBA
                        |
          +-------------+-------------+
          |             |             |
       Server         Storage       Security
          |             |             |
       Config         InnoDB       Users
       Network        Buffer       Privileges
       Logs           Pool
          |             |
          +------+------+
                 |
             Operations
                 |
      +----------+----------+
      |          |          |
    Backup    Patching   Upgrade
      |          |          |
      +----------+----------+
                 |
              Recovery
                 |
                HA
```

### Lesson 1 takeaway

The most important concept is this:

> **A DBA isn't just someone who knows SQL. A DBA manages the entire database service—server, configuration, storage, security, performance, backups, recovery, upgrades, operating system interaction, and availability.**

---

## Next: Lesson 2 — Installation & Initial Configuration

In **Lesson 2**, we'll get hands-on with:

* MySQL installation on **Windows**
* MySQL installation on **Linux**
* MSI vs ZIP on Windows
* APT vs DNF/YUM on Linux
* Creating/configuring the data directory
* `my.ini` vs `my.cnf`
* Windows Services
* `systemctl`
* Starting/stopping/restarting MySQL
* Setting the root/admin account
* `PATH`
* Port configuration
* `bind-address`
* Basic firewall configuration
* Initial security
* Verifying a successful installation
* **Windows and Linux troubleshooting when MySQL fails to start**

We will **not skip installation details** before moving into advanced administration.
