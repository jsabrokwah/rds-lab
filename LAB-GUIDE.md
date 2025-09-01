# AWS RDS Labs - Step-by-Step Instructions
## DevOps Training - Solutions Architect Associate Preparation

---

## Prerequisites for All Labs

### Required Access:
- AWS Account with administrative permissions
- MySQL client or web-based database tool (phpMyAdmin, MySQL Workbench, or DBeaver)
- Basic SQL knowledge for testing
- SSH client for EC2 connection (Lab 3)

### Important Notes:
- **Use your initials** in all resource names (e.g., `js-rds-lab1-20250824`)
- **Work in us-east-1** region unless specified otherwise
- **Monitor costs** - RDS instances incur charges while running
- **Clean up resources** immediately after each lab
- **Document observations** for exam preparation

### Cost Warning:
⚠️ **These labs will incur charges!** Even the smallest RDS instances cost ~$0.017/hour. Complete labs promptly and clean up immediately.

---

## Lab 1: RDS Instance Creation and Basic Operations

### Objective:
Create RDS instances with different engines and configurations, understand instance classes, and perform basic database operations.

### Duration: 60 minutes

### Step 1: Create MySQL RDS Instance (20 minutes)

1. **Navigate to RDS Console:**
   - Login to AWS Console
   - Navigate to RDS service
   - Click "Create database"

2. **Choose Database Creation Method:**
   - Select "Standard create" (gives more configuration options)
   - Do NOT select "Easy create"

3. **Engine Selection:**
   - Engine type: "MySQL"
   - Version: Latest available (likely MySQL 8.0.x)
   - Leave default version selected

4. **Templates:**
   - Select "Free tier" (if available in your account)
   - If not available, select "Dev/Test"

5. **Settings Configuration:**
   - DB instance identifier: `[YOUR-INITIALS]-mysql-lab1`
   - Master username: `admin`
   - Master password: `Lab123456!` (or use auto-generate)
   - Confirm password if entered manually

6. **Instance Configuration:**
   - DB instance class: `db.t3.micro` (or smallest available)
   - Storage type: "General Purpose SSD (gp2)"
   - Allocated storage: 20 GB (minimum)
   - Enable storage autoscaling: Leave checked
   - Maximum storage threshold: 100 GB

7. **Connectivity:**
   - VPC: Default VPC
   - Subnet group: default
   - Public access: **Yes** (for lab purposes only)
   - VPC security group: Create new
   - Security group name: `[YOUR-INITIALS]-mysql-sg`
   - Availability Zone: No preference
   - Database port: 3306 (default)

8. **Database Authentication:**
   - Database authentication: "Password authentication"

9. **Additional Configuration:**
   - Initial database name: `labdb`
   - DB parameter group: default
   - Option group: default
   - Automated backups: Enable
   - Backup retention period: 7 days
   - Backup window: Default
   - Copy tags to snapshots: Yes
   - Enable encryption: No (for simplicity)
   - Enable Enhanced monitoring: No
   - Enable Performance Insights: No
   - Maintenance window: Default
   - Auto minor version upgrade: Yes
   - Deletion protection: **No** (important for cleanup)

10. **Create Database:**
    - Review all settings
    - Click "Create database"
    - Wait for status to change to "Available" (~10-15 minutes)

### Step 2: Create PostgreSQL RDS Instance (15 minutes)

11. **Create Second Database:**
    - Click "Create database" again
    - Engine type: "PostgreSQL"
    - Version: Latest available
    - Template: "Free tier" or "Dev/Test"

12. **Settings:**
    - DB instance identifier: `[YOUR-INITIALS]-postgres-lab1`
    - Master username: `postgres`
    - Master password: `Lab123456!`

13. **Instance Configuration:**
    - Same settings as MySQL instance
    - Port: 5432 (default for PostgreSQL)

14. **Additional Configuration:**
    - Initial database name: `labdb`
    - Other settings same as MySQL

15. **Create and Wait:**
    - Create database
    - Both databases should be available

### Step 3: Configure Security Groups (10 minutes)

16. **Modify MySQL Security Group:**
    - Go to EC2 → Security Groups
    - Find your MySQL security group
    - Click "Edit inbound rules"
    - Add rule:
      - Type: MySQL/Aurora
      - Protocol: TCP
      - Port: 3306
      - Source: Your IP address (select "My IP")
    - Save rules

17. **Modify PostgreSQL Security Group:**
    - Repeat for PostgreSQL security group
    - Add rule:
      - Type: PostgreSQL
      - Protocol: TCP
      - Port: 5432
      - Source: Your IP address
    - Save rules

### Step 4: Connect and Test Databases (15 minutes)

18. **Gather Connection Information:**
    - In RDS console, click on MySQL instance
    - Copy the "Endpoint" value
    - Note the port (3306)
    - Repeat for PostgreSQL instance

19. **Test MySQL Connection:**
    Using MySQL command line client:
    ```bash
    mysql -h [YOUR-MYSQL-ENDPOINT] -P 3306 -u admin -p
    # Enter password: Lab123456!
    
    # Test commands:
    SHOW DATABASES;
    USE labdb;
    CREATE TABLE test_table (id INT PRIMARY KEY, name VARCHAR(50));
    INSERT INTO test_table VALUES (1, 'Test Data');
    SELECT * FROM test_table;
    EXIT;
    ```

20. **Test PostgreSQL Connection:**
    Using psql client:
    ```bash
    psql -h [YOUR-POSTGRES-ENDPOINT] -p 5432 -U postgres -d labdb
    # Enter password: Lab123456!
    
    # Test commands:
    \l
    \c labdb
    CREATE TABLE test_table (id SERIAL PRIMARY KEY, name VARCHAR(50));
    INSERT INTO test_table (name) VALUES ('Test Data');
    SELECT * FROM test_table;
    \q
    ```

### Lab 1 Assessment Questions:
1. What are the key differences you observed between MySQL and PostgreSQL setup?
2. How long did it take for each instance to become available?
3. What would happen if you chose a larger instance class?

### Lab 1 Deliverables:
- Screenshots of both RDS instances showing "Available" status
- Screenshots of successful database connections
- Cost estimate for running these instances for 24 hours

---

## Lab 2: Multi-AZ Deployment and Failover Testing

### Objective:
Configure Multi-AZ deployment, understand failover behavior, and test high availability features.

### Duration: 45 minutes

### Step 1: Enable Multi-AZ on Existing Instance (15 minutes)

1. **Modify MySQL Instance:**
   - In RDS console, select your MySQL instance
   - Click "Modify"
   - Scroll to "Availability & durability"
   - Multi-AZ deployment: Change to "Create a standby instance"
   - Click "Continue"

2. **Schedule Modification:**
   - When to apply: "Apply immediately"
   - Click "Modify DB instance"
   - **Note:** This will cause a brief outage while enabling Multi-AZ

3. **Monitor the Process:**
   - Watch the instance status change to "Modifying"
   - This process takes 10-15 minutes
   - Status will return to "Available" when complete

### Step 2: Verify Multi-AZ Configuration (10 minutes)

4. **Check Multi-AZ Status:**
   - Click on your modified MySQL instance
   - In the "Configuration" tab, verify:
     - Multi-AZ: Yes
     - Secondary AZ: Should show different AZ than primary

5. **Understand the Setup:**
   - Note that you still have only one endpoint
   - The standby instance is not visible separately
   - Connection string remains the same

6. **Test Connection Still Works:**
   ```bash
   mysql -h [YOUR-MYSQL-ENDPOINT] -P 3306 -u admin -p
   SELECT @@hostname;  # Shows which instance you're connected to
   SHOW DATABASES;
   EXIT;
   ```

### Step 3: Perform Failover Testing (15 minutes)

7. **Initiate Manual Failover:**
   - In RDS console, select your Multi-AZ instance
   - Actions → "Reboot"
   - Check "Reboot with failover"
   - Click "Reboot"

8. **Monitor Failover Process:**
   - Watch instance status change to "Rebooting"
   - Time the failover process (typically 1-2 minutes)
   - Status returns to "Available"

9. **Verify Failover Completed:**
   - Check if the Primary AZ changed in the Configuration tab
   - Original primary should now be secondary
   - Test database connection again

10. **Document Observations:**
    - Record failover duration
    - Note any application impact
    - Verify data integrity

### Step 4: Compare Single-AZ vs Multi-AZ (5 minutes)

11. **Cost Comparison:**
    - Compare estimated costs:
      - Single-AZ: 1 × instance cost
      - Multi-AZ: ~2 × instance cost
    - Document the cost difference

12. **Availability Comparison:**
    - Single-AZ: Subject to AZ failures
    - Multi-AZ: Survives AZ failures with automatic failover

### Lab 2 Assessment Questions:
1. How long did the Multi-AZ enablement process take?
2. What was the actual failover duration during testing?
3. How does Multi-AZ affect your connection string?
4. What is the cost impact of enabling Multi-AZ?

### Lab 2 Deliverables:
- Screenshot showing Multi-AZ enabled status
- Timing records for Multi-AZ enablement and failover
- Before/after screenshots of Primary AZ configuration

---

## Lab 3: Read Replicas and Performance Scaling

### Objective:
Create read replicas, test read scaling, and understand cross-region replication capabilities.

### Duration: 50 minutes

### Step 1: Create Read Replica in Same Region (20 minutes)

1. **Create Local Read Replica:**
   - Select your MySQL instance (ensure it's Multi-AZ enabled)
   - Actions → "Create read replica"

2. **Read Replica Settings:**
   - DB instance identifier: `[YOUR-INITIALS]-mysql-replica-local`
   - Destination region: "Same as source"
   - Destination AZ: Different from source (if Multi-AZ source)
   - Instance class: Can be same or different (try `db.t3.micro`)

3. **Network & Security:**
   - VPC: Same as source
   - Subnet group: Same as source
   - Public accessibility: Yes
   - VPC security groups: Same as source
   - Availability Zone: No preference

4. **Database Options:**
   - Database port: 3306
   - Parameter group: default
   - Option group: default
   - Enable Enhanced monitoring: No
   - Enable Performance Insights: No

5. **Backup & Maintenance:**
   - Copy tags from source: Yes
   - Enable encryption: No
   - Backup retention: 0 days (read replicas can have different backup settings)
   - Auto minor version upgrade: Yes

6. **Create Replica:**
   - Click "Create read replica"
   - Wait for status to become "Available" (~10-15 minutes)

### Step 2: Create Cross-Region Read Replica (15 minutes)

7. **Create Cross-Region Replica:**
   - Select source MySQL instance again
   - Actions → "Create read replica"

8. **Cross-Region Configuration:**
   - DB instance identifier: `[YOUR-INITIALS]-mysql-replica-west`
   - **Destination region: "US West (Oregon)" - us-west-2**
   - Instance class: `db.t3.micro`

9. **Network Settings:**
   - VPC: Default VPC (in us-west-2)
   - Public accessibility: Yes
   - Create new security group in us-west-2

10. **Create and Wait:**
    - Create the cross-region replica
    - Switch to us-west-2 region in console to monitor
    - Wait for "Available" status

### Step 3: Test Read Replica Functionality (10 minutes)

11. **Test Local Read Replica:**
    ```bash
    # Connect to local read replica
    mysql -h [LOCAL-REPLICA-ENDPOINT] -P 3306 -u admin -p
    
    # Test read operations
    SELECT * FROM labdb.test_table;
    
    # Try write operation (should fail)
    INSERT INTO labdb.test_table VALUES (2, 'This should fail');
    # Expected: ERROR 1290 (HY000): The MySQL server is running with the --read-only option
    
    EXIT;
    ```

12. **Test Cross-Region Replica:**
    - Update security group in us-west-2 to allow your IP
    - Connect to cross-region replica
    - Test same read operations

13. **Test Replication Lag:**
    ```bash
    # Connect to source (master)
    mysql -h [MASTER-ENDPOINT] -P 3306 -u admin -p
    INSERT INTO labdb.test_table VALUES (3, 'Replication test');
    SELECT NOW();  # Note the time
    EXIT;
    
    # Immediately connect to replica
    mysql -h [REPLICA-ENDPOINT] -P 3306 -u admin -p
    SELECT * FROM labdb.test_table WHERE id = 3;
    SELECT NOW();  # Compare time
    EXIT;
    ```

### Step 4: Read Replica Promotion Testing (5 minutes)

14. **Promote Read Replica:**
    - In RDS console, select local read replica
    - Actions → "Promote read replica"
    - This breaks replication and makes it independent
    - **Warning:** This is irreversible!

15. **Verify Promotion:**
    - After promotion, replica becomes independent RDS instance
    - Can now accept write operations
    - No longer synchronized with original master

### Lab 3 Assessment Questions:
1. What was the replication lag between master and replica?
2. What happens when you try to write to a read replica?
3. How does cross-region replication affect data transfer costs?
4. What are the use cases for promoting a read replica?

### Lab 3 Deliverables:
- Screenshots of both local and cross-region replicas
- Documentation of replication lag testing results
- Screenshot of promoted replica status

---

## Lab 4: Automated Backups and Point-in-Time Recovery

### Objective:
Configure backup settings, perform point-in-time recovery, and understand backup/restore processes.

### Duration: 40 minutes

### Step 1: Configure Backup Settings (10 minutes)

1. **Modify Backup Configuration:**
   - Select your original MySQL instance
   - Click "Modify"
   - Scroll to "Backup" section

2. **Update Backup Settings:**
   - Backup retention period: 14 days
   - Backup window: Choose specific time (e.g., 03:00-04:00 UTC)
   - Copy tags to snapshots: Yes
   - Click "Continue" → "Apply immediately"

3. **Verify Backup Configuration:**
   - Check that automated backups are enabled
   - Note the backup window setting
   - Understand backup storage costs

### Step 2: Create Manual Snapshot (10 minutes)

4. **Create Manual Snapshot:**
   - Select your MySQL instance
   - Actions → "Take snapshot"
   - Snapshot identifier: `[YOUR-INITIALS]-mysql-manual-snapshot-1`
   - Click "Take snapshot"

5. **Monitor Snapshot Creation:**
   - Go to "Snapshots" in left navigation
   - Watch snapshot status change to "Available"
   - Note the snapshot size and creation time

6. **Create Test Data:**
   ```bash
   mysql -h [YOUR-MYSQL-ENDPOINT] -P 3306 -u admin -p
   USE labdb;
   CREATE TABLE backup_test (
     id INT PRIMARY KEY,
     data VARCHAR(100),
     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   INSERT INTO backup_test VALUES (1, 'Before snapshot', NOW());
   SELECT * FROM backup_test;
   EXIT;
   ```

### Step 3: Simulate Data Loss and Recovery (15 minutes)

7. **Create More Test Data:**
   ```bash
   mysql -h [YOUR-MYSQL-ENDPOINT] -P 3306 -u admin -p
   USE labdb;
   INSERT INTO backup_test VALUES (2, 'Data to be lost', NOW());
   INSERT INTO backup_test VALUES (3, 'More data to lose', NOW());
   SELECT * FROM backup_test;
   # Note the current time
   SELECT NOW();
   EXIT;
   ```

8. **Simulate Accidental Deletion:**
   ```bash
   mysql -h [YOUR-MYSQL-ENDPOINT] -P 3306 -u admin -p
   USE labdb;
   DROP TABLE backup_test;  # Simulate accidental deletion
   SHOW TABLES;  # Verify table is gone
   EXIT;
   ```

9. **Perform Point-in-Time Recovery:**
   - In RDS console, select your MySQL instance
   - Actions → "Restore to point in time"
   - Use automated backup to restore

10. **Configure Restored Instance:**
    - DB instance identifier: `[YOUR-INITIALS]-mysql-restored`
    - Restore time: Choose "Custom" and select time before deletion
    - Instance class: Same as source
    - Other settings: Same as source
    - **Important:** This creates a NEW instance

11. **Create Restored Instance:**
    - Click "Restore DB instance"
    - Wait for new instance to become available

### Step 4: Verify Recovery (5 minutes)

12. **Test Restored Data:**
    ```bash
    # Connect to restored instance
    mysql -h [RESTORED-INSTANCE-ENDPOINT] -P 3306 -u admin -p
    USE labdb;
    SHOW TABLES;  # backup_test table should exist
    SELECT * FROM backup_test;  # Should show data before deletion
    EXIT;
    ```

13. **Compare Instances:**
    - Original instance: Missing backup_test table
    - Restored instance: Has backup_test table with data
    - Demonstrates point-in-time recovery capability

### Lab 4 Assessment Questions:
1. How does backup retention period affect storage costs?
2. What's the difference between manual snapshots and automated backups?
3. How long did the point-in-time recovery process take?
4. What are the limitations of point-in-time recovery?

### Lab 4 Deliverables:
- Screenshots of backup configuration settings
- Documentation of recovery process timing
- Screenshots showing data before and after recovery

---

## Post-Lab Cleanup Instructions

### Critical: Clean Up All Resources to Avoid Charges

#### RDS Instances:
1. **Delete All RDS Instances:**
   ```
   - Original MySQL instance
   - PostgreSQL instance  
   - Local read replica (if not promoted)
   - Cross-region read replica
   - Promoted read replica
   - Restored instance
   ```

2. **Deletion Process:**
   - Select instance → Actions → "Delete"
   - Uncheck "Create final snapshot" (for lab purposes)
   - Check "I acknowledge that automatic backups will be deleted"
   - Type "delete me" to confirm
   - Click "Delete"

#### Manual Snapshots:
3. **Delete Manual Snapshots:**
   - Go to "Snapshots" in RDS console
   - Select your manual snapshots
   - Actions → "Delete snapshot"
   - Confirm deletion

#### Security Groups:
4. **Clean Up Security Groups:**
   - Go to EC2 → Security Groups
   - Delete the RDS security groups you created
   - (Only delete if no longer referenced by instances)

#### Cross-Region Resources:
5. **Switch to us-west-2:**
   - Delete cross-region read replica
   - Delete associated security groups
   - Verify no resources remain

### Verification Checklist:
- [ ] All RDS instances deleted (check both regions)
- [ ] All manual snapshots deleted
- [ ] Custom security groups deleted
- [ ] No remaining charges in billing dashboard

---

## Lab Assessment and Exam Preparation

### Lab 1 Key Takeaways:
- **Engine selection** impacts features and costs
- **Instance classes** should match workload requirements
- **Security groups** control database access
- **Connection endpoints** remain consistent

### Lab 2 Key Takeaways:
- **Multi-AZ** provides high availability, not performance
- **Failover time** is typically 1-2 minutes
- **Cost doubles** with Multi-AZ enabled
- **Applications** don't need connection string changes

### Lab 3 Key Takeaways:
- **Read replicas** improve read performance
- **Cross-region replicas** provide DR capability
- **Replication lag** varies based on distance/load
- **Promotion** breaks replication permanently

### Lab 4 Key Takeaways:
- **Automated backups** enable point-in-time recovery
- **Manual snapshots** persist beyond instance deletion
- **Recovery creates new instance**, doesn't replace existing
- **Backup retention** affects storage costs

### Common Exam Scenarios:

**"Application needs 99.99% availability"**
→ Multi-AZ deployment with automated failover

**"Read-heavy workload causing performance issues"**
→ Read replicas to distribute read load

**"Need disaster recovery in different region"**
→ Cross-region read replica for DR

**"Accidentally deleted critical data"**
→ Point-in-time recovery from automated backups

**"Need long-term backup retention for compliance"**
→ Manual snapshots with appropriate retention

---

## Troubleshooting Guide

### Common Issues:

#### Connection Problems:
- **Error:** "Can't connect to MySQL server"
  - **Check:** Security group allows your IP
  - **Check:** Public accessibility enabled
  - **Check:** Correct endpoint and port

#### Multi-AZ Issues:
- **Error:** "Cannot modify to Multi-AZ"
  - **Check:** Instance is in "Available" state
  - **Check:** Sufficient account limits

#### Read Replica Problems:
- **Error:** "Cannot create read replica"
  - **Check:** Automated backups enabled on source
  - **Check:** Source instance is "Available"
  - **Check:** Account limits not exceeded

#### Backup/Recovery Issues:
- **Error:** "Cannot restore to point in time"
  - **Check:** Automated backups enabled
  - **Check:** Restore time within retention period
  - **Check:** Different instance identifier used

### Getting Help:
- Check AWS CloudTrail for detailed API errors
- Review RDS Events in console for instance-specific issues
- Monitor CloudWatch metrics for performance problems
- Contact AWS Support for account-specific issues

---

## Final Lab Report Template

### Complete this report for all labs:

#### Instance Performance:
- Creation time for each instance type
- Connection establishment time
- Query response times

#### Cost Analysis:
- Estimated hourly costs for each configuration
- Multi-AZ cost impact
- Read replica additional costs
- Storage costs for backups

#### High Availability Testing:
- Multi-AZ failover duration
- Application impact during failover
- Data consistency verification

#### Backup and Recovery:
- Backup window impact on performance
- Point-in-time recovery duration
- Data integrity after recovery

#### Lessons Learned:
- Which configurations best suit different use cases?
- What surprised you about RDS behavior?
- How would you apply this knowledge in production?
- What additional testing would you perform?

### Submission Format:
- **Lab report:** [YOUR-NAME]-RDS-Lab-Report.pdf
- **Screenshots:** [YOUR-NAME]-RDS-Screenshots/
- **SQL scripts:** [YOUR-NAME]-RDS-Test-Scripts.sql