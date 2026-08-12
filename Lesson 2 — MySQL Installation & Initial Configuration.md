# Lesson 2 — MySQL Installation & Initial Configuration

This is **Lesson 2 of 10**.

Today we'll build the foundation you will use for every later DBA task:

**Install → Configure → Start → Secure → Verify → Troubleshoot**

We'll cover both **Windows and Linux**.

---

# 1. Installation Strategy

As a DBA, don't think of installation as simply:

> "Run installer."

A production installation should be treated as:

```text
OS Preparation
      ↓
MySQL Package/Installer
      ↓
Data Directory
      ↓
Configuration
      ↓
Service
      ↓
Networking
      ↓
Security
      ↓
Validation
      ↓
Monitoring
```

Before installing, decide:

* MySQL version
* Operating system
* Installation method
* Data directory
* Log location
* Port
* Memory allocation
* Backup location
* Security requirements
* Whether this is development, test, or production

---

# 2. Supported Installation Approaches

## Windows

Common approaches:

1. **MySQL Installer**
2. **ZIP archive**

The Installer is convenient for learning and many standard Windows deployments.

The ZIP distribution gives you more manual control and is useful for understanding how MySQL works underneath.

---

## Linux

Common approaches:

### Debian/Ubuntu

APT packages:

```bash
sudo apt update
sudo apt install mysql-server
```

### RHEL-compatible systems

Depending on the distribution and repository configuration:

```bash
sudo dnf install mysql-server
```

Exact package names and repository choices can vary by OS/version, so in production you should always follow the MySQL-supported repository/package documentation for your particular platform.

---

# 3. Windows Installation — MySQL Installer

For the lab, the easiest approach is the official MySQL Installer.

During installation, you'll encounter options such as:

```text
Developer Default
Server Only
Client Only
Full
Custom
```

For a DBA lab, **Server Only** can be sufficient if you want a clean server environment.

A development installation can include additional client/development components.

---

# 4. Windows Installation Architecture

After installation, conceptually you'll have:

```text
Windows
 |
 +-- MySQL Server
 |      |
 |      +-- mysqld.exe
 |
 +-- MySQL Configuration
 |      |
 |      +-- my.ini
 |
 +-- MySQL Data Directory
 |
 +-- Windows Service
 |      |
 |      +-- MySQL80
 |
 +-- MySQL Client Tools
```

The exact service name can differ depending on the installation.

---

# 5. Windows MySQL Configuration

One important configuration file is:

```text
my.ini
```

A simplified example:

```ini
[mysqld]
port=3306
datadir=C:/ProgramData/MySQL/MySQL Server 8.0/Data
```

Don't blindly copy this path into your installation.

Your actual:

* MySQL version
* installation location
* data directory

may differ.

---

# 6. Windows Service

MySQL normally runs as a Windows service.

Open:

```text
Services
```

or run:

```text
services.msc
```

You should see your MySQL service.

From a DBA perspective:

```text
Windows Service
      |
      v
   mysqld
      |
      v
MySQL Instance
```

You can also manage services from an elevated command prompt/PowerShell.

For example, if the service is named `MySQL80`:

```powershell
Get-Service MySQL80
```

Start:

```powershell
Start-Service MySQL80
```

Stop:

```powershell
Stop-Service MySQL80
```

Restart:

```powershell
Restart-Service MySQL80
```

**Important:** Use the actual service name installed on your system.

---

# 7. Windows PATH

You want to be able to execute:

```powershell
mysql
```

without typing the complete installation path.

The MySQL `bin` directory might look similar to:

```text
C:\Program Files\MySQL\MySQL Server 8.0\bin
```

You can add the appropriate `bin` directory to the Windows PATH.

Then:

```powershell
mysql --version
```

should work.

You can also find the executable directly:

```powershell
where.exe mysql
```

---

# 8. Verify Windows Installation

Open PowerShell:

```powershell
mysql --version
```

You should receive a version response.

Then connect:

```powershell
mysql -u root -p
```

After authentication:

```sql
SELECT VERSION();
```

Then:

```sql
SHOW DATABASES;
```

If these work, your basic installation is functional.

---

# 9. Linux Installation

Now let's look at Linux.

For Ubuntu/Debian-style systems, a basic package installation may look like:

```bash
sudo apt update
sudo apt install mysql-server
```

After installation, check the service:

```bash
sudo systemctl status mysql
```

On some Linux distributions/install methods, the service can instead be named:

```bash
sudo systemctl status mysqld
```

Don't assume the service name—verify it for the particular package you're using.

---

# 10. Linux Service Architecture

Conceptually:

```text
Linux
 |
 +-- systemd
       |
       +-- mysql.service / mysqld.service
                 |
                 v
               mysqld
                 |
                 v
           MySQL Instance
```

Start:

```bash
sudo systemctl start mysql
```

Stop:

```bash
sudo systemctl stop mysql
```

Restart:

```bash
sudo systemctl restart mysql
```

Enable automatic startup:

```bash
sudo systemctl enable mysql
```

Check:

```bash
sudo systemctl status mysql
```

Again, substitute `mysqld` if that's the service name used by your installation.

---

# 11. Linux Configuration Files

You may encounter:

```text
/etc/my.cnf
```

or:

```text
/etc/mysql/my.cnf
```

and configuration directories such as:

```text
/etc/mysql/mysql.conf.d/
```

or:

```text
/etc/mysql/conf.d/
```

The exact layout depends on the distribution and installation method.

A useful question is:

> "Which configuration files is this MySQL instance actually reading?"

Don't simply edit the first `my.cnf` you find.

---

# 12. Finding MySQL Configuration

Once MySQL is running, you can inspect variables:

```sql
SHOW VARIABLES LIKE 'datadir';
```

For example:

```text
datadir
-------
/var/lib/mysql/
```

Find the port:

```sql
SHOW VARIABLES LIKE 'port';
```

Find the configuration-related values you care about:

```sql
SHOW VARIABLES LIKE 'log_error';
```

Find the server hostname:

```sql
SHOW VARIABLES LIKE 'hostname';
```

These commands are extremely useful because they tell you what the running server is actually using.

---

# 13. Data Directory

One of the first things you should learn to identify:

```sql
SHOW VARIABLES LIKE 'datadir';
```

Example:

```text
/var/lib/mysql/
```

On Windows, it could be something similar to:

```text
C:\ProgramData\MySQL\MySQL Server 8.0\Data\
```

But **never assume the path**.

Always verify.

Why is the data directory important?

Because it contains critical database files.

It also matters when:

* Moving storage
* Expanding storage
* Migrating servers
* Planning backups
* Troubleshooting disk usage
* Performing certain recovery procedures

---

# 14. Port Configuration

Default:

```text
3306
```

Check it:

```sql
SHOW VARIABLES LIKE 'port';
```

You can configure another port if required.

Example:

```ini
[mysqld]
port=3307
```

After changing a server configuration parameter that requires restart, you need to restart MySQL.

Then verify:

```sql
SHOW VARIABLES LIKE 'port';
```

---

# 15. Bind Address

This is extremely important for network security.

A MySQL server may listen on a particular network interface/address.

For example:

```ini
[mysqld]
bind-address=127.0.0.1
```

This generally means local connections only.

If configured to listen on an external interface, remote clients may be able to reach the server—subject to firewall and MySQL authentication/authorization.

Conceptually:

```text
Application
     |
     | Network
     v
Firewall
     |
     v
MySQL listening interface
     |
     v
MySQL authentication
     |
     v
Database
```

Never expose MySQL broadly to the Internet simply because you need remote access.

---

# 16. Windows Firewall

Suppose MySQL is listening on TCP 3306.

That does **not automatically mean** remote clients can connect.

You may have:

```text
Client
   |
   v
Windows Firewall
   |
   X
TCP 3306 blocked
```

or:

```text
Client
   |
   v
Windows Firewall
   |
   v
MySQL 3306
```

Firewall rules should be as restrictive as possible.

Ideally, permit only required source networks/hosts.

---

# 17. Linux Firewall

Depending on the Linux distribution, you might encounter:

```text
firewalld
ufw
nftables
iptables
```

For example, Ubuntu commonly uses UFW in many installations.

Check:

```bash
sudo ufw status
```

On systems using firewalld:

```bash
sudo firewall-cmd --list-all
```

Do not open port 3306 globally just to "make it work."

A production approach is:

```text
Application subnet
        |
        | TCP 3306
        v
Firewall
        |
        v
MySQL
```

not:

```text
Internet
   |
   | TCP 3306
   v
MySQL
```

---

# 18. MySQL Authentication

A database account is not simply:

```text
username + password
```

MySQL accounts are associated with a host context.

For example:

```text
'appuser'@'localhost'
```

and:

```text
'appuser'@'10.10.10.50'
```

can represent different accounts.

This is an extremely important DBA concept.

You might see:

```sql
SELECT User, Host
FROM mysql.user;
```

The host component affects where that account can connect from.

---

# 19. Root Account

During installation, you'll configure administrative access.

The administrative account must be protected carefully.

Don't use the root account for normal application activity.

Instead:

```text
                 MySQL
                   |
       +-----------+-----------+
       |           |           |
      DBA       Application   Reporting
       |           |           |
   Admin        Limited      Read-only
 privileges    privileges    privileges
```

We'll cover proper account/role design in **Lesson 3**.

---

# 20. Initial Security

After installation, your initial security checklist should include:

* Protect administrative credentials
* Remove unnecessary anonymous access if present
* Remove unnecessary test/sample databases if applicable
* Restrict network access
* Use TLS where appropriate
* Create separate application accounts
* Apply least privilege
* Avoid using administrative accounts in applications
* Configure appropriate auditing/monitoring

A useful rule:

> **Installation isn't complete until security is considered.**

---

# 21. MySQL Version Verification

From the shell:

```bash
mysql --version
```

From SQL:

```sql
SELECT VERSION();
```

Also:

```sql
SHOW VARIABLES LIKE 'version%';
```

You should record the version as part of your DBA inventory.

For example:

```text
Server: PROD-DB-01
OS: Linux
MySQL: 8.x
Port: 3306
Data directory: /var/lib/mysql/
```

This kind of information becomes extremely useful during incidents, patching, and upgrades.

---

# 22. MySQL Startup Troubleshooting

This is a critical DBA skill.

Suppose:

```bash
sudo systemctl start mysql
```

fails.

**Don't immediately reinstall MySQL.**

Investigate.

First:

```bash
sudo systemctl status mysql
```

Then inspect logs:

```bash
sudo journalctl -u mysql
```

Depending on the setup, also inspect the MySQL error log.

Check:

```sql
SHOW VARIABLES LIKE 'log_error';
```

if the server is running.

Common startup problems include:

```text
Invalid configuration
       ↓
Wrong file permissions
       ↓
Port already in use
       ↓
Insufficient disk space
       ↓
Data directory problem
       ↓
InnoDB recovery issue
       ↓
SELinux/AppArmor issue
       ↓
Corrupt/missing files
```

---

# 23. Port Already in Use

One common problem:

```text
MySQL cannot start
because port 3306 is already occupied.
```

On Linux:

```bash
sudo ss -lntp | grep 3306
```

or:

```bash
sudo lsof -i :3306
```

On Windows:

```powershell
netstat -ano | findstr :3306
```

Then identify the process.

This is a classic DBA troubleshooting scenario.

---

# 24. Disk Space

Another extremely common production issue:

```text
Disk = 100%
```

Linux:

```bash
df -h
```

Then:

```bash
df -i
```

to inspect inode usage.

Windows:

Check the drive's free space and use Windows storage/system tools.

Why is this important?

MySQL may fail or behave badly when the filesystem has insufficient free space.

---

# 25. Configuration Changes

As a DBA, don't randomly edit configuration files.

Use this workflow:

```text
Identify requirement
       ↓
Check current value
       ↓
Understand parameter
       ↓
Determine dynamic/static
       ↓
Change configuration
       ↓
Validate syntax/configuration
       ↓
Restart if required
       ↓
Verify
       ↓
Monitor
```

For example:

```sql
SHOW VARIABLES LIKE 'max_connections';
```

Then determine whether and how the desired change should be applied.

---

# 26. Dynamic vs Static Variables

Some MySQL variables can be changed while the server is running.

Others require a restart or configuration-file change.

Conceptually:

```text
Dynamic
   |
   +--> Change at runtime
   |
   +--> May persist only if configured appropriately

Static
   |
   +--> Configuration change
   |
   +--> Restart
```

We'll study this much more deeply when we start performance tuning.

---

# 27. Initial DBA Health Check

After installation, perform a basic health check.

### Version

```sql
SELECT VERSION();
```

### Uptime

```sql
SHOW GLOBAL STATUS LIKE 'Uptime';
```

### Port

```sql
SHOW VARIABLES LIKE 'port';
```

### Data directory

```sql
SHOW VARIABLES LIKE 'datadir';
```

### Error log

```sql
SHOW VARIABLES LIKE 'log_error';
```

### Storage engines

```sql
SHOW ENGINES;
```

### Databases

```sql
SHOW DATABASES;
```

### Current connections

```sql
SHOW STATUS LIKE 'Threads_connected';
```

This is the beginning of a DBA health-check script.

---

# 28. Windows Installation Validation

Your Windows lab should eventually look like:

```text
Windows
 |
 +-- MySQL Service
 |       |
 |       +-- Running
 |
 +-- MySQL Server
 |       |
 |       +-- Port 3306
 |       +-- Data Directory
 |       +-- Configuration
 |
 +-- MySQL Client
 |
 +-- Logs
 |
 +-- Firewall
```

Verify:

```powershell
mysql --version
```

Then:

```powershell
mysql -u root -p
```

And:

```sql
SELECT VERSION();
SHOW DATABASES;
SHOW VARIABLES LIKE 'port';
SHOW VARIABLES LIKE 'datadir';
```

---

# 29. Linux Installation Validation

Your Linux lab:

```text
Linux
 |
 +-- systemd
 |     |
 |     +-- MySQL service
 |
 +-- mysqld
 |
 +-- Configuration
 |
 +-- Data directory
 |
 +-- Logs
 |
 +-- Firewall
```

Verify:

```bash
sudo systemctl status mysql
```

Then:

```bash
mysql -u root -p
```

And:

```sql
SELECT VERSION();
SHOW DATABASES;
SHOW VARIABLES LIKE 'port';
SHOW VARIABLES LIKE 'datadir';
```

---

# 30. Your Lesson 2 Lab

You now need to actually perform this.

## Windows

Install MySQL and verify:

```text
[ ] MySQL Server installed
[ ] MySQL service exists
[ ] MySQL service is running
[ ] mysql client works
[ ] Root/admin login works
[ ] SELECT VERSION() works
[ ] Port verified
[ ] Data directory verified
[ ] Configuration file located
[ ] Error log location identified
```

## Linux

Install MySQL and verify:

```text
[ ] MySQL Server installed
[ ] MySQL service exists
[ ] MySQL service is running
[ ] mysql client works
[ ] Administrative login works
[ ] SELECT VERSION() works
[ ] Port verified
[ ] Data directory verified
[ ] Configuration file located
[ ] Error log location identified
```

---

# 31. DBA Challenge

Don't just run the commands.

For each server, create your own inventory:

```text
Hostname:
Operating System:
MySQL Version:
MySQL Service Name:
MySQL Port:
Data Directory:
Configuration File:
Error Log:
MySQL Binary Location:
MySQL Client Version:
```

Then answer:

1. How do you start MySQL on Windows?
2. How do you stop it?
3. How do you start it on Linux?
4. How do you identify the MySQL service name?
5. How do you determine the data directory?
6. How do you determine the MySQL port?
7. How do you find the error log?
8. How do you determine whether port 3306 is already being used?
9. How do you check available disk space?
10. What is the difference between `mysql` and `mysqld`?
11. What is `my.ini`?
12. What is `my.cnf`?
13. What does `bind-address` control?
14. Why shouldn't you expose port 3306 to the entire Internet?
15. What should you investigate if MySQL fails to start?

---

## Lesson 2 — The DBA Principle

Remember this workflow:

```text
INSTALL
   ↓
CONFIGURE
   ↓
START
   ↓
SECURE
   ↓
VERIFY
   ↓
MONITOR
   ↓
DOCUMENT
```

**Never consider "the installer completed" to mean "the database server is production-ready."**

### Next — Lesson 3

We'll move into **MySQL Security & Access Administration**:

* Users
* `mysql.user`
* Authentication
* Passwords
* Roles
* `GRANT`
* `REVOKE`
* `SHOW GRANTS`
* DBA accounts
* Application accounts
* Read-only accounts
* Remote access
* Host restrictions
* TLS/SSL
* Account locking
* Password expiration
* Least privilege
* Security troubleshooting
* Real-world access-control scenarios
