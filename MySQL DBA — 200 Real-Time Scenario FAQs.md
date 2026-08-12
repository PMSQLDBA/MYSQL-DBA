Absolutely. Below is a **200-question real-time MySQL DBA scenario FAQ bank**, designed for **production DBA interviews, L2/L3 support, incident handling, and day-to-day operations**.

Each answer is kept to **5 points** so you can revise quickly while still covering the important DBA reasoning.

# MySQL DBA — 200 Real-Time Scenario FAQs

## 1–20: Installation & Configuration

### 1. MySQL service is not starting after installation. What will you check?

1. Check service status using `systemctl status mysql` on Linux or Services on Windows.
2. Check the MySQL error log for the exact startup failure.
3. Verify configuration syntax and recently changed parameters.
4. Check disk space, permissions, port conflicts, and data-directory availability.
5. Correct the root cause, start MySQL, and validate with `SELECT VERSION();`.

---

### 2. MySQL starts and immediately stops. What could be wrong?

1. Check the MySQL error log first.
2. Look for InnoDB corruption, permission problems, invalid configuration, or missing files.
3. Check whether another process is using the configured port.
4. Verify ownership/permissions of the MySQL data directory.
5. Never delete InnoDB files randomly; determine the exact failure first.

---

### 3. MySQL is consuming very high memory. How do you investigate?

1. Check OS memory usage using `free -h`, Task Manager, or Performance Monitor.
2. Review `innodb_buffer_pool_size`.
3. Check `max_connections` and per-connection memory consumers.
4. Investigate temporary tables, Performance Schema, sort buffers, and other memory allocations.
5. Reduce the actual source of pressure rather than blindly reducing the buffer pool.

---

### 4. How would you configure MySQL after installation?

1. Confirm CPU, RAM, storage, OS, and workload requirements.
2. Configure networking, data directory, logging, and InnoDB settings.
3. Configure authentication and least-privilege accounts.
4. Enable required binary logging/replication features.
5. Test the configuration and establish a performance baseline.

---

### 5. MySQL port 3306 is not reachable. What do you check?

1. Confirm MySQL is running.
2. Check `bind_address` and configured port.
3. Check whether MySQL is listening with `ss -lntp` or Windows networking tools.
4. Check OS firewall/security-group rules.
5. Test connectivity from the actual application server.

---

### 6. MySQL works locally but not remotely. Why?

1. MySQL may be bound only to localhost.
2. Firewall rules may block the port.
3. The MySQL account may not permit connections from the application host.
4. Network ACL/security-group rules may block traffic.
5. Validate both network connectivity and MySQL authentication.

---

### 7. How do you verify a MySQL installation?

1. Run `mysql --version`.
2. Connect using the MySQL client.
3. Run `SELECT VERSION();`.
4. Verify `datadir`, port, character set, and important variables.
5. Create and remove a test database to confirm read/write functionality.

---

### 8. What would you check before installing MySQL on a production server?

1. Supported OS and MySQL version.
2. CPU, RAM, storage, and filesystem capacity.
3. Network and security requirements.
4. Backup, monitoring, and HA requirements.
5. Application compatibility and organizational standards.

---

### 9. How do you identify MySQL's data directory?

1. Connect to MySQL.
2. Run `SHOW VARIABLES LIKE 'datadir';`.
3. Confirm the path at OS level.
4. Check filesystem capacity and permissions.
5. Avoid modifying files directly inside the data directory unless the procedure explicitly requires it.

---

### 10. How do you safely change MySQL configuration?

1. Identify the parameter and understand its effect.
2. Check whether it is dynamic or requires restart.
3. Test the change outside production first.
4. Back up/version-control the configuration.
5. Apply, validate, monitor, and document the change.

---

### 11. MySQL configuration contains an invalid parameter. What happens?

1. MySQL may fail to start.
2. The error log normally identifies the invalid option.
3. Review recent configuration changes.
4. Correct or remove the unsupported parameter.
5. Restart and validate all required services.

---

### 12. How do you determine which MySQL version is installed?

1. Use `mysql --version`.
2. Run `SELECT VERSION();`.
3. Check package information on Linux if required.
4. Check the Windows installation/service information where applicable.
5. Record the exact version for patching and upgrade planning.

---

### 13. MySQL is installed but the client cannot connect. What do you check?

1. Confirm the server is running.
2. Confirm host and port.
3. Confirm username/password.
4. Check account host permissions.
5. Review the MySQL error log and client error message.

---

### 14. How do you configure MySQL for a new application?

1. Create a dedicated database.
2. Create a dedicated application account.
3. Grant only required privileges.
4. Configure connection limits and TLS according to requirements.
5. Test application connectivity before production deployment.

---

### 15. Why shouldn't you copy a random `my.cnf` from the internet?

1. Settings depend on MySQL version.
2. Hardware and workload differ.
3. Some parameters may be obsolete.
4. Incorrect memory settings can cause instability.
5. Configuration should be evidence-driven and tested.

---

### 16. MySQL server has 128 GB RAM. Should you allocate 100 GB to Buffer Pool?

1. Not automatically.
2. Account for OS memory and other MySQL allocations.
3. Consider connection count and workload.
4. Consider monitoring, backup, and other processes.
5. Establish sizing based on workload and measured memory behavior.

---

### 17. MySQL hostname resolution is causing connection delays. What do you investigate?

1. Check DNS behavior.
2. Check client connection configuration.
3. Review MySQL account host definitions.
4. Investigate network latency and name-resolution failures.
5. Use appropriate hostname-resolution configuration based on your environment.

---

### 18. How do you safely restart MySQL in production?

1. Confirm the reason for restart.
2. Check application and HA impact.
3. Verify backup/recovery readiness.
4. Perform controlled shutdown/restart during an approved window.
5. Validate connections, queries, replication, and application health afterward.

---

### 19. MySQL restarted unexpectedly. How do you investigate?

1. Check MySQL error logs.
2. Check OS/system logs.
3. Determine whether the server itself rebooted.
4. Look for OOM killer, hardware, storage, or service-manager events.
5. Identify root cause before assuming MySQL caused the restart.

---

### 20. MySQL starts slowly after a crash. What do you do?

1. Check the error log for InnoDB recovery.
2. Determine whether crash recovery is progressing.
3. Monitor CPU, disk I/O, and logs.
4. Avoid repeatedly killing the process during recovery.
5. Allow recovery to complete unless evidence shows a separate failure.

---

# 21–40: Users, Security & Access

### 21. Application suddenly gets "Access denied". What do you check?

1. Verify username and password.
2. Check the account's host definition.
3. Run `SHOW GRANTS`.
4. Check whether privileges or authentication configuration changed.
5. Review recent deployments and security changes.

---

### 22. How do you create a secure application user?

1. Create a dedicated account.
2. Use a strong authentication method/password policy.
3. Restrict the allowed source host/network.
4. Grant only required privileges.
5. Store credentials securely and rotate them according to policy.

---

### 23. An application has `GRANT ALL ON *.*`. What is wrong?

1. It violates least privilege.
2. A compromised application could modify or destroy unrelated databases.
3. It may allow administrative operations.
4. Determine the application's actual required privileges.
5. Replace broad access with database/table-level permissions.

---

### 24. How do you investigate suspicious MySQL access?

1. Identify the account and source host.
2. Review available authentication/audit logs.
3. Check recent privilege changes.
4. Determine what activity occurred.
5. Rotate credentials and contain the account if compromise is suspected.

---

### 25. How do you check a user's privileges?

1. Identify the exact username and host.
2. Run `SHOW GRANTS FOR 'user'@'host';`.
3. Review direct privileges.
4. Review role-based privileges where applicable.
5. Remove unnecessary privileges.

---

### 26. A DBA accidentally dropped a database. What do you do?

1. Stop further destructive activity.
2. Determine exactly what was deleted and when.
3. Identify the latest valid backup.
4. Restore to a safe location and recover required changes.
5. Validate data before production recovery/cutover.

---

### 27. How should DBA administrative accounts be managed?

1. Use individual named accounts where organizational policy supports them.
2. Avoid shared credentials.
3. Use least privilege.
4. Protect privileged credentials and MFA where supported by the surrounding environment.
5. Audit privileged activity.

---

### 28. MySQL root password is lost. What is your approach?

1. Follow your organization's approved password-recovery procedure.
2. Protect production access during the recovery process.
3. Use the appropriate MySQL-version-specific recovery method.
4. Reset credentials securely.
5. Verify applications and other administrative accounts afterward.

---

### 29. Why should MySQL not be publicly exposed?

1. It increases attack surface.
2. Authentication attacks become easier.
3. Database vulnerabilities become directly reachable.
4. Sensitive data is placed at unnecessary risk.
5. Prefer private networking, firewall restrictions, and secure application access.

---

### 30. How do you secure MySQL backups?

1. Encrypt backups where required.
2. Restrict filesystem/object-storage access.
3. Protect backup credentials.
4. Store appropriate copies offsite.
5. Regularly test restoration and access controls.

---

### 31. What is TLS used for in MySQL?

1. It protects data in transit.
2. It protects credentials from network interception.
3. It can provide server authentication.
4. Client certificates can provide stronger mutual authentication where required.
5. Certificates and supported TLS settings must be maintained.

---

### 32. A certificate is about to expire. What should the DBA do?

1. Identify affected MySQL endpoints and clients.
2. Obtain/renew the certificate through the approved process.
3. Test the new certificate.
4. Deploy it using a controlled change.
5. Validate TLS connections before the old certificate expires.

---

### 33. How do you implement least privilege?

1. Identify exactly what the application needs.
2. Create purpose-specific accounts.
3. Grant only required permissions.
4. Restrict hosts where practical.
5. Review privileges periodically.

---

### 34. Why shouldn't monitoring use the root account?

1. Monitoring normally doesn't require full privileges.
2. Root credentials increase security risk.
3. A compromised monitoring system could compromise MySQL.
4. Create a dedicated monitoring account.
5. Grant only required read/instrumentation access.

---

### 35. What is SQL injection from a DBA perspective?

1. It is primarily an application security vulnerability.
2. Unsafe SQL allows attackers to manipulate queries.
3. Excessive database privileges increase potential damage.
4. Parameterized queries should be used by applications.
5. Least privilege limits the blast radius.

---

### 36. How do you investigate repeated failed logins?

1. Identify source addresses/accounts.
2. Review available authentication/audit logs.
3. Determine whether failures are application-related or suspicious.
4. Check for credential rotation/deployment issues.
5. Escalate security incidents according to organizational procedures.

---

### 37. A user requests `SUPER`/high-level privileges. What do you do?

1. Ask why the privilege is required.
2. Identify a narrower privilege if possible.
3. Check organizational security policy.
4. Approve only when justified.
5. Document and periodically review the elevated privilege.

---

### 38. How do you remove an unnecessary account?

1. Confirm the account is no longer used.
2. Check application/jobs/dependencies.
3. Disable or remove it according to policy.
4. Monitor for failed connections afterward.
5. Document the change.

---

### 39. Why should application credentials not be stored in source code?

1. Source repositories may be widely accessible.
2. Credentials can leak through logs or copies.
3. Rotation becomes difficult.
4. Use a secure secret-management mechanism.
5. Rotate credentials if exposure is suspected.

---

### 40. How do you perform a MySQL security review?

1. Review accounts and hosts.
2. Review privileges and roles.
3. Review network exposure.
4. Review TLS, authentication, logs, and patch level.
5. Document findings and remediation actions.

---

# 41–60: Backup & Restore

### 41. A scheduled backup failed overnight. What do you do?

1. Identify the exact backup failure.
2. Check storage, credentials, connectivity, and backup-tool logs.
3. Determine whether another valid backup exists.
4. Fix the failure and run a controlled backup.
5. Verify the new backup rather than simply trusting the job status.

---

### 42. How do you prove that backups are usable?

1. Restore the backup to a test environment.
2. Check database/table availability.
3. Validate representative data.
4. Test application connectivity where appropriate.
5. Record restore time and recovery results.

---

### 43. What is RPO?

1. Recovery Point Objective defines acceptable data loss.
2. It is measured in time.
3. Example: RPO of 5 minutes means losing more than 5 minutes of data is unacceptable.
4. Backup/replication design must support the required RPO.
5. RPO must come from business requirements.

---

### 44. What is RTO?

1. Recovery Time Objective defines acceptable downtime.
2. It is measured in time.
3. Example: RTO of 30 minutes means service should be restored within 30 minutes.
4. HA/DR architecture must support it.
5. Test actual recovery time rather than assuming it.

---

### 45. Backup is successful but restore fails. What does this mean?

1. The backup process did not provide usable recovery.
2. Investigate backup integrity and completeness.
3. Check compatibility and corruption.
4. Fix the backup process.
5. Establish recurring restore testing.

---

### 46. How do you recover accidentally deleted data?

1. Stop further destructive activity.
2. Determine the deletion time.
3. Restore the backup to a separate environment.
4. Apply available recovery information such as binary logs when appropriate.
5. Extract/restore the required data and validate it.

---

### 47. What is point-in-time recovery?

1. PITR restores data to a selected point in time.
2. It generally combines a base backup with subsequent binary-log events.
3. Identify the desired recovery timestamp/position.
4. Restore the base backup first.
5. Replay the required logs and validate the result.

---

### 48. Binary logs were deleted accidentally. What happens?

1. PITR coverage may be lost for the deleted period.
2. Replicas may also be affected if they need missing logs.
3. Determine what backups/replicas still contain the required data.
4. Restore from another valid source if possible.
5. Review log-retention and deletion procedures.

---

### 49. Why are backup retention policies important?

1. They determine how far back recovery is possible.
2. They affect storage requirements.
3. They support regulatory/business requirements.
4. They must align with RPO and recovery strategy.
5. Retention should be documented and periodically reviewed.

---

### 50. Where should production backups be stored?

1. Not exclusively on the production server.
2. Use appropriate separate/offsite storage.
3. Protect access to the backup repository.
4. Encrypt sensitive backups where required.
5. Regularly test recovery from the stored copies.

---

### 51. Backup storage is 95% full. What do you do?

1. Identify which backups consume the space.
2. Review retention policies.
3. Remove only backups eligible for deletion.
4. Increase storage if required.
5. Fix the capacity trend so it doesn't recur.

---

### 52. How do you calculate backup storage requirements?

1. Estimate database size.
2. Estimate growth rate.
3. Account for backup frequency.
4. Account for retention and duplicate/offsite copies.
5. Include operational headroom.

---

### 53. How do you perform a restore safely?

1. Confirm the backup and recovery objective.
2. Restore into an isolated/test location when possible.
3. Validate database consistency.
4. Validate required users, objects, and data.
5. Cut over only after successful validation.

---

### 54. What should be included in a backup runbook?

1. Backup method and schedule.
2. Storage location and retention.
3. Verification procedure.
4. Restore procedure.
5. Escalation and failure-handling process.

---

### 55. What is the difference between logical and physical backup?

1. Logical backup exports database objects/data.
2. Physical backup copies database storage structures/files.
3. Logical backups are generally more portable.
4. Physical approaches can be advantageous for large databases and faster recovery.
5. Selection depends on size, RTO, portability, and tooling.

---

### 56. A 2 TB database must be backed up daily. What do you consider?

1. Backup duration.
2. Storage capacity.
3. Network bandwidth.
4. Impact on production workload.
5. Recovery time and backup/restore strategy.

---

### 57. Why should you not run an untested restore directly over production?

1. You can destroy usable data.
2. Recovery mistakes may worsen the incident.
3. You need to validate backup contents first.
4. Restore to an isolated environment whenever practical.
5. Use a controlled cutover after validation.

---

### 58. A backup job takes twice as long as usual. What do you investigate?

1. Database growth.
2. Storage throughput.
3. Network performance.
4. Concurrent workload.
5. Backup-tool/system changes.

---

### 59. How often should restore testing be performed?

1. According to business and compliance requirements.
2. Critical systems should be tested regularly.
3. Test after significant backup architecture changes.
4. Measure actual RTO/RPO.
5. Document and remediate failures.

---

### 60. What is the DBA's most important backup rule?

1. Backup is only part of recovery.
2. Verification is necessary.
3. Restore testing is essential.
4. Offsite copies protect against infrastructure failure.
5. Recovery procedures must be documented and practiced.

---

# 61–80: Performance & Troubleshooting

### 61. Application reports that MySQL is slow. What do you do first?

1. Define the exact symptom and affected workload.
2. Check CPU, memory, disk I/O, and connections.
3. Identify slow/high-frequency SQL.
4. Check locks, waits, and execution plans.
5. Mitigate the actual bottleneck and then investigate root cause.

---

### 62. CPU suddenly reaches 100%. What do you check?

1. Identify which process consumes CPU.
2. Check active MySQL statements.
3. Look for expensive queries or query storms.
4. Check recent application/configuration changes.
5. Mitigate and optimize the offending workload.

---

### 63. Disk I/O is extremely high. What could cause it?

1. Large table scans.
2. Heavy writes.
3. Insufficient caching.
4. Backup or maintenance activity.
5. Temporary tables, logs, or other filesystem activity.

---

### 64. Disk space is rapidly increasing. What do you investigate?

1. Database/table growth.
2. Binary logs.
3. Backup files.
4. Error/slow/general logs.
5. Temporary or other OS/application files.

---

### 65. MySQL has too many connections. What do you check?

1. Current connection count.
2. `max_connections`.
3. Application connection pooling.
4. Idle/stuck connections.
5. Whether increasing the limit would create memory pressure.

---

### 66. Application gets "Too many connections." What is your immediate approach?

1. Confirm the connection count.
2. Identify connection sources.
3. Check whether a deployment caused a connection leak.
4. Restore healthy connection behavior.
5. Increase `max_connections` only after capacity analysis.

---

### 67. One query suddenly becomes very slow. What do you investigate?

1. Run `EXPLAIN`.
2. Compare the current plan with the expected plan.
3. Check indexes and statistics.
4. Check locking and data-volume changes.
5. Investigate application/query changes.

---

### 68. Query performs a full table scan. Is that always a problem?

1. No.
2. A full scan can be efficient for small tables.
3. It may be appropriate when a large percentage of rows is required.
4. Check execution time and actual workload.
5. Add indexes only when they improve the access pattern.

---

### 69. How do you identify expensive queries?

1. Use Performance Schema/sys schema where available.
2. Review slow query logging.
3. Look at execution time and frequency.
4. Check rows examined and workload impact.
5. Prioritize queries based on total business impact.

---

### 70. A query has an index but still runs slowly. Why?

1. The optimizer may choose another access path.
2. The index may have poor selectivity.
3. The query may return a large percentage of the table.
4. Additional sorting/join work may dominate.
5. Use execution-plan analysis rather than assuming "index = fast."

---

### 71. What is a covering index?

1. An index contains all columns needed for a query.
2. The engine may avoid additional table-row lookups.
3. This can reduce I/O.
4. It can improve certain read-heavy workloads.
5. But additional index storage/write cost must be considered.

---

### 72. Query uses `SELECT *`. Why can that be undesirable?

1. It returns unnecessary data.
2. It increases network traffic.
3. It can increase memory usage.
4. It may prevent efficient covering-index designs.
5. Explicitly selecting required columns is usually better.

---

### 73. A query became slow after data volume increased. Why?

1. The execution plan may no longer be optimal.
2. Index selectivity may have changed.
3. More rows may need to be processed.
4. Statistics/optimizer decisions may differ.
5. Reassess indexing and query design.

---

### 74. What is a slow query log?

1. It records queries exceeding configured criteria.
2. It helps identify expensive SQL.
3. It can show execution time and related metrics.
4. It is useful for performance investigations.
5. Avoid leaving high-volume logging enabled unnecessarily without capacity planning.

---

### 75. What is Performance Schema?

1. It provides MySQL instrumentation.
2. It helps analyze statements, waits, locks, and resources.
3. It is useful for troubleshooting.
4. It can provide detailed performance visibility.
5. Learn to use it rather than relying only on OS metrics.

---

### 76. What is the sys schema?

1. It provides DBA-friendly views.
2. It uses underlying performance instrumentation.
3. It simplifies common troubleshooting tasks.
4. It helps identify expensive statements and activity.
5. It is useful for operational analysis.

---

### 77. What does `SHOW PROCESSLIST` help with?

1. It shows active MySQL sessions.
2. It shows running/waiting statements.
3. It helps identify blocked sessions.
4. It helps identify long-running operations.
5. `SHOW FULL PROCESSLIST` provides more statement detail.

---

### 78. A query is blocked. How do you investigate?

1. Identify the waiting session.
2. Identify the blocking session.
3. Determine which transaction holds the lock.
4. Check how long it has been open.
5. Resolve the underlying transaction problem safely.

---

### 79. What is a deadlock?

1. Two or more transactions wait on each other.
2. Neither can proceed.
3. InnoDB detects many deadlocks automatically.
4. One transaction is normally rolled back.
5. Analyze access order and transaction design to prevent recurrence.

---

### 80. How do you reduce deadlocks?

1. Keep transactions short.
2. Access resources in consistent order.
3. Use appropriate indexes.
4. Avoid unnecessary locking.
5. Make applications retry appropriate deadlocked transactions.

---

# 81–100: InnoDB & Transactions

### 81. What is the InnoDB Buffer Pool?

1. It is a major memory area for InnoDB.
2. It caches data and index pages.
3. It reduces disk I/O.
4. Its sizing affects performance and memory usage.
5. It must be sized based on the whole server workload.

---

### 82. What is redo log used for?

1. It records information needed for recovery.
2. It helps preserve committed changes across crashes.
3. It supports InnoDB crash recovery.
4. It works with flushing/write ordering mechanisms.
5. Redo configuration must match workload and MySQL version.

---

### 83. What is undo data?

1. It supports transaction rollback.
2. It supports MVCC.
3. It helps provide previous row versions.
4. Long transactions can delay purge.
5. Excessive history can contribute to storage/transaction-management issues.

---

### 84. What is MVCC?

1. Multi-Version Concurrency Control.
2. It maintains multiple row versions.
3. It allows consistent reads.
4. It reduces reader/writer blocking in many cases.
5. It is fundamental to understanding InnoDB concurrency.

---

### 85. What is a long-running transaction?

1. A transaction remains open for an extended period.
2. It can retain row versions.
3. It may hold locks.
4. It can delay purge and affect replication/workload.
5. Find and fix the application behavior causing it.

---

### 86. Why are long transactions dangerous?

1. They consume transactional resources.
2. They can retain undo/history.
3. They may block other transactions.
4. They complicate backup/maintenance operations.
5. They can cause production performance problems.

---

### 87. What happens if a transaction is not committed?

1. Changes remain uncommitted.
2. Locks may remain held.
3. Other transactions may wait.
4. Undo/history may be retained.
5. The transaction should be committed or rolled back appropriately.

---

### 88. What is autocommit?

1. It controls automatic transaction commit behavior.
2. With autocommit enabled, individual statements generally commit automatically.
3. Explicit transactions can override this behavior.
4. Check with `SELECT @@autocommit;`.
5. Application transaction design should be understood before changing it.

---

### 89. What is isolation level?

1. It controls visibility/concurrency behavior between transactions.
2. MySQL supports multiple isolation levels.
3. Stronger isolation can reduce concurrency.
4. Weaker isolation may permit more anomalies.
5. Select it based on application consistency requirements.

---

### 90. What is a lock wait timeout?

1. A transaction waits for a lock.
2. The lock is not released within the configured timeout.
3. MySQL returns an error.
4. Investigate the blocking transaction.
5. Don't simply increase timeout without fixing the locking problem.

---

### 91. What is metadata locking?

1. MySQL protects object definitions with metadata locks.
2. DDL often requires metadata locks.
3. Long transactions can block DDL.
4. DDL can then create a queue of waiting sessions.
5. Check transaction state before production DDL.

---

### 92. `ALTER TABLE` appears hung. What could be happening?

1. It may be waiting for a metadata lock.
2. Another transaction may be holding the required lock.
3. The DDL may also genuinely be performing a large operation.
4. Check process/lock information.
5. Determine the exact state before killing anything.

---

### 93. How do you safely perform production DDL?

1. Understand the DDL operation.
2. Check whether it is instant/online/copying for your version and operation.
3. Check table size and workload.
4. Test in staging.
5. Monitor metadata locks and application impact.

---

### 94. Why can a transaction block an `ALTER TABLE`?

1. The transaction may hold a metadata lock.
2. DDL requires an appropriate metadata lock.
3. The DDL waits.
4. New queries can potentially become affected depending on lock queues.
5. Commit/rollback the blocking transaction appropriately.

---

### 95. What is a transaction deadlock retry?

1. InnoDB may roll back one transaction.
2. The application receives a deadlock error.
3. The application can retry the transaction where safe.
4. The transaction should be designed to be retryable.
5. Root cause should still be investigated.

---

### 96. Why should transactions avoid external API calls?

1. External calls may be slow.
2. Database locks remain open during the call.
3. Lock contention can increase.
4. Rollbacks become more complicated.
5. Separate external work from short database transactions when possible.

---

### 97. How do you identify long-running transactions?

1. Inspect transaction metadata.
2. Use Performance Schema/sys information where available.
3. Look for old transaction start times.
4. Correlate with blocking/undo/history behavior.
5. Identify the application/session responsible.

---

### 98. What happens during an InnoDB crash recovery?

1. MySQL starts InnoDB recovery.
2. Recovery uses transactional log information.
3. Required changes are reapplied/processed.
4. Incomplete transactions are handled appropriately.
5. The database becomes available after recovery completes.

---

### 99. Should you kill MySQL during crash recovery?

1. Generally, no.
2. Recovery may be doing necessary work.
3. Repeated interruption can prolong recovery.
4. Monitor logs and disk activity first.
5. Act only when evidence shows recovery itself is stuck or there is another serious failure.

---

### 100. How do you investigate InnoDB problems?

1. Check the MySQL error log.
2. Run `SHOW ENGINE INNODB STATUS\G`.
3. Inspect Performance Schema where relevant.
4. Review locks, transactions, I/O, and buffer-pool behavior.
5. Correlate database evidence with OS metrics.

---

# 101–120: Replication & HA

### 101. Replication suddenly stopped. What do you check?

1. Run `SHOW REPLICA STATUS\G`.
2. Check IO and SQL/apply error information.
3. Check network connectivity.
4. Check disk space and permissions.
5. Determine whether the problem is source, transport, or apply related.

---

### 102. Replica lag is increasing. What do you investigate?

1. Determine whether source generation increased.
2. Check replica CPU, disk I/O, and workload.
3. Check the replication apply state.
4. Look for long-running transactions or problematic queries.
5. Fix the bottleneck instead of blindly ignoring lag.

---

### 103. Replica is seconds behind. Is that always a problem?

1. Not necessarily.
2. Some asynchronous workloads naturally have small lag.
3. Compare lag against the application's RPO requirement.
4. Check whether lag is stable or continuously increasing.
5. Alert thresholds should reflect business requirements.

---

### 104. Replica lag suddenly jumps from seconds to hours. What do you do?

1. Check replication errors.
2. Check source and replica workload.
3. Check disk/CPU/I/O saturation.
4. Identify large transactions or DDL.
5. Determine whether replication can catch up or requires intervention.

---

### 105. What is GTID?

1. Global Transaction Identifier.
2. It identifies transactions across replication.
3. It simplifies transaction tracking.
4. It can simplify failover and replica positioning.
5. Understand GTID behavior for your specific MySQL version/topology.

---

### 106. Why is GTID useful for failover?

1. Transactions can be identified consistently.
2. Replica positioning becomes easier.
3. It simplifies topology changes.
4. It reduces manual binary-log position management.
5. It supports cleaner operational procedures.

---

### 107. Replica has a SQL error. Should you skip it?

1. Not automatically.
2. Skipping can create data inconsistency.
3. First determine why the error occurred.
4. Compare source and replica state.
5. Use skipping only through an approved recovery procedure after understanding the impact.

---

### 108. What causes replication errors?

1. Data inconsistencies.
2. Unsupported/changed SQL behavior.
3. DDL differences.
4. Permission/configuration problems.
5. Storage/network/system failures.

---

### 109. How do you validate a replica before failover?

1. Confirm replication is healthy.
2. Check lag.
3. Confirm no unresolved errors.
4. Validate required databases and application state.
5. Confirm the candidate meets the failover criteria.

---

### 110. What is failover?

1. Moving service from a failed/unavailable primary to another database node.
2. The candidate must have sufficiently current data.
3. Old primary may need isolation/fencing.
4. Applications must be redirected.
5. Post-failover topology must be repaired.

---

### 111. What is switchover?

1. A planned role change.
2. The current primary is intentionally changed to another node.
3. Replication is coordinated before the change.
4. Traffic is redirected.
5. It should be tested regularly.

---

### 112. What is split brain?

1. Two nodes believe they are primary.
2. Both may accept writes.
3. Data divergence can occur.
4. It is a severe HA failure.
5. Proper fencing/coordination is essential.

---

### 113. Why is fencing important?

1. It prevents an old primary from continuing writes.
2. It reduces split-brain risk.
3. It can involve network or infrastructure controls.
4. It must be part of failover design.
5. Promotion without isolation can create data divergence.

---

### 114. How do you recover an old primary after failover?

1. Do not immediately put it back into production.
2. Determine its data state.
3. Rebuild/reseed or rejoin it according to topology.
4. Validate replication.
5. Add it back only after consistency is confirmed.

---

### 115. Application cannot connect after failover. What do you check?

1. DNS/endpoint/router configuration.
2. Firewall/network connectivity.
3. MySQL account permissions.
4. TLS/certificate configuration.
5. Application connection-pool behavior.

---

### 116. Replica is running out of disk. What do you check?

1. Database growth.
2. Relay logs.
3. Binary logs.
4. Temporary files.
5. Backup or other filesystem usage.

---

### 117. Replication is healthy but application reads stale data. Why?

1. Application may be reading from an asynchronous replica.
2. Replication lag may exist.
3. Read-after-write consistency may not be guaranteed.
4. Application routing may need adjustment.
5. Choose primary reads or consistency controls when business logic requires them.

---

### 118. How do you monitor replication?

1. Monitor replication state.
2. Monitor lag.
3. Monitor errors.
4. Monitor disk/CPU/I/O.
5. Alert on conditions that threaten RPO or HA.

---

### 119. What should an HA runbook contain?

1. Detection criteria.
2. Promotion procedure.
3. Fencing procedure.
4. Application redirection.
5. Validation and recovery of the old primary.

---

### 120. How often should failover be tested?

1. According to business requirements.
2. Critical systems should be exercised regularly.
3. Test both planned and unplanned scenarios.
4. Measure actual recovery time.
5. Fix issues discovered during drills.

---

# 121–140: Patching & Upgrades

### 121. Production MySQL needs a security patch. What is your approach?

1. Confirm affected version and vulnerability.
2. Determine the supported target release.
3. Test the patch in non-production.
4. Back up and plan rollback/HA procedure.
5. Patch, validate, and monitor production.

---

### 122. How do you patch a replicated environment?

1. Confirm topology and health.
2. Patch a replica first where operationally appropriate.
3. Validate replication and application compatibility.
4. Perform controlled switchover if required.
5. Patch remaining nodes and restore redundancy.

---

### 123. Why patch a replica first?

1. It reduces production risk.
2. It provides a realistic validation environment.
3. It helps identify compatibility issues.
4. It can support rolling maintenance.
5. The exact approach depends on topology and MySQL version.

---

### 124. What is the difference between patching and upgrading?

1. Patching generally updates within a release/support path.
2. Upgrading can involve a new major/minor release with compatibility changes.
3. Upgrade risk is generally broader.
4. Both require testing.
5. Always follow the supported path for the installed version.

---

### 125. What do you check before a MySQL upgrade?

1. Supported upgrade path.
2. Release notes.
3. Deprecated/removed features.
4. Application and connector compatibility.
5. Backup, restore, rollback, and HA plan.

---

### 126. Upgrade succeeded but application fails. What do you investigate?

1. Check MySQL error log.
2. Check application errors.
3. Check authentication/plugin changes.
4. Check SQL behavior and driver compatibility.
5. Compare pre-upgrade and post-upgrade behavior.

---

### 127. How do you validate a MySQL upgrade?

1. Check exact version.
2. Check service health.
3. Check databases/tables.
4. Check application connectivity.
5. Check replication, performance, errors, and backups.

---

### 128. Why should you never test an upgrade only on the DBA's machine?

1. Application compatibility is environment-specific.
2. Production data volume differs.
3. Configuration differs.
4. Replication/HA behavior must be tested.
5. Realistic staging is much safer.

---

### 129. What is rollback planning for an upgrade?

1. Determine whether downgrade is supported.
2. Identify restore/rebuild options.
3. Preserve a valid backup.
4. Define application cutback procedures.
5. Test the recovery approach where possible.

---

### 130. Can every MySQL upgrade be rolled back by installing the old version?

1. No.
2. Data-format/behavior changes may prevent a simple downgrade.
3. Some upgrades require logical or backup-based recovery.
4. The supported upgrade documentation must be followed.
5. Rollback should be designed before the upgrade.

---

### 131. What is an upgrade compatibility test?

1. Test database startup.
2. Test application queries.
3. Test authentication.
4. Test stored objects and replication.
5. Test performance and operational tooling.

---

### 132. MySQL upgrade causes higher CPU. What do you investigate?

1. Query execution plans.
2. Optimizer changes.
3. New workload behavior.
4. Configuration/default changes.
5. Compare before/after performance data.

---

### 133. Upgrade caused replication failure. What do you do?

1. Stop further topology changes.
2. Check replication errors.
3. Verify source/replica version compatibility.
4. Review changed configuration or SQL behavior.
5. Restore replication through a controlled procedure.

---

### 134. Why should release notes be mandatory before upgrading?

1. They identify important behavior changes.
2. They identify security fixes.
3. They identify deprecated/removed features.
4. They identify upgrade requirements.
5. They help build the upgrade test plan.

---

### 135. What is a patch baseline?

1. The approved version for an environment.
2. It ensures servers remain within supported security levels.
3. It simplifies compliance.
4. It helps identify outdated systems.
5. Maintain an inventory against the baseline.

---

### 136. How do you handle an emergency security patch?

1. Assess severity and exposure.
2. Confirm the correct vendor release.
3. Perform rapid validation.
4. Use HA/backup protections.
5. Patch and monitor with documented emergency-change procedures.

---

### 137. What should you document after patching?

1. Old version.
2. New version.
3. Date/time.
4. Change performed.
5. Validation and any issues encountered.

---

### 138. Why is version standardization important?

1. It simplifies support.
2. It reduces compatibility differences.
3. It simplifies patching.
4. It improves troubleshooting.
5. It makes automation easier.

---

### 139. How do you handle multiple MySQL versions across environments?

1. Maintain a version inventory.
2. Define supported versions.
3. Plan standardization.
4. Test upgrades systematically.
5. Track unsupported/end-of-life installations.

---

### 140. What is the biggest mistake during MySQL upgrades?

1. Treating the upgrade as only a software installation.
2. Ignoring application compatibility.
3. Not testing restore/rollback.
4. Ignoring replication/HA behavior.
5. Failing to validate after the upgrade.

---

# 141–160: Migration

### 141. How do you migrate MySQL from Windows to Linux?

1. Assess source/target versions and compatibility.
2. Build and secure the Linux target.
3. Choose logical/physical/replication-based migration.
4. Test the migration and validate data.
5. Perform controlled cutover and keep rollback available.

---

### 142. How do you migrate Linux to Windows?

1. Assess OS/version/application compatibility.
2. Build the Windows MySQL target.
3. Select a suitable migration mechanism.
4. Validate users, schemas, data, and application connectivity.
5. Perform cutover with a rollback plan.

---

### 143. What do you check before cross-platform migration?

1. MySQL versions.
2. Character sets/collations.
3. Database objects.
4. Filesystem/path differences.
5. Application/driver compatibility.

---

### 144. What is logical migration?

1. Data is exported as logical SQL/data representation.
2. Target reconstructs database objects/data.
3. It is generally portable across OS platforms.
4. It can take longer for very large databases.
5. It is useful when physical file portability isn't appropriate.

---

### 145. What is physical migration?

1. Database storage files are transferred/reused according to supported procedures.
2. It can be faster for large databases.
3. It is more dependent on version/storage compatibility.
4. It requires careful shutdown/backup/tooling procedures.
5. Never copy raw database files casually.

---

### 146. Migration completed but row counts differ. What do you do?

1. Identify affected tables.
2. Compare source and target counts.
3. Check excluded/system tables.
4. Compare checksums/data samples where appropriate.
5. Correct migration errors before cutover.

---

### 147. Migration completed but application fails. What do you check?

1. Connection endpoint.
2. User/password.
3. Grants.
4. SQL/application compatibility.
5. Character-set/TLS/driver differences.

---

### 148. How do you validate migrated data?

1. Compare database/table inventory.
2. Compare row counts.
3. Validate important business tables.
4. Compare checksums/samples where appropriate.
5. Run application-level validation.

---

### 149. What is a migration freeze?

1. A period where source writes are restricted or stopped.
2. It allows final synchronization.
3. It reduces source/target divergence.
4. It is used during controlled cutover.
5. Duration should be minimized and planned.

---

### 150. How do you reduce migration downtime?

1. Preload most data before cutover.
2. Use replication or incremental synchronization where appropriate.
3. Test the process beforehand.
4. Minimize final write-freeze duration.
5. Automate validation and cutover steps.

---

### 151. How do you migrate a 10 TB database?

1. Avoid assuming a simple dump/restore is sufficient.
2. Evaluate physical backup, replication, parallel migration, and other methods.
3. Measure transfer and restore throughput.
4. Design around RTO/RPO.
5. Perform multiple rehearsals before production.

---

### 152. Source and target MySQL versions differ. What do you check?

1. Supported migration/upgrade path.
2. SQL compatibility.
3. Authentication differences.
4. Character sets/collations.
5. Data types and deprecated features.

---

### 153. What happens if you migrate without testing?

1. Data problems may appear during production cutover.
2. Application incompatibilities may surface late.
3. Downtime can increase dramatically.
4. Rollback becomes harder.
5. A rehearsal significantly reduces risk.

---

### 154. How do you migrate users and privileges?

1. Inventory source accounts.
2. Identify application/service accounts.
3. Recreate required accounts using supported mechanisms.
4. Validate grants/roles.
5. Avoid copying unnecessary privileged accounts.

---

### 155. What is cutover?

1. The point at which production traffic moves to the target.
2. It follows final synchronization.
3. Application endpoints/DNS/router are changed.
4. Target functionality is validated.
5. Rollback remains available until acceptance.

---

### 156. What do you do if migration validation fails during cutover?

1. Stop further changes.
2. Determine severity and data impact.
3. Use the preplanned rollback/cutback procedure.
4. Preserve source availability.
5. Fix and rehearse the issue before another attempt.

---

### 157. Why should the source remain available after migration?

1. It provides rollback protection.
2. It allows comparison.
3. It helps recover unexpected data issues.
4. It provides operational safety during the acceptance period.
5. Decommission only after formal approval.

---

### 158. How do you migrate without losing data?

1. Establish a synchronization mechanism.
2. Minimize the final write window.
3. Verify source/target consistency.
4. Freeze writes during final cutover where required.
5. Validate after cutover.

---

### 159. What should a migration rollback plan contain?

1. Source remains recoverable.
2. Application can be redirected back.
3. Data divergence is understood.
4. DNS/connection routing can be reverted.
5. Decision criteria and owners are defined.

---

### 160. What is the most important migration principle?

1. Test before production.
2. Validate data, not just successful commands.
3. Minimize downtime.
4. Maintain rollback capability.
5. Never decommission the source immediately.

---

# 161–180: Linux, Windows & Production Operations

### 161. Linux shows `/var/lib/mysql` is 95% full. What do you do?

1. Identify which files/directories consume space.
2. Check database growth and binary logs.
3. Check backups and logs.
4. Clean only files covered by a safe retention process.
5. Increase capacity if the growth trend requires it.

---

### 162. Windows C: drive is nearly full. MySQL is installed there. What do you check?

1. Check MySQL data/log locations.
2. Identify the largest directories.
3. Check Windows event/log files.
4. Check backups and temporary files.
5. Free/expand capacity safely without deleting active database files.

---

### 163. MySQL Linux process is killed by OOM. What do you investigate?

1. Check kernel/system logs.
2. Check MySQL memory configuration.
3. Review connection count.
4. Check other processes consuming memory.
5. Correct total memory pressure rather than only restarting MySQL.

---

### 164. Windows MySQL service won't start after reboot. What do you check?

1. Check Windows Services.
2. Check MySQL error log.
3. Check Event Viewer.
4. Check configuration/data-directory permissions.
5. Check dependencies, port conflicts, and startup configuration.

---

### 165. How do you check Linux MySQL service health?

1. `systemctl status mysql`.
2. Check the error log.
3. Test a local MySQL connection.
4. Check listening ports.
5. Check OS resource health.

---

### 166. How do you check Windows MySQL service health?

1. Check Services/PowerShell.
2. Check MySQL error logs.
3. Check Event Viewer.
4. Test a local database connection.
5. Check CPU, memory, disk, and network.

---

### 167. Linux disk latency is high but CPU is low. What does that suggest?

1. Storage may be the bottleneck.
2. Check IOPS/throughput and filesystem behavior.
3. Investigate database scans/writes.
4. Check backup or maintenance workloads.
5. Coordinate with infrastructure/storage teams if necessary.

---

### 168. MySQL process is using high CPU but no obvious query is running. What do you investigate?

1. Check all sessions with full process information.
2. Inspect Performance Schema.
3. Look for background InnoDB activity.
4. Check replication and purge activity.
5. Correlate with OS monitoring and logs.

---

### 169. MySQL log files are consuming too much disk. What do you do?

1. Identify which log is growing.
2. Review configured retention/rotation.
3. Compress/archive according to policy.
4. Remove only files safe to remove.
5. Fix log configuration so recurrence is controlled.

---

### 170. Why is time synchronization important for DBA operations?

1. Logs need accurate timestamps.
2. Incident correlation depends on consistent time.
3. Replication/monitoring systems may rely on timestamps.
4. Certificates and security mechanisms depend on correct time.
5. Ensure OS time synchronization is healthy.

---

### 171. What OS metrics should a DBA monitor?

1. CPU.
2. Memory.
3. Disk capacity.
4. Disk latency/IOPS.
5. Network utilization.

---

### 172. What MySQL metrics should a DBA monitor?

1. Connections.
2. Query latency/throughput.
3. Buffer-pool activity.
4. Locks/waits.
5. Replication and error status.

---

### 173. MySQL is healthy but application is slow. Could MySQL still be the cause?

1. Yes, a specific query may be slow.
2. Lock waits may affect only part of the workload.
3. Connection pools may interact badly with MySQL.
4. A database endpoint may have increased latency.
5. Correlate application traces with database metrics.

---

### 174. Application is slow but database CPU is only 20%. What does that tell you?

1. CPU isn't necessarily the bottleneck.
2. Storage latency may be high.
3. Queries may be waiting on locks.
4. Network/application delays may exist.
5. Investigate waits rather than CPU alone.

---

### 175. What is the difference between capacity and performance?

1. Capacity concerns available resources.
2. Performance concerns response time/throughput.
3. A server can have spare CPU but poor latency.
4. A server can have enough disk but slow storage.
5. Monitor both dimensions.

---

### 176. What is change management?

1. A controlled process for production modifications.
2. It includes risk and impact assessment.
3. Testing and approvals are performed.
4. Implementation and validation are documented.
5. Rollback/mitigation is defined.

---

### 177. Why should DBA configuration be version-controlled?

1. Changes become traceable.
2. Previous versions can be reviewed.
3. Rollback is easier.
4. Peer review becomes possible.
5. Configuration drift is reduced.

---

### 178. Why is documentation important for DBAs?

1. Incidents often happen outside normal working hours.
2. Other DBAs need repeatable procedures.
3. DR depends on documented steps.
4. Audits require evidence.
5. Documentation reduces dependence on individual knowledge.

---

### 179. What should be included in a DBA runbook?

1. Normal operating procedures.
2. Backup/restore steps.
3. Troubleshooting procedures.
4. Failover/DR procedures.
5. Escalation contacts and validation steps.

---

### 180. What is configuration drift?

1. Servers that should be identical become different.
2. Manual changes commonly cause it.
3. Drift makes troubleshooting difficult.
4. Configuration management reduces it.
5. Regular audits should identify differences.

---

# 181–200: Senior DBA / Production Scenarios

### 181. Production database is suddenly completely unavailable. What is your first priority?

1. Confirm the outage and business impact.
2. Determine whether MySQL, OS, network, or infrastructure failed.
3. Initiate the appropriate HA/failover procedure.
4. Restore service safely.
5. Perform detailed root-cause analysis after stabilization.

---

### 182. You discover the primary server has failed. What do you do?

1. Confirm the failure rather than assuming it.
2. Prevent the failed primary from accepting writes if necessary.
3. Assess replica health and lag.
4. Promote the appropriate candidate through the approved procedure.
5. Redirect application traffic and validate.

---

### 183. Primary is down but replicas are also behind. Which replica do you promote?

1. Compare replication position/GTID state.
2. Determine which candidate has the most complete data.
3. Consider business RPO.
4. Select the healthiest viable replica.
5. Document any potential data loss.

---

### 184. Management asks, "Can we guarantee zero data loss?" What do you say?

1. It depends on the architecture.
2. Asynchronous replication can potentially lose recent transactions.
3. Synchronous/stronger consistency architectures have different trade-offs.
4. Backups alone don't guarantee zero loss.
5. Define realistic RPO based on architecture and business requirements.

---

### 185. Management asks, "Can we guarantee zero downtime?" What do you say?

1. Absolute zero downtime is difficult to guarantee.
2. HA can greatly reduce downtime.
3. Planned maintenance can often be performed with minimal interruption.
4. Failover itself has detection and transition time.
5. Define measurable availability objectives rather than promising the impossible.

---

### 186. Application deployment caused database performance degradation. What do you do?

1. Correlate the incident with deployment timing.
2. Identify changed SQL/workload.
3. Compare execution plans.
4. Roll back or mitigate the deployment if necessary.
5. Fix and test the application before redeployment.

---

### 187. A developer asks to disable foreign keys to improve performance. What do you do?

1. Understand the exact performance problem.
2. Determine whether constraints are actually the bottleneck.
3. Measure before changing behavior.
4. Consider indexing/query optimization instead.
5. Never disable integrity controls blindly in production.

---

### 188. Developer asks for a production `DROP TABLE`. What do you do?

1. Verify the business requirement.
2. Confirm backups/recovery capability.
3. Check dependencies.
4. Follow change-management approval.
5. Execute only with validated rollback/recovery procedures.

---

### 189. A developer says, "Just restart MySQL." How do you respond?

1. Ask what problem the restart is intended to solve.
2. Gather evidence first.
3. Determine application/HA impact.
4. Restart only if justified and controlled.
5. Validate after restart and identify the root cause.

---

### 190. A DBA says, "The database is slow because CPU is high." Is that enough?

1. No.
2. CPU could be a symptom rather than the root cause.
3. Identify which queries/processes consume CPU.
4. Check I/O, locks, memory, and workload changes.
5. Use evidence before changing configuration.

---

### 191. A production table needs a new index immediately. What do you consider?

1. Table size.
2. Current workload.
3. DDL method and locking behavior.
4. Disk space and write impact.
5. Test and schedule the change safely.

---

### 192. A 500 GB table needs an urgent schema change. What is your approach?

1. Understand the exact DDL operation.
2. Determine whether the operation can be performed online/instant for the installed version.
3. Test against a representative dataset.
4. Assess metadata-lock risk and disk requirements.
5. Use an appropriate online schema-change strategy if necessary.

---

### 193. Database is growing 100 GB every month. What should you do?

1. Establish the source of growth.
2. Forecast future capacity.
3. Review data-retention requirements.
4. Consider archival/purging strategies.
5. Plan storage and backup capacity ahead of time.

---

### 194. You inherit a MySQL server with no documentation. What do you do?

1. Inventory servers, versions, databases, users, and configuration.
2. Identify backups and replication.
3. Establish monitoring.
4. Document dependencies and recovery procedures.
5. Prioritize critical risks before optimization.

---

### 195. You inherit a server with no tested backups. What is your priority?

1. Determine whether any backups actually exist.
2. Verify backup freshness.
3. Perform a controlled restore test.
4. Establish a proper backup/recovery process.
5. Document RPO/RTO and recovery procedures.

---

### 196. You inherit a server with an unknown replication topology. What do you do?

1. Identify all MySQL nodes.
2. Inspect replication configuration/status.
3. Determine primary/replica relationships.
4. Document GTID/binlog state and dependencies.
5. Establish monitoring and failover documentation.

---

### 197. How would you prepare for a DBA audit?

1. Maintain server/version inventory.
2. Maintain access and privilege records.
3. Maintain backup/restore evidence.
4. Maintain patch/change records.
5. Maintain security, monitoring, HA, and DR documentation.

---

### 198. What distinguishes an L1, L2, and L3 MySQL DBA?

1. **L1:** monitoring, basic checks, standard procedures.
2. **L2:** troubleshooting, backup/restore, performance, replication.
3. **L3:** architecture, complex incidents, HA/DR, upgrades, migrations.
4. Senior DBAs also handle root-cause analysis and automation.
5. The levels vary by organization, but depth and ownership increase.

---

### 199. What should a senior MySQL DBA automate?

1. Installation and baseline configuration.
2. Backup verification.
3. Health checks and monitoring.
4. Patch/version inventory.
5. Standard operational tasks and reporting.

---

### 200. You are the only DBA on call and receive a critical production alert. What is your complete approach?

1. **Stabilize:** confirm impact, protect data, and restore service using the safest available mitigation.
2. **Investigate:** collect MySQL, OS, application, replication, and log evidence.
3. **Recover:** use HA, backup, restore, or failover according to the incident.
4. **Validate:** confirm database consistency, application functionality, replication, backups, and monitoring.
5. **Prevent:** document the root cause, corrective actions, automation opportunities, and lessons learned.

---

# 🎯 How to Use These 200 Scenarios

Don't memorize all 1,000+ individual points mechanically.

For **every production scenario**, train yourself to think in this sequence:

```text
             INCIDENT
                ↓
        1. CONFIRM SYMPTOM
                ↓
        2. CHECK SCOPE/IMPACT
                ↓
        3. COLLECT EVIDENCE
                ↓
        4. IDENTIFY ROOT CAUSE
                ↓
        5. MITIGATE SAFELY
                ↓
        6. VALIDATE RECOVERY
                ↓
        7. DOCUMENT/PREVENT
```

And remember these **10 DBA questions** whenever you're troubleshooting:

```text
1. What exactly is failing?
2. When did it start?
3. What changed?
4. Is MySQL running?
5. Is the OS healthy?
6. Is the database reachable?
7. Are queries/transactions blocked?
8. Is replication healthy?
9. Do we have a safe backup/recovery path?
10. What evidence proves the root cause?
```

If you can answer these **200 scenarios confidently and perform the corresponding labs**, you'll have a strong foundation for **MySQL DBA L1/L2/L3 interviews and real production operations**.
