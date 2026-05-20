# MongoDB CLI Reference: Comprehensive Guide for Tech Support Operations

## 1. Introduction

This document serves as a comprehensive, production-focused CLI reference for MongoDB operations. It is designed specifically for tech support engineers, database administrators, and DevOps professionals who need to manage, troubleshoot, and recover MongoDB deployments under pressure. The tools covered in this guide include `mongosh`, `mongodump`, `mongorestore`, `mongoexport`, and `mongoimport`. 

In high-stakes environments, having the right command at your fingertips can mean the difference between a minor hiccup and a catastrophic outage. This guide goes beyond basic usage, providing advanced one-liners, worst-case scenario recovery tactics, and performance optimization techniques.

## 2. The MongoDB Shell (`mongosh`)

The MongoDB Shell (`mongosh`) is a fully functional JavaScript and Node.js environment for interacting with MongoDB deployments. It replaces the legacy `mongo` shell and offers improved syntax highlighting, auto-completion, and error reporting.

### 2.1 Connection and Authentication

Connecting to a MongoDB instance securely is the first step in any operation. In production, you will almost always deal with authentication, TLS/SSL, and replica sets.

**Basic Connection:**
```bash
mongosh "mongodb://localhost:27017"
```

**Connecting to a Replica Set with Authentication:**
```bash
mongosh "mongodb://user:password@node1:27017,node2:27017,node3:27017/admin?replicaSet=myReplicaSet"
```

**Connecting with TLS/SSL and Certificate Validation:**
```bash
mongosh "mongodb://db.example.com:27017" --tls --tlsCAFile /etc/ssl/ca.pem --tlsCertificateKeyFile /etc/ssl/client.pem
```

### 2.2 Advanced Querying and Aggregation

Tech support often requires extracting specific data points or summarizing large datasets to identify anomalies.

**Find Queries with Projections and Sorting:**
```javascript
// Find all active users, return only their email and last login, sorted by last login descending
db.users.find({ status: "active" }, { email: 1, lastLogin: 1, _id: 0 }).sort({ lastLogin: -1 }).limit(100)
```

**Complex Aggregation Pipeline for Performance Analysis:**
```javascript
// Identify the most frequently accessed collections by analyzing the logs (assuming a custom log collection)
db.systemLogs.aggregate([
  { $match: { severity: "ERROR", timestamp: { $gte: new ISODate("2023-01-01T00:00:00Z") } } },
  { $group: { _id: "$component", errorCount: { $sum: 1 } } },
  { $sort: { errorCount: -1 } },
  { $limit: 5 }
])
```

### 2.3 Index Management and Optimization

Missing or inefficient indexes are the primary cause of performance degradation in MongoDB.

**Identifying Missing Indexes:**
```javascript
// View the query planner output to see if a query uses an index (IXSCAN) or a collection scan (COLLSCAN)
db.orders.find({ customerId: "CUST-123", status: "shipped" }).explain("executionStats")
```

**Creating Indexes in the Background:**
In production, always create indexes in the background to avoid locking the database.
```javascript
db.orders.createIndex({ customerId: 1, status: 1 }, { background: true })
```

**Finding and Dropping Unused Indexes:**
```javascript
// View index usage statistics
db.orders.aggregate([ { $indexStats: { } } ])

// Drop an index that is no longer needed
db.orders.dropIndex("customerId_1_status_1")
```

### 2.4 Replica Set and Cluster Management

Managing the health of a replica set is crucial for high availability.

**Checking Replica Set Status:**
```javascript
rs.status()
```

**Forcing a Node to Step Down:**
If the primary node is experiencing hardware issues, you may need to force an election.
```javascript
rs.stepDown(120) // Steps down for 120 seconds
```

**Reconfiguring a Replica Set (Emergency):**
If a majority of nodes are lost, you may need to force a reconfiguration.
```javascript
let cfg = rs.conf();
cfg.members = [cfg.members[0]]; // Keep only the surviving node
rs.reconfig(cfg, { force: true });
```

## 3. Data Backup with `mongodump`

`mongodump` is a utility for creating binary exports of the contents of a database. It is essential for disaster recovery and point-in-time backups.

### 3.1 Basic and Advanced Backups

**Full Database Backup:**
```bash
mongodump --uri="mongodb://user:password@localhost:27017/admin" --out=/backups/full_backup_$(date +%F)
```

**Backing Up a Specific Collection with a Query:**
Useful for extracting a subset of data for debugging without copying the entire collection.
```bash
mongodump --uri="mongodb://localhost:27017/mydb" --collection=audit_logs --query='{ "timestamp": { "$gte": { "$date": "2023-10-01T00:00:00Z" } } }' --out=/backups/audit_logs
```

**Archiving and Compressing on the Fly:**
Saves disk space and reduces I/O overhead.
```bash
mongodump --uri="mongodb://localhost:27017/mydb" --archive=/backups/mydb_$(date +%F).archive.gz --gzip
```

### 3.2 Oplog Backups for Point-in-Time Recovery

To achieve point-in-time recovery, you must include the oplog in your backup.

```bash
mongodump --uri="mongodb://localhost:27017/admin" --oplog --out=/backups/oplog_backup
```

## 4. Data Restoration with `mongorestore`

`mongorestore` reads the binary files produced by `mongodump` and restores them to a MongoDB instance.

### 4.1 Standard Restoration

**Restoring a Full Backup:**
```bash
mongorestore --uri="mongodb://localhost:27017/admin" /backups/full_backup_2023-10-25
```

**Restoring from a Compressed Archive:**
```bash
mongorestore --uri="mongodb://localhost:27017/mydb" --archive=/backups/mydb_2023-10-25.archive.gz --gzip
```

### 4.2 Advanced Restoration Techniques

**Restoring to a Different Database:**
Useful for creating staging environments from production backups.
```bash
mongorestore --uri="mongodb://localhost:27017/staging_db" --nsFrom="mydb.*" --nsTo="staging_db.*" /backups/mydb_backup
```

**Point-in-Time Recovery using the Oplog:**
Restores the database and replays the oplog up to a specific timestamp.
```bash
mongorestore --uri="mongodb://localhost:27017/admin" --oplogReplay --oplogLimit="1698278400" /backups/oplog_backup
```

**Dropping Collections Before Restore:**
Ensures a clean slate by dropping existing collections before restoring.
```bash
mongorestore --uri="mongodb://localhost:27017/mydb" --drop /backups/mydb_backup
```

## 5. Data Export with `mongoexport`

`mongoexport` produces a JSON or CSV export of data stored in a MongoDB instance. It is primarily used for integrating with external systems or generating reports.

### 5.1 Exporting to JSON and CSV

**Exporting a Collection to JSON:**
```bash
mongoexport --uri="mongodb://localhost:27017/mydb" --collection=users --out=users.json
```

**Exporting to CSV with Specific Fields:**
CSV exports require specifying the fields to include.
```bash
mongoexport --uri="mongodb://localhost:27017/mydb" --collection=orders --type=csv --fields="_id,customerId,totalAmount,status" --out=orders.csv
```

### 5.2 Advanced Exporting

**Exporting with a Query:**
```bash
mongoexport --uri="mongodb://localhost:27017/mydb" --collection=events --query='{ "type": "login_failure" }' --out=login_failures.json
```

**Using a File for Field Names:**
If you have many fields, you can list them in a file.
```bash
mongoexport --uri="mongodb://localhost:27017/mydb" --collection=products --type=csv --fieldFile=product_fields.txt --out=products.csv
```

## 6. Data Import with `mongoimport`

`mongoimport` imports content from an Extended JSON, CSV, or TSV export created by `mongoexport`, or potentially, another third-party export tool.

### 6.1 Importing JSON and CSV

**Importing a JSON File:**
```bash
mongoimport --uri="mongodb://localhost:27017/mydb" --collection=users --file=users.json
```

**Importing a CSV File with Header Line:**
```bash
mongoimport --uri="mongodb://localhost:27017/mydb" --collection=orders --type=csv --headerline --file=orders.csv
```

### 6.2 Advanced Importing Techniques

**Upserting Data:**
Updates existing documents and inserts new ones based on a matching field.
```bash
mongoimport --uri="mongodb://localhost:27017/mydb" --collection=inventory --file=inventory_update.json --mode=upsert --upsertFields=sku
```

**Handling Errors During Import:**
In production, you may want to continue importing even if some documents fail validation.
```bash
mongoimport --uri="mongodb://localhost:27017/mydb" --collection=logs --file=dirty_logs.json --bypassDocumentValidation
```

**Using Multiple Workers for Faster Import:**
Speeds up the import process by using multiple threads.
```bash
mongoimport --uri="mongodb://localhost:27017/mydb" --collection=large_dataset --file=huge_file.json --numInsertionWorkers=8
```

## 7. Worst-Case Scenarios and Tech Support Tactics

Tech support operations often involve dealing with critical failures. Here are some advanced tactics for worst-case scenarios.

### 7.1 Recovering from a Dropped Collection

If a collection is accidentally dropped, and you have an oplog backup, you can recover it.

1.  **Stop all writes** to the database if possible.
2.  **Dump the oplog** from the primary or a secondary node.
3.  **Identify the drop command** in the oplog and note its timestamp.
4.  **Restore the last full backup**.
5.  **Replay the oplog** up to the timestamp just before the drop command using `mongorestore --oplogLimit`.

### 7.2 Handling High CPU or Memory Usage

When a MongoDB node is unresponsive due to high resource usage:

1.  **Connect via `mongosh`** (you may need to connect locally if remote connections are timing out).
2.  **Check current operations:** `db.currentOp({ "active": true, "secs_running": { "$gt": 3 } })`
3.  **Kill long-running queries:** `db.killOp(<opid>)`
4.  **Analyze the logs** to identify the source of the queries.
5.  **Add missing indexes** or optimize the application code.

### 7.3 Repairing a Corrupted Database

If the MongoDB process crashes and the data files become corrupted (rare with WiredTiger, but possible):

1.  **Stop the `mongod` process.**
2.  **Run the repair command:** `mongod --repair --dbpath /var/lib/mongodb`
3.  **Check the logs** for any unrecoverable errors.
4.  **Restart the `mongod` process.**
5.  **Run `db.collection.validate({full: true})`** on critical collections to ensure data integrity.

## 8. Advanced One-Liners for Daily Operations

These one-liners are designed to save time and provide immediate insights during troubleshooting.

**Find the top 5 largest collections in a database:**
```bash
mongosh mydb --quiet --eval 'db.getCollectionNames().map(c => ({name: c, size: db[c].stats().size})).sort((a, b) => b.size - a.size).slice(0, 5)'
```

**Kill all queries running longer than 60 seconds:**
```bash
mongosh admin --quiet --eval 'db.currentOp({ "active": true, "secs_running": { "$gt": 60 } }).inprog.forEach(op => db.killOp(op.opid))'
```

**Export all user emails to a text file:**
```bash
mongoexport --uri="mongodb://localhost:27017/mydb" --collection=users --fields=email --type=csv | tail -n +2 > emails.txt
```

**Monitor replication lag in real-time (run in a loop):**
```bash
watch -n 2 'mongosh admin --quiet --eval "rs.printSlaveReplicationInfo()"'
```

## 9. Security Best Practices for CLI Tools

When using these CLI tools in a production environment, security must be a top priority.

*   **Never hardcode passwords** in scripts. Use environment variables or configuration files with restricted permissions.
*   **Use the `--askPassword` flag** or read passwords from a secure prompt when running commands manually.
*   **Restrict access to backup files.** `mongodump` outputs contain sensitive data. Ensure the output directory has strict permissions (e.g., `chmod 700`).
*   **Use TLS/SSL** for all connections, especially when connecting across different networks or the internet.
*   **Audit CLI usage.** Ensure that command history (e.g., `.bash_history`) does not contain sensitive connection strings.

## 10. Conclusion

Mastering the MongoDB CLI tools is essential for any tech support or operations professional. `mongosh`, `mongodump`, `mongorestore`, `mongoexport`, and `mongoimport` provide the necessary capabilities to manage, troubleshoot, and recover MongoDB deployments effectively. By understanding advanced techniques, such as point-in-time recovery, background index creation, and performance analysis, you can ensure the stability and reliability of your database infrastructure even in the most challenging situations.

## 11. Deep Dive: Performance Tuning and Diagnostics

Performance tuning is a continuous process. When users report slow response times, the database is often the first suspect. Here is a deeper dive into diagnosing and resolving performance bottlenecks using the CLI.

### 11.1 Profiling Slow Queries

MongoDB includes a built-in database profiler that logs detailed information about database operations.

**Enabling the Profiler:**
You can enable the profiler for a specific database to log queries that take longer than a specified threshold (e.g., 100 milliseconds).
```javascript
// Enable profiling for queries slower than 100ms
db.setProfilingLevel(1, { slowms: 100 })
```

**Analyzing Profiler Data:**
The profiler writes data to the `system.profile` collection. You can query this collection to identify the most problematic queries.
```javascript
// Find the top 10 slowest queries in the last hour
db.system.profile.find({
  ts: { $gt: new ISODate(Date.now() - 1000 * 60 * 60) }
}).sort({ millis: -1 }).limit(10)
```

### 11.2 Analyzing the WiredTiger Storage Engine

WiredTiger is the default storage engine for MongoDB. Understanding its internal state can help diagnose I/O bottlenecks and memory pressure.

**Viewing WiredTiger Statistics:**
```javascript
// Get detailed statistics for the WiredTiger storage engine
db.serverStatus().wiredTiger
```

**Key Metrics to Monitor:**
*   `cache.bytes currently in the cache`: Indicates how much memory WiredTiger is using.
*   `cache.maximum bytes configured`: The maximum memory WiredTiger is allowed to use.
*   `cache.tracked dirty bytes in the cache`: The amount of modified data that has not yet been written to disk. High values can indicate I/O bottlenecks.

### 11.3 Network Diagnostics

Network latency and bandwidth limitations can also impact MongoDB performance.

**Checking Network Statistics:**
```javascript
// View network connection statistics
db.serverStatus().connections
```

**Identifying Connection Spikes:**
A sudden increase in connections can overwhelm the database. Monitor the `current` and `created` connection metrics to detect anomalies.

## 12. Advanced Data Migration Strategies

Migrating data between MongoDB clusters or from other database systems requires careful planning and execution.

### 12.1 Live Migrations with `mongosync`

For zero-downtime migrations between MongoDB clusters, `mongosync` (part of the Cluster-to-Cluster Sync feature) is the recommended tool. While not covered in detail here, it is the enterprise standard for live migrations.

### 12.2 Phased Migrations using `mongoexport` and `mongoimport`

For smaller datasets or when migrating from legacy systems, a phased approach using `mongoexport` and `mongoimport` can be effective.

1.  **Export the initial dataset:** Use `mongoexport` to extract the data.
2.  **Import the data into the new cluster:** Use `mongoimport` to load the data.
3.  **Sync the delta:** Use a custom script or application logic to capture and apply changes that occurred during the initial migration.

### 12.3 Data Transformation during Import

`mongoimport` does not support complex data transformations. If you need to modify the data during migration, consider using a custom script (e.g., Python with PyMongo) or an ETL tool.

## 13. Troubleshooting Common Errors

Tech support engineers frequently encounter specific error messages. Here is a guide to troubleshooting some of the most common MongoDB errors.

### 13.1 "Connection Refused"

**Symptoms:** The application or CLI tool cannot connect to the MongoDB instance.
**Causes:**
*   The `mongod` process is not running.
*   A firewall is blocking the connection.
*   The `bindIp` configuration is restricting access.
**Resolution:**
1.  Check the status of the `mongod` service (`systemctl status mongod`).
2.  Verify firewall rules (e.g., `ufw status` or AWS Security Groups).
3.  Check the `net.bindIp` setting in `/etc/mongod.conf`.

### 13.2 "Authentication Failed"

**Symptoms:** The connection is established, but authentication fails.
**Causes:**
*   Incorrect username or password.
*   Connecting to the wrong authentication database (usually `admin`).
*   The user does not have the necessary roles.
**Resolution:**
1.  Verify the credentials.
2.  Ensure the connection string specifies the correct `authSource` (e.g., `?authSource=admin`).
3.  Check the user's roles using `db.getUser("username")`.

### 13.3 "Cursor Not Found"

**Symptoms:** A long-running query or aggregation pipeline fails with a "cursor not found" error.
**Causes:**
*   The cursor timed out on the server (default is 10 minutes of inactivity).
*   The MongoDB node restarted or stepped down.
**Resolution:**
1.  Use the `noCursorTimeout()` option for long-running queries (remember to close the cursor manually).
2.  Optimize the query to execute faster.
3.  Check the server logs for restarts or elections.

## 14. Scripting and Automation

Automating routine tasks is essential for efficient tech support operations. `mongosh` supports executing JavaScript files, making it a powerful automation tool.

### 14.1 Executing JavaScript Files

You can write complex logic in a `.js` file and execute it using `mongosh`.

**Example Script (`cleanup.js`):**
```javascript
// Connect to the database
const db = db.getSiblingDB('mydb');

// Find and delete old log entries
const threshold = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000); // 30 days ago
const result = db.logs.deleteMany({ timestamp: { $lt: threshold } });

print(`Deleted ${result.deletedCount} old log entries.`);
```

**Executing the Script:**
```bash
mongosh "mongodb://localhost:27017/mydb" cleanup.js
```

### 14.2 Integrating with Bash Scripts

You can combine `mongosh` with Bash scripting for system-level automation.

**Example Bash Script (`backup_and_notify.sh`):**
```bash
#!/bin/bash

# Perform the backup
mongodump --uri="mongodb://localhost:27017/mydb" --out=/backups/mydb_$(date +%F)

# Check the exit status
if [ $? -eq 0 ]; then
  echo "Backup successful."
  # Send a notification (e.g., via Slack or email)
else
  echo "Backup failed!"
  # Send an alert
fi
```

## 15. Comprehensive Reference Tables

### 15.1 `mongosh` Command Summary

| Command | Description | Example |
| :--- | :--- | :--- |
| `show dbs` | Lists all databases. | `show dbs` |
| `use <db>` | Switches to the specified database. | `use mydb` |
| `show collections` | Lists all collections in the current database. | `show collections` |
| `db.<collection>.find()` | Queries a collection. | `db.users.find({ age: { $gt: 18 } })` |
| `db.<collection>.insertOne()` | Inserts a single document. | `db.users.insertOne({ name: "Alice" })` |
| `db.<collection>.updateOne()` | Updates a single document. | `db.users.updateOne({ name: "Alice" }, { $set: { age: 30 } })` |
| `db.<collection>.deleteOne()` | Deletes a single document. | `db.users.deleteOne({ name: "Alice" })` |
| `db.<collection>.aggregate()` | Runs an aggregation pipeline. | `db.orders.aggregate([{ $match: { status: "A" } }])` |
| `db.<collection>.createIndex()` | Creates an index. | `db.users.createIndex({ email: 1 }, { unique: true })` |
| `rs.status()` | Shows the status of the replica set. | `rs.status()` |
| `db.serverStatus()` | Returns a document with server statistics. | `db.serverStatus()` |
| `db.currentOp()` | Returns information about in-progress operations. | `db.currentOp()` |
| `db.killOp()` | Terminates an operation. | `db.killOp(12345)` |

### 15.2 `mongodump` Options Summary

| Option | Description | Example |
| :--- | :--- | :--- |
| `--uri` | The connection string URI. | `--uri="mongodb://localhost:27017"` |
| `--out` | The output directory for the dump. | `--out=/backups/dump` |
| `--archive` | Writes the dump to a single archive file. | `--archive=/backups/dump.archive` |
| `--gzip` | Compresses the output using gzip. | `--gzip` |
| `--db` | Specifies the database to dump. | `--db=mydb` |
| `--collection` | Specifies the collection to dump. | `--collection=users` |
| `--query` | Provides a JSON query to filter the documents. | `--query='{ "status": "active" }'` |
| `--oplog` | Includes the oplog for point-in-time recovery. | `--oplog` |

### 15.3 `mongorestore` Options Summary

| Option | Description | Example |
| :--- | :--- | :--- |
| `--uri` | The connection string URI. | `--uri="mongodb://localhost:27017"` |
| `--archive` | Reads the dump from an archive file. | `--archive=/backups/dump.archive` |
| `--gzip` | Decompresses the input using gzip. | `--gzip` |
| `--drop` | Drops collections before restoring them. | `--drop` |
| `--nsFrom` | The namespace pattern to match in the dump. | `--nsFrom="mydb.*"` |
| `--nsTo` | The namespace pattern to restore to. | `--nsTo="staging_db.*"` |
| `--oplogReplay` | Replays the oplog included in the dump. | `--oplogReplay` |
| `--oplogLimit` | Replays the oplog up to a specific timestamp. | `--oplogLimit="1698278400"` |

### 15.4 `mongoexport` Options Summary

| Option | Description | Example |
| :--- | :--- | :--- |
| `--uri` | The connection string URI. | `--uri="mongodb://localhost:27017"` |
| `--collection` | Specifies the collection to export. | `--collection=users` |
| `--out` | The output file. | `--out=users.json` |
| `--type` | The output format (json or csv). | `--type=csv` |
| `--fields` | A comma-separated list of fields to export (required for CSV). | `--fields="name,email"` |
| `--query` | Provides a JSON query to filter the documents. | `--query='{ "status": "active" }'` |

### 15.5 `mongoimport` Options Summary

| Option | Description | Example |
| :--- | :--- | :--- |
| `--uri` | The connection string URI. | `--uri="mongodb://localhost:27017"` |
| `--collection` | Specifies the collection to import into. | `--collection=users` |
| `--file` | The input file. | `--file=users.json` |
| `--type` | The input format (json, csv, or tsv). | `--type=csv` |
| `--headerline` | Uses the first line of the CSV/TSV file as field names. | `--headerline` |
| `--mode` | The import mode (insert, upsert, or merge). | `--mode=upsert` |
| `--upsertFields` | A comma-separated list of fields to use for upserting. | `--upsertFields="email"` |
| `--bypassDocumentValidation` | Bypasses document validation during import. | `--bypassDocumentValidation` |
| `--numInsertionWorkers` | The number of concurrent workers to use. | `--numInsertionWorkers=4` |

## 16. Final Thoughts on Tech Support Readiness

Being prepared for any scenario is the hallmark of a great tech support engineer. The tools and techniques outlined in this document provide a solid foundation for managing MongoDB deployments. However, true mastery comes from practice and experience.

*   **Regularly test your backups:** A backup is only useful if you can restore it. Schedule regular test restorations to ensure your disaster recovery plan works.
*   **Document your procedures:** Create runbooks for common tasks and emergency procedures. This ensures consistency and reduces the risk of errors during stressful situations.
*   **Stay updated:** MongoDB is constantly evolving. Keep up with the latest releases, features, and best practices to ensure your skills remain relevant.
*   **Monitor proactively:** Don't wait for users to report issues. Implement robust monitoring and alerting to detect and resolve problems before they impact the business.

By combining the power of the MongoDB CLI tools with a proactive and disciplined approach, you can ensure the health, performance, and security of your database infrastructure.
