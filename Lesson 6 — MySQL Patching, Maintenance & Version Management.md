# Lesson 6 — MySQL Patching, Maintenance & Version Management

This lesson teaches you how a DBA safely manages MySQL versions in **production**, including **Windows and Linux**.

The key idea:

> **Patching is not "installing a new package." Patching is a controlled change to a production database system.**

---

# 1. MySQL Versioning

First, understand the terminology.

A MySQL version might look like:

```text
8.4.6
```

Conceptually:

```text
8       .       4       .       6
│               │               │
Major           Release         Patch
generation      series          level
```

For modern MySQL, Oracle distinguishes **LTS** and **Innovation** release tracks.

For production environments, organizations commonly prefer an **LTS release** when they prioritize longer-term stability and support.

Before patching, always verify the exact release/support information for your target version.

---

# 2. Why Patch MySQL?

You patch for several reasons:

```text
Security
   +
Bug fixes
   +
Stability
   +
Performance improvements
   +
Compatibility
   +
Support lifecycle
```

Security fixes are especially important.

A DBA should never have a policy of:

> "We only upgrade when something breaks."

---

# 3. Patch vs Upgrade

These terms are often confused.

### Patch

Usually moving to a newer maintenance release within the same release series.

Example conceptually:

```text
8.4.x
  ↓
8.4.y
```

### Upgrade

Moving between release series.

Example:

```text
8.0
 ↓
8.4
```

This generally requires more extensive compatibility testing.

---

# 4. Upgrade Complexity

Think:

```text
Patch
  ↓
Lower risk
  ↓
Still requires testing
```

versus:

```text
Major/Release-series upgrade
  ↓
Schema compatibility
SQL behavior
Authentication
Configuration
Drivers
Applications
Replication
Backup/restore
  ↓
More testing
```

Never assume:

> "It worked on my test server, therefore production is safe."

---

# 5. Your Patching Lifecycle

A professional DBA follows:

```text
1. Identify target version
2. Read release notes
3. Review known issues
4. Check compatibility
5. Inventory environment
6. Back up
7. Test
8. Prepare rollback
9. Schedule maintenance
10. Patch
11. Validate
12. Monitor
13. Document
```

This is your core patching runbook.

---

# 6. Step 1 — Inventory

Before touching anything, collect:

```text
MySQL version
OS version
Architecture
Installation method
Configuration
Data directory
Log locations
Service name
Plugins
Authentication configuration
Replication
Applications
Drivers/connectors
Backup method
Monitoring
```

Start with:

```sql
SELECT VERSION();
```

Also:

```sql
SHOW VARIABLES LIKE 'datadir';
```

```sql
SHOW VARIABLES LIKE 'log_error';
```

```sql
SHOW VARIABLES LIKE 'port';
```

---

# 7. Record the Configuration

Important configuration:

```sql
SHOW VARIABLES;
```

For targeted information:

```sql
SHOW VARIABLES LIKE 'innodb%';
```

```sql
SHOW VARIABLES LIKE 'max_connections';
```

```sql
SHOW VARIABLES LIKE 'character_set%';
```

```sql
SHOW VARIABLES LIKE 'collation%';
```

You should also preserve the actual MySQL configuration file.

Linux is commonly:

```text
/etc/my.cnf
/etc/mysql/my.cnf
/etc/mysql/mysql.conf.d/
```

depending on distribution and installation.

Windows commonly uses a configuration file such as:

```text
my.ini
```

The actual location depends on how MySQL was installed.

---

# 8. Don't Modify Configuration Blindly

Before patching, record:

```text
Current value
Desired value
Reason
```

For example:

```text
innodb_buffer_pool_size
Current: X
Purpose: memory allocation
```

Why?

Because after an upgrade you want to distinguish:

```text
Upgrade changed behavior
```

from:

```text
Configuration was accidentally changed
```

---

# 9. Step 2 — Read Release Notes

Before installing anything, review the release documentation for the target release.

Look for:

```text
Security fixes
Bug fixes
Known issues
Deprecated features
Removed features
Changed defaults
InnoDB changes
Authentication changes
Replication changes
Upgrade requirements
Operating-system support
```

This is mandatory DBA behavior.

---

# 10. Step 3 — Compatibility

Check:

```text
Application
     |
     v
MySQL Connector/Driver
     |
     v
MySQL Server
     |
     v
Operating System
```

All four matter.

For example:

```text
Application works with MySQL 8.0
```

doesn't automatically mean:

```text
Application works identically with MySQL 8.4
```

---

# 11. Check Application Drivers

Common drivers include:

```text
JDBC
ODBC
.NET
PHP
Python
Go
Node.js
```

Before an upgrade, verify that the application's connector/driver version supports the target MySQL release.

A database upgrade can fail operationally even if the MySQL server itself starts successfully.

---

# 12. Step 4 — Check Replication

If replication exists, determine:

```text
Primary
Replicas
Replication topology
GTID
Binary logging
Replication filters
Replication users
Lag
```

Check:

```sql
SHOW VARIABLES LIKE 'gtid_mode';
```

```sql
SHOW VARIABLES LIKE 'log_bin';
```

On a replica, inspect:

```sql
SHOW REPLICA STATUS\G
```

Exact output depends on your MySQL version and topology.

---

# 13. Step 5 — Check Backup

Before patching:

> **Have a known-good backup.**

But don't stop there.

Ask:

```text
When was the last successful backup?
Has it been validated?
Can it be restored?
Are required binary logs available?
```

Your patching strategy should never depend on:

> "I think the backup works."

---

# 14. Step 6 — Check Disk Space

Upgrades may require temporary storage.

Linux:

```bash
df -h
```

Also inspect inode availability:

```bash
df -i
```

Windows:

```text
This PC
→ Disk capacity
```

and use Resource Monitor/PowerShell where appropriate.

You need room for:

```text
Package
Temporary files
Logs
Backup
Rollback artifacts
```

---

# 15. Step 7 — Test the Upgrade

The best test environment resembles production:

```text
Production
   |
   +-- Same MySQL release
   +-- Similar configuration
   +-- Similar schema
   +-- Representative data
   +-- Same application version
   +-- Same connectors
```

Then:

```text
Test upgrade
      ↓
Run validation
      ↓
Run application tests
      ↓
Performance test
      ↓
Review errors
```

---

# 16. Upgrade Testing Isn't Just "Does MySQL Start?"

A successful startup is only step one.

Validate:

```text
MySQL starts
       ↓
Connections work
       ↓
Authentication works
       ↓
Schemas accessible
       ↓
Queries work
       ↓
Application works
       ↓
Stored programs work
       ↓
Events work
       ↓
Replication works
       ↓
Backups work
       ↓
Monitoring works
```

---

# 17. Windows Installation Methods

On Windows, MySQL may be installed using:

```text
MSI installer
ZIP archive
MySQL Installer
Other managed deployment mechanisms
```

Your upgrade procedure depends on the installation method.

You should identify how the existing server was installed **before** choosing an upgrade procedure.

---

# 18. Windows — MSI/Installer Concept

Conceptually:

```text
Current MySQL
     |
     v
Backup
     |
     v
Stop service
     |
     v
Install/update package
     |
     v
Start service
     |
     v
Validate
```

But don't blindly follow these steps against production.

The exact upgrade behavior depends on the MySQL version, installation type, service configuration, and target release.

---

# 19. Windows Service

Find the MySQL service.

You can use:

```text
services.msc
```

or PowerShell:

```powershell
Get-Service *mysql*
```

You might see something such as:

```text
MySQL80
```

The actual service name can differ.

---

# 20. Windows Service Commands

Depending on how the service is configured:

```powershell
Get-Service MySQL80
```

Stop:

```powershell
Stop-Service MySQL80
```

Start:

```powershell
Start-Service MySQL80
```

Check:

```powershell
Get-Service MySQL80
```

Do not assume the service name is `MySQL80`.

Always verify.

---

# 21. Windows ZIP Installation

With a ZIP-based installation, the upgrade process may involve:

```text
Old MySQL binaries
        |
        v
New MySQL binaries
        |
        v
Existing data directory
        |
        v
Configuration
```

This gives you more control but also more responsibility.

You must understand:

* Data directory
* `my.ini`
* Service registration
* PATH
* Plugins
* DLL dependencies
* File permissions

---

# 22. Never Treat the Data Directory Like an Application Folder

A critical principle:

```text
MySQL binaries
      ≠
MySQL data
```

Conceptually:

```text
C:\MySQL\
     |
     +-- bin\
     |
     +-- lib\

C:\MySQLData\
     |
     +-- InnoDB data
     +-- Redo logs
     +-- System tables
     +-- User databases
```

Your actual layout may be different.

The important point is:

> **Know exactly where your data lives before changing binaries.**

---

# 23. Linux Installation Methods

Common approaches include:

```text
APT
DNF/YUM
Oracle MySQL repositories
Distribution packages
Manual installation
```

Examples:

Ubuntu/Debian:

```bash
apt
```

RHEL-family:

```bash
dnf
```

The exact supported package/repository procedure depends on your Linux distribution and MySQL release.

---

# 24. Linux — Check Package

For Debian/Ubuntu-style systems:

```bash
dpkg -l | grep mysql
```

For RPM-based systems:

```bash
rpm -qa | grep mysql
```

You can also inspect the installed client/server:

```bash
mysql --version
```

and:

```bash
mysqld --version
```

---

# 25. Linux — Service Management

Modern Linux systems commonly use systemd.

Check:

```bash
systemctl status mysqld
```

or, depending on distribution:

```bash
systemctl status mysql
```

Again:

> **Don't assume the service name.**

---

# 26. Linux Package Update Concept

For a maintenance release within a supported package stream, a package-manager-based environment might follow:

```text
Repository
   ↓
Package metadata
   ↓
Target version
   ↓
Package update
   ↓
MySQL service restart
   ↓
Validation
```

For example, on a controlled Debian/Ubuntu environment you might use the system's package-management workflow.

On RPM-based systems, the equivalent workflow uses the appropriate `dnf`/repository configuration.

The exact command should be determined from your **specific OS + MySQL installation method** rather than copied blindly.

---

# 27. Repository Management

For production:

```text
Production
    |
    v
Approved MySQL repository
    |
    v
Controlled version
```

Don't allow:

```text
Random repository
       ↓
Unexpected MySQL version
       ↓
Production
```

Repository configuration is part of change management.

---

# 28. Version Pinning / Controlled Updates

In production, you generally want deliberate version control.

Conceptually:

```text
Current
8.4.x

Target
8.4.y
```

rather than:

```text
"Install whatever the repository happens to provide."
```

This is especially important when managing large fleets.

---

# 29. Configuration Compatibility

One of the most common upgrade problems:

```text
Old my.cnf
      +
New MySQL
      =
Potential incompatibility
```

Parameters can become:

```text
Deprecated
Removed
Changed
Ignored
Invalid
```

Therefore:

> Review configuration against the target release.

Don't simply copy an ancient configuration file onto a new server and assume everything is correct.

---

# 30. Configuration Migration

A safer approach:

```text
Old configuration
       |
       v
Review each important setting
       |
       v
Target-version documentation
       |
       v
New/updated configuration
```

Maintain a change record:

```text
Parameter
Old value
New value
Reason
```

---

# 31. Authentication Changes

Authentication deserves special attention during upgrades.

Check:

```sql
SELECT user, host, plugin
FROM mysql.user;
```

You want to know:

```text
Which users exist?
Which authentication plugins are used?
Are applications compatible?
Are administrators compatible?
```

Don't discover authentication incompatibility after production is upgraded.

---

# 32. SQL Mode

Check:

```sql
SELECT @@sql_mode;
```

SQL mode can affect application behavior.

For example:

```text
Invalid data
Date handling
GROUP BY behavior
Type conversion
```

An application that depends on old behavior may encounter problems after a version change.

---

# 33. Character Sets and Collations

Check:

```sql
SHOW VARIABLES LIKE 'character_set%';
```

and:

```sql
SHOW VARIABLES LIKE 'collation%';
```

Also inspect important schemas/tables.

Changes in character-set/collation behavior can cause:

```text
Comparison differences
Sorting differences
Index limitations
Application behavior differences
```

These deserve explicit testing.

---

# 34. Pre-Patching Checklist

Before the maintenance window:

```text
[ ] Target version approved
[ ] Release notes reviewed
[ ] Security issues reviewed
[ ] OS compatibility verified
[ ] Application compatibility verified
[ ] Connector compatibility verified
[ ] Backup completed
[ ] Backup validated
[ ] Binary logs protected
[ ] Configuration backed up
[ ] Disk space checked
[ ] Monitoring checked
[ ] Replication checked
[ ] Rollback plan prepared
[ ] Maintenance window approved
[ ] Stakeholders notified
```

This checklist should become second nature.

---

# 35. Production Maintenance Window

A typical sequence:

```text
T-7 days
  ↓
Testing

T-1 day
  ↓
Backup verification

T-30 min
  ↓
Final health check

T-0
  ↓
Application drain/stop if required

T+5
  ↓
MySQL maintenance

T+15
  ↓
Startup

T+20
  ↓
Validation

T+30
  ↓
Application testing

T+45
  ↓
Monitoring

T+60
  ↓
Close/change completion
```

Actual timing depends entirely on environment.

---

# 36. Pre-Change Health Check

Before patching, capture:

```sql
SELECT VERSION();
```

```sql
SHOW GLOBAL STATUS LIKE 'Threads_connected';
```

```sql
SHOW GLOBAL STATUS LIKE 'Threads_running';
```

```sql
SHOW VARIABLES LIKE 'read_only';
```

If replication exists, capture its status.

Also check:

```text
CPU
Memory
Disk
Disk space
Application health
Backup status
```

Why?

Because after patching you need a **before/after comparison**.

---

# 37. Stop the Application Gracefully

If the application is running, don't necessarily just kill MySQL.

Prefer:

```text
Application traffic
       ↓
Drain
       ↓
Stop application writes
       ↓
Confirm active workload reduced
       ↓
Perform database maintenance
```

This can reduce:

* Active transactions
* Connection count
* User impact
* Recovery complexity

---

# 38. Stop MySQL

Windows:

```powershell
Stop-Service MySQL80
```

Linux:

```bash
systemctl stop mysqld
```

or:

```bash
systemctl stop mysql
```

Use the actual service name.

Confirm it stopped:

```bash
systemctl status mysqld
```

or Windows service tools.

---

# 39. Apply the Patch

This is the platform-specific step.

### Windows

Use your approved installation/update method.

### Linux

Use the approved repository/package workflow.

The important DBA principle:

> **Don't improvise the package-management procedure during the production maintenance window.**

Test it beforehand.

---

# 40. Start MySQL

Linux:

```bash
systemctl start mysqld
```

Windows:

```powershell
Start-Service MySQL80
```

Then check:

```bash
systemctl status mysqld
```

or:

```powershell
Get-Service MySQL80
```

---

# 41. First Validation — Version

Immediately:

```sql
SELECT VERSION();
```

Verify that the running server is actually the expected version.

Never assume the package update means the running server changed.

---

# 42. Second Validation — Error Log

Inspect the error log.

Look for:

```text
Startup errors
InnoDB errors
Plugin errors
Configuration errors
Authentication errors
Upgrade warnings
```

A successful service start doesn't mean there are no problems.

---

# 43. Third Validation — Connectivity

Test:

```bash
mysql -u testuser -p
```

Then:

```sql
SELECT 1;
```

Check an application user.

Why?

Because:

```text
root login works
```

doesn't prove:

```text
application login works
```

---

# 44. Fourth Validation — Database Objects

Run:

```sql
SHOW DATABASES;
```

Then:

```sql
USE application_db;
```

Check important tables:

```sql
SHOW TABLES;
```

Check critical data:

```sql
SELECT COUNT(*)
FROM important_table;
```

---

# 45. Fifth Validation — Application

This is mandatory.

Test:

```text
Login
Read
Insert
Update
Delete where appropriate
Transactions
Reports
Critical business workflow
```

For example:

```text
User login
   ↓
Create order
   ↓
Payment
   ↓
Order retrieval
   ↓
Report
```

Database upgrade testing must include real application workflows.

---

# 46. Sixth Validation — Performance

Compare:

```text
Before
CPU
Latency
Connections
Queries/sec
Slow queries

After
CPU
Latency
Connections
Queries/sec
Slow queries
```

If performance changed significantly, investigate.

---

# 47. Seventh Validation — Backup

After a successful upgrade:

> **Verify that your backup process still works.**

Why?

Because an upgrade can affect:

* Backup tools
* Authentication
* Paths
* Permissions
* Plugins
* Utilities

A server that's upgraded but no longer backed up is not fully operational.

---

# 48. Eighth Validation — Monitoring

Check:

```text
Monitoring agent
Metrics
Alerts
Log collection
Dashboards
Health checks
```

Don't forget this.

A production system can appear healthy while monitoring is silently broken.

---

# 49. Rollback

This is one of the most misunderstood areas.

For some patching operations, rollback can be as simple as restoring the previous package/binaries **if the data files remain compatible**.

For other upgrades, especially release-series upgrades, you may **not** be able to safely downgrade the data directory.

Therefore:

> **Never assume downgrade is a valid rollback strategy.**

---

# 50. Better Rollback Strategy

Before the change:

```text
Current Server
      |
      +--> Verified Backup
      |
      +--> Configuration backup
      |
      +--> Binary logs
      |
      +--> Documented old version
```

If the upgrade fails:

```text
Incident
   ↓
Stop
   ↓
Assess
   ↓
Use approved rollback/recovery procedure
```

For major upgrades, the rollback plan may involve:

```text
New server
     X

Old server / restored copy
     ↓
Recovery
     ↓
Application
```

rather than simply installing the old package.

---

# 51. Rollback Must Be Tested

A rollback plan that exists only in a document isn't enough.

Test:

```text
Upgrade
   ↓
Failure simulation
   ↓
Rollback/recovery
   ↓
Application validation
```

This is similar to our Lesson 4 principle:

> **Untested recovery is an assumption.**

---

# 52. Patching a Standalone Server

Simple topology:

```text
Application
    |
    v
MySQL
```

Maintenance:

```text
Backup
 ↓
Stop application
 ↓
Stop MySQL
 ↓
Patch
 ↓
Start MySQL
 ↓
Validate
 ↓
Start application
 ↓
Monitor
```

---

# 53. Patching a Replication Topology

Now imagine:

```text
             Primary
                |
          +-----+-----+
          |           |
       Replica 1   Replica 2
```

You don't necessarily patch everything simultaneously.

A controlled strategy may involve:

```text
Replica
   ↓
Patch
   ↓
Validate
   ↓
Failover/role change if appropriate
   ↓
Patch former primary
   ↓
Validate
```

The exact strategy depends on your topology, MySQL version, replication design, application requirements, and HA tooling.

We'll go much deeper into this in **Lesson 8**.

---

# 54. Why Replicas Make Maintenance Easier

Without a replica:

```text
MySQL
  |
  X
Maintenance
  |
  X
Application unavailable
```

With appropriate HA architecture:

```text
Primary
   |
Replica
   |
Maintenance / failover
   |
Application
```

You can potentially reduce downtime significantly.

But replication isn't automatically HA.

We'll cover this in detail later.

---

# 55. Patching Mistakes to Avoid

### Mistake 1

> "It's just a minor update."

Wrong.

Even maintenance releases require testing.

---

### Mistake 2

> "The package installed successfully."

That doesn't mean MySQL is healthy.

---

### Mistake 3

> "MySQL started, so we're done."

No.

You still need application validation.

---

### Mistake 4

> "We have a backup."

Was it tested?

---

### Mistake 5

> "We can always downgrade."

Not necessarily.

---

### Mistake 6

> "We'll figure out the rollback during the outage."

Never.

---

# 56. Production Patching Runbook

Here is a practical template.

## PRE-CHECK

```text
1. Confirm change ticket
2. Confirm target version
3. Review release notes
4. Verify compatibility
5. Check backup
6. Check replication
7. Check disk space
8. Record configuration
9. Record current version
10. Capture health metrics
```

## CHANGE

```text
11. Notify stakeholders
12. Drain application
13. Stop application if required
14. Stop MySQL
15. Apply package/binaries
16. Apply required upgrade procedure
17. Start MySQL
```

## POST-CHECK

```text
18. Verify version
19. Check error log
20. Test connectivity
21. Test authentication
22. Test databases
23. Test critical SQL
24. Test application
25. Check performance
26. Check replication
27. Check monitoring
28. Check backups
```

## CLOSE

```text
29. Monitor
30. Document result
31. Record final version
32. Record issues
33. Close change
```

---

# 57. Hands-On Lab — Windows

On a **non-production Windows VM**:

### Step 1

Find the service:

```powershell
Get-Service *mysql*
```

### Step 2

Check server version:

```bash
mysql --version
```

Then:

```sql
SELECT VERSION();
```

### Step 3

Locate configuration and data directory:

```sql
SHOW VARIABLES LIKE 'datadir';
```

```sql
SHOW VARIABLES LIKE 'log_error';
```

### Step 4

Back up your configuration.

### Step 5

Take a database backup using the Lesson 4 procedure.

### Step 6

Record:

```text
Version
Configuration
Data directory
Service
Port
Backup
```

**Do not perform an actual version upgrade on a production Windows server as part of this lab.**

---

# 58. Hands-On Lab — Linux

On a non-production Linux VM:

```bash
mysql --version
```

Then:

```bash
systemctl status mysqld
```

If your distribution uses `mysql`:

```bash
systemctl status mysql
```

Check:

```bash
df -h
```

Then:

```bash
free -h
```

Then:

```bash
uname -a
```

Collect:

```text
OS
MySQL version
Service
Disk
Memory
Configuration
Data directory
Error log
```

This is the same information you'd gather before a production change.

---

# 59. Build Your Own Pre-Upgrade Report

Create a document containing:

```text
SERVER
--------------------
Hostname:
OS:
Architecture:

MYSQL
--------------------
Version:
Port:
Data directory:
Error log:
Config:

REPLICATION
--------------------
Role:
GTID:
Binary logging:
Replica status:

RESOURCES
--------------------
CPU:
RAM:
Disk:
Disk free:

BACKUP
--------------------
Last successful backup:
Backup location:
Restore test date:

APPLICATION
--------------------
Application version:
Driver:
Critical workflows:

TARGET
--------------------
Current:
Target:
Reason:
Maintenance window:

ROLLBACK
--------------------
Procedure:
Owner:
Validation:
```

This is a real DBA habit worth developing.

---

# 60. Lesson 6 Interview Questions

You should now be able to answer:

1. What is the difference between patching and upgrading?
2. What is an LTS release?
3. Why should production systems generally have a controlled release strategy?
4. What should you check before patching?
5. Why should you read release notes?
6. Why is application-driver compatibility important?
7. Why should configuration be backed up?
8. Why shouldn't you blindly copy an old `my.cnf`/`my.ini` to a new release?
9. How do you identify the MySQL data directory?
10. How do you identify the MySQL error log?
11. How do you check MySQL's current version?
12. How do you check the MySQL service on Windows?
13. How do you check the MySQL service on Linux?
14. Why is rollback not always a simple downgrade?
15. What should you validate after patching?
16. Why should you test backups after an upgrade?
17. Why should you test the application after MySQL starts?
18. What is a maintenance window?
19. What should a production patching runbook contain?
20. How would you patch a replicated environment differently from a standalone server?

---

# The DBA Rule to Remember

```text
             PATCHING
                 |
       +---------+---------+
       |         |         |
    BEFORE     DURING     AFTER
       |         |         |
    Backup     Control   Validate
    Test       Change    Monitor
    Plan       Window    Document
    Rollback
```

The most dangerous DBA is the one who knows **how to install MySQL** but doesn't know **how to safely change production MySQL**.

The professional DBA knows both.

---

# 10-Lesson Roadmap

We've completed:

```text
✅ Lesson 1 — MySQL Architecture & DBA Fundamentals
✅ Lesson 2 — Installation & Initial Configuration
✅ Lesson 3 — Users, Security & Configuration
✅ Lesson 4 — Backup, Restore & Disaster Recovery
✅ Lesson 5 — Monitoring, Performance & Troubleshooting
✅ Lesson 6 — Patching, Maintenance & Version Management
```

Remaining:

```text
➡️ Lesson 7 — MySQL Migration: Windows ↔ Linux, Server-to-Server,
               Dump/Load, MySQL Shell, Zero/Low-Downtime Concepts

➡️ Lesson 8 — Replication, High Availability & Disaster Recovery

➡️ Lesson 9 — Advanced Administration: InnoDB, Storage, Schema Changes,
               Security Hardening & Production Operations

➡️ Lesson 10 — Complete Production DBA: End-to-End Architecture,
                Incident Response, Upgrade/Migration Project,
                Automation, Runbooks & DBA Interview/Certification Prep
```

**Lesson 7 is the big migration lesson:** we'll build a practical DBA migration methodology covering **Windows → Linux, Linux → Windows, same-version migration, cross-version migration, dump/load, MySQL Shell, validation, cutover, rollback, and downtime minimization.**
