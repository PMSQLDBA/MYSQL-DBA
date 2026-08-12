# Lesson 3 — MySQL Users, Security & Access Administration

This is **Lesson 3 of 10**.

Today we're going to build the security foundation of a MySQL environment.

The DBA goal is:

> **Give the right person/application the minimum permissions required, from the right location, using a secure authentication method.**

Our progression:

```text
Accounts
   ↓
Authentication
   ↓
Privileges
   ↓
Roles
   ↓
Remote Access
   ↓
TLS
   ↓
Auditing
   ↓
Security Validation
```

---

# 1. MySQL Security Has Multiple Layers

Don't think of MySQL security as just a password.

A connection passes through several layers:

```text
Client
  |
  v
Network / Firewall
  |
  v
MySQL Listener
  |
  v
Authentication
  |
  v
Authorization
  |
  v
Database Objects
```

For example:

```text
Application
    |
    | TCP 3306
    v
Firewall
    |
    v
MySQL
    |
    | Who are you?
    v
Authentication
    |
    | What are you allowed to do?
    v
Authorization
    |
    v
Database
```

A secure MySQL environment needs all of these layers.

---

# 2. Authentication vs Authorization

These two terms are fundamental.

### Authentication

**Who are you?**

Example:

```text
Username: appuser
Password: ********
```

### Authorization

**What are you allowed to do?**

For example:

```text
appuser
   |
   +-- SELECT
   +-- INSERT
   +-- UPDATE
```

but perhaps:

```text
appuser
   X-- DROP DATABASE
   X-- CREATE USER
   X-- SHUTDOWN
```

So:

```text
Authentication = Identity
Authorization  = Permissions
```

---

# 3. MySQL Accounts

A MySQL account is conceptually:

```text
'user'@'host'
```

This is extremely important.

For example:

```sql
'appuser'@'localhost'
```

is not necessarily the same account as:

```sql
'appuser'@'10.10.10.20'
```

And:

```sql
'appuser'@'%'
```

is another account definition.

The `Host` component is part of MySQL account matching.

---

# 4. See Existing Accounts

As an administrator, you can inspect accounts with:

```sql
SELECT User, Host
FROM mysql.user;
```

You may see something like:

```text
+----------------+-----------+
| User           | Host      |
+----------------+-----------+
| root           | localhost |
| appuser        | localhost |
+----------------+-----------+
```

Do not modify the `mysql` system tables directly just because you can query them.

Use supported SQL account-management statements such as:

```sql
CREATE USER
ALTER USER
DROP USER
GRANT
REVOKE
```

This distinction is important for safe administration.

---

# 5. Creating a User

Basic syntax:

```sql
CREATE USER 'appuser'@'localhost'
IDENTIFIED BY 'StrongPasswordHere';
```

For production, don't use a password literally like the example above.

Use a strong, securely managed credential.

Then test:

```sql
SHOW GRANTS FOR 'appuser'@'localhost';
```

A newly created account generally has very limited privileges until you grant appropriate access.

---

# 6. Create Different Types of Accounts

A production environment should generally separate accounts.

For example:

```text
                   MySQL
                     |
        +------------+------------+
        |            |            |
       DBA       Application    Reporting
        |            |            |
      Admin       Limited       Read-only
```

You might have:

```text
dba_admin
app_orders
reporting_user
backup_user
monitoring_user
```

Each should have only the privileges necessary for its purpose.

---

# 7. Never Use the DBA Account for Applications

Bad architecture:

```text
Application
     |
     v
root
     |
     +-- SELECT
     +-- INSERT
     +-- UPDATE
     +-- DELETE
     +-- DROP
     +-- CREATE USER
     +-- GRANT
     +-- ...
```

If the application credentials are compromised, the attacker effectively gets administrative database access.

Better:

```text
Application
     |
     v
app_orders
     |
     +-- SELECT
     +-- INSERT
     +-- UPDATE
     +-- DELETE
```

while:

```text
DBA
 |
 +-- Administrative account
```

is used only for administration.

---

# 8. Understanding GRANT

`GRANT` gives permissions.

For example:

```sql
GRANT SELECT
ON companydb.*
TO 'reporting_user'@'localhost';
```

This means the user can perform `SELECT` against objects in:

```text
companydb.*
```

---

# 9. Database-Level Permissions

You can grant permissions at a database level.

For example:

```sql
GRANT SELECT, INSERT, UPDATE
ON sales.*
TO 'sales_app'@'localhost';
```

Conceptually:

```text
sales_app
    |
    +-- SELECT
    +-- INSERT
    +-- UPDATE
    |
    v
sales database
```

It does not automatically mean:

```text
DROP USER
CREATE USER
SHUTDOWN
```

Those are separate administrative capabilities.

---

# 10. Table-Level Permissions

You can restrict permissions to specific tables.

Example:

```sql
GRANT SELECT
ON sales.customers
TO 'reporting_user'@'localhost';
```

Now you're applying a much narrower permission.

This is useful when an account only needs access to a specific object.

---

# 11. Column-Level Permissions

MySQL also supports more granular privilege scenarios.

For example:

```sql
GRANT SELECT (customer_id, customer_name)
ON sales.customers
TO 'reporting_user'@'localhost';
```

This can be useful when certain columns contain sensitive information.

However, for complex security requirements, database design, views, and application architecture should also be considered.

---

# 12. `SHOW GRANTS`

One of your most important DBA commands:

```sql
SHOW GRANTS FOR 'appuser'@'localhost';
```

This tells you what permissions have been granted to that account.

When troubleshooting:

> "Why can't this user do X?"

one of the first things you should investigate is:

```sql
SHOW GRANTS FOR 'username'@'host';
```

---

# 13. REVOKE

If permissions are no longer required:

```sql
REVOKE UPDATE
ON sales.*
FROM 'appuser'@'localhost';
```

Then verify:

```sql
SHOW GRANTS FOR 'appuser'@'localhost';
```

Security isn't just about granting permissions.

It's also about **removing permissions that are no longer required**.

---

# 14. Roles

Roles make privilege management much easier in larger environments.

Instead of:

```text
User A → 15 privileges
User B → 15 privileges
User C → 15 privileges
User D → 15 privileges
```

you can define a role:

```text
reporting_role
     |
     +-- SELECT database A
     +-- SELECT database B
```

Then assign the role:

```text
User A ──┐
User B ──┼──> reporting_role
User C ──┘
```

This simplifies administration.

---

# 15. Creating a Role

Example:

```sql
CREATE ROLE 'reporting_role';
```

Grant privileges to the role:

```sql
GRANT SELECT
ON sales.*
TO 'reporting_role';
```

Then assign it:

```sql
GRANT 'reporting_role'
TO 'reporting_user'@'localhost';
```

You can inspect role/user grants with:

```sql
SHOW GRANTS FOR 'reporting_user'@'localhost';
```

---

# 16. Why Roles Matter

Imagine a company with 200 reporting users.

Without roles:

```text
200 users
×
multiple privileges
=
large administration problem
```

With roles:

```text
                 reporting_role
                 /     |      \
                /      |       \
             User1   User2    User3
```

When the role changes, you can manage the privilege model centrally.

---

# 17. Least Privilege

This is one of the most important security principles in DBA work.

### Bad

```text
Application
     |
     v
ALL PRIVILEGES
```

### Better

```text
Application
     |
     v
Required permissions only
```

For example, if an application only needs:

```text
SELECT
INSERT
UPDATE
```

don't automatically give:

```text
DROP
ALTER
CREATE USER
GRANT OPTION
SHUTDOWN
```

Least privilege reduces the blast radius of:

* Credential theft
* Application vulnerabilities
* Human mistakes
* SQL injection
* Insider threats

---

# 18. Remote Access

Suppose the application is on:

```text
10.10.10.50
```

and MySQL is on:

```text
10.10.10.100
```

The application needs network access.

Architecture:

```text
Application
10.10.10.50
      |
      | TCP 3306
      v
Firewall
      |
      v
MySQL
10.10.10.100
```

You need all of these to work:

1. MySQL listening on an appropriate interface
2. Network routing
3. Firewall rules
4. MySQL account matching
5. Authentication
6. Authorization

---

# 19. `localhost` vs Remote Host

Consider:

```sql
'appuser'@'localhost'
```

That account is associated with local connections.

Now:

```sql
'appuser'@'10.10.10.50'
```

is associated with connections from that host.

This is why you may encounter:

> "The username and password are correct, but the application still can't connect."

The account/host combination may not match the incoming connection.

---

# 20. The `%` Host Wildcard

You may encounter:

```sql
'appuser'@'%'
```

The `%` is a wildcard host pattern.

It can potentially allow connections from many hosts, subject to other network controls.

This is convenient but can be overly broad.

Avoid:

```text
'appuser'@'%'
```

when you can safely use a more restrictive source.

For example:

```text
'appuser'@'10.10.10.50'
```

or an appropriately designed restricted host/network arrangement.

---

# 21. Firewall + MySQL Security

Never rely on MySQL permissions alone.

You want defense in depth:

```text
             Internet
                |
                X
          Firewall blocks
                |
          Application Network
                |
                v
         MySQL Firewall Rule
                |
                v
        MySQL Authentication
                |
                v
        MySQL Authorization
```

Security should exist at multiple layers.

---

# 22. Password Management

Never store production passwords in:

```text
Plain text scripts
Git repositories
Public documentation
Chat messages
Shell history
Application source code
```

Instead, use appropriate secrets-management mechanisms.

A production environment might use:

```text
Application
     |
     v
Secret Manager
     |
     v
Credential
     |
     v
MySQL
```

Your exact implementation depends on your infrastructure.

---

# 23. Account Locking

MySQL provides account-management capabilities for controlling account status.

For example:

```sql
ALTER USER 'appuser'@'localhost'
ACCOUNT LOCK;
```

Unlock:

```sql
ALTER USER 'appuser'@'localhost'
ACCOUNT UNLOCK;
```

This is useful when an account needs to be temporarily disabled without necessarily deleting it.

---

# 24. Password Expiration

MySQL supports password-expiration controls.

For example, an administrator can configure password expiration policies using `ALTER USER`.

Conceptually:

```text
Account
   |
   +-- Password
   |
   +-- Expiration policy
   |
   +-- Lock status
   |
   +-- Authentication method
```

Security policies should match your organization's requirements rather than blindly applying arbitrary expiration periods.

---

# 25. Authentication Plugins

MySQL supports authentication mechanisms/plugins.

The exact default authentication method depends on the MySQL version and installation.

You can inspect account authentication information through appropriate account metadata.

For example:

```sql
SELECT User, Host, plugin
FROM mysql.user;
```

This is useful when troubleshooting authentication compatibility.

---

# 26. TLS / SSL

For sensitive environments, network encryption is important.

Without encryption:

```text
Client
   |
   | potentially exposed network traffic
   v
MySQL
```

With TLS:

```text
Client
   |
   | encrypted TLS connection
   v
MySQL
```

TLS helps protect data in transit.

You should consider:

* Server certificate
* CA certificate
* Client verification
* Certificate expiration
* TLS configuration
* Encrypted replication connections where appropriate

We'll revisit TLS during advanced administration.

---

# 27. Check Whether SSL/TLS Is Being Used

Depending on your MySQL version and connection setup, you can inspect connection/session information and server TLS variables.

For example:

```sql
SHOW SESSION STATUS LIKE 'Ssl%';
```

You can also inspect TLS-related server variables:

```sql
SHOW VARIABLES LIKE '%ssl%';
```

The exact variables exposed depend on the MySQL version.

---

# 28. Security Auditing

As a DBA, you need to know:

> Who accessed the database?

> What did they do?

> When did they do it?

> Was the action authorized?

Possible sources include:

* MySQL logs
* Performance Schema
* General query logging
* Enterprise auditing capabilities where applicable
* OS logging
* SIEM/security infrastructure

Don't enable expensive logging blindly in production.

Logging must balance:

```text
Security
   +
Troubleshooting
   +
Performance
   +
Storage
   +
Privacy
```

---

# 29. Dangerous Privileges

Some privileges deserve particular attention.

Examples include privileges that can enable:

* User administration
* Granting privileges
* Database structure changes
* Server administration
* Broad data access

An application should almost never have unrestricted administrative privileges.

The exact risk depends on the privilege and MySQL version.

---

# 30. Example: Application Account

Imagine an application named:

```text
OrdersApp
```

Database:

```text
orders
```

The application needs:

```text
SELECT
INSERT
UPDATE
DELETE
```

You might create:

```sql
CREATE USER 'orders_app'@'10.10.10.50'
IDENTIFIED BY 'StrongSecret';
```

Then:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON orders.*
TO 'orders_app'@'10.10.10.50';
```

Verify:

```sql
SHOW GRANTS FOR 'orders_app'@'10.10.10.50';
```

Notice the important design:

```text
Specific user
+
Specific source
+
Specific database
+
Specific privileges
```

That's much better than:

```text
'user'@'%'
+
ALL PRIVILEGES
```

---

# 31. Example: Reporting User

Reporting usually needs read access.

```sql
CREATE USER 'reporting'@'10.10.20.50'
IDENTIFIED BY 'StrongSecret';
```

Then:

```sql
GRANT SELECT
ON orders.*
TO 'reporting'@'10.10.20.50';
```

Now:

```text
Reporting
    |
    +-- SELECT
    |
    X-- INSERT
    X-- UPDATE
    X-- DELETE
    X-- DROP
```

This is a good example of least privilege.

---

# 32. Example: DBA Role

For your lab, you can create a DBA role and grant appropriate administrative privileges.

However, **don't automatically give every administrator unrestricted access in production**.

A real organization may separate:

```text
Junior DBA
Senior DBA
Security DBA
Backup operator
Monitoring account
Automation account
```

based on responsibilities.

---

# 33. Removing an Account

When an account is no longer required:

```sql
DROP USER 'olduser'@'localhost';
```

But don't immediately delete accounts during an incident without understanding their purpose.

First investigate:

```text
Who owns it?
What application uses it?
What permissions does it have?
When was it last used?
Is it referenced by automation?
```

Security changes can themselves cause outages.

---

# 34. Security Troubleshooting

Suppose an application says:

> "Access denied."

Don't immediately reset the password.

Work through:

```text
1. Is MySQL running?
        ↓
2. Is port reachable?
        ↓
3. Is firewall allowing connection?
        ↓
4. Does account exist?
        ↓
5. Does User/Host match?
        ↓
6. Is authentication successful?
        ↓
7. Does the account have required privileges?
        ↓
8. Is TLS required?
        ↓
9. Is the application using the expected credentials?
```

Useful commands:

```sql
SELECT User, Host
FROM mysql.user;
```

and:

```sql
SHOW GRANTS FOR 'appuser'@'10.10.10.50';
```

---

# 35. Classic DBA Scenario

### Problem

Application:

```text
10.20.30.40
```

MySQL:

```text
10.20.30.50
```

The developer says:

> "I created `appuser`, but the application gets Access Denied."

You inspect:

```sql
SELECT User, Host
FROM mysql.user
WHERE User = 'appuser';
```

You discover:

```text
appuser | localhost
```

But the application connects remotely from:

```text
10.20.30.40
```

The account doesn't match the intended source.

This is why understanding:

```text
'user'@'host'
```

is critical.

---

# 36. Security Checklist

For a new MySQL server:

```text
[ ] Administrative credentials secured
[ ] Separate application accounts created
[ ] No application uses root
[ ] Least privilege applied
[ ] Unnecessary accounts removed/disabled
[ ] Host restrictions reviewed
[ ] Firewall configured
[ ] MySQL network exposure minimized
[ ] TLS requirements evaluated
[ ] Password/secret management established
[ ] Account locking policy reviewed
[ ] Privileges reviewed
[ ] Audit/logging requirements evaluated
[ ] Security changes documented
```

---

# 37. Lesson 3 Practical Lab

Create these three lab accounts:

### DBA test account

```text
dba_lab
```

### Application account

```text
orders_app
```

### Reporting account

```text
reporting_lab
```

Create a test database:

```sql
CREATE DATABASE dba_lab;
```

Create a table:

```sql
USE dba_lab;

CREATE TABLE employees
(
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department VARCHAR(100)
);
```

Insert sample data:

```sql
INSERT INTO employees
VALUES
(1, 'John', 'IT'),
(2, 'Sarah', 'Finance'),
(3, 'David', 'HR');
```

Now practice creating users and granting different privileges.

For example:

```text
orders_app
    ↓
SELECT/INSERT/UPDATE/DELETE

reporting_lab
    ↓
SELECT only
```

Then test each account.

---

# 38. Your Lesson 3 Challenge

You should be able to answer these without looking back:

1. What is authentication?
2. What is authorization?
3. What does `'user'@'host'` mean?
4. Why is `'appuser'@'localhost'` different from `'appuser'@'%'`?
5. What does `GRANT` do?
6. What does `REVOKE` do?
7. What does `SHOW GRANTS` do?
8. What is a role?
9. Why should applications not use `root`?
10. What is least privilege?
11. Why is a firewall still needed if MySQL has user privileges?
12. What is TLS protecting?
13. How do you lock a MySQL account?
14. How do you remove a MySQL account?
15. What should you investigate when you receive `Access denied`?
16. How would you give a reporting user read-only access?
17. How would you restrict an application account to a specific host?
18. Why shouldn't passwords be stored in application source code?

---

# 39. The DBA Security Model

Remember this:

```text
                 SECURITY
                    |
       +------------+------------+
       |            |            |
    Network    Authentication  Authorization
       |            |            |
    Firewall      Identity      Privileges
       |            |            |
       +------------+------------+
                    |
                   TLS
                    |
                  Audit
                    |
              Least Privilege
```

A secure database isn't achieved by one setting.

It's **layered security**.

---

## Lesson 3 Complete

You now have the foundations for:

* Creating accounts
* Managing privileges
* Using roles
* Restricting access
* Securing applications
* Troubleshooting authentication
* Applying least privilege

### Next: Lesson 4 — Backup, Restore & Disaster Recovery

This is one of the **most important lessons in the entire course**.

We'll cover:

* `mysqldump`
* MySQL Shell backup utilities
* Logical vs physical backups
* Full backups
* Incremental concepts
* Binary logs
* Point-in-time recovery
* Backup strategies
* Backup retention
* Backup validation
* Restore procedures
* Accidental `DROP TABLE`
* Accidental `DROP DATABASE`
* Server failure
* Corruption scenarios
* RPO/RTO
* Windows and Linux backup locations
* Production backup architecture
* **A complete backup-and-restore lab**
