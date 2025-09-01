# Lab 1: RDS Instance Creation and Basic Operations
## Documentation Report

### Lab Completion Status: ✅ COMPLETED

### Objective Achieved:
Successfully created RDS instances with different engines (MySQL and PostgreSQL), configured security groups, and performed basic database operations.

---

## Lab Execution Summary

### Step 1: MySQL RDS Instance Creation
- **Instance Identifier**: `js-mysql-lab1`
- **Engine**: MySQL 8.0.x
- **Instance Class**: db.t3.micro
- **Storage**: 20 GB General Purpose SSD (gp2)
- **Creation Time**: ~12 minutes
- **Status**: Available ✅

### Step 2: PostgreSQL RDS Instance Creation
- **Instance Identifier**: `js-postgres-lab1`
- **Engine**: PostgreSQL (latest version)
- **Instance Class**: db.t3.micro
- **Storage**: 20 GB General Purpose SSD (gp2)
- **Creation Time**: ~14 minutes
- **Status**: Available ✅

### Step 3: Security Group Configuration
- **MySQL Security Group**: Configured port 3306 access
- **PostgreSQL Security Group**: Configured port 5432 access
- **Source**: My IP address for secure access
- **Status**: Successfully configured ✅

### Step 4: Database Connection Testing
- **MySQL Connection**: Successful ✅
- **PostgreSQL Connection**: Successful ✅
- **Test Tables Created**: Both databases operational
- **Data Operations**: INSERT/SELECT operations verified

---

## Assessment Questions & Answers

### 1. What are the key differences you observed between MySQL and PostgreSQL setup?

**Answer:**
- **Port Numbers**: MySQL uses port 3306, PostgreSQL uses port 5432
- **Default Username**: MySQL allows custom admin username, PostgreSQL defaults to 'postgres'
- **SQL Syntax**: PostgreSQL uses `\l` for listing databases, MySQL uses `SHOW DATABASES;`
- **Table Creation**: PostgreSQL uses SERIAL for auto-increment, MySQL uses AUTO_INCREMENT
- **Connection Commands**: Different client tools (mysql vs psql)

### 2. How long did it take for each instance to become available?

**Answer:**
- **MySQL Instance**: Approximately 12 minutes from creation to "Available" status
- **PostgreSQL Instance**: Approximately 14 minutes from creation to "Available" status
- **Total Setup Time**: ~30 minutes including security group configuration and testing

### 3. What would happen if you chose a larger instance class?

**Answer:**
- **Performance**: Better CPU, memory, and network performance
- **Cost Impact**: Significantly higher hourly costs (e.g., db.t3.small costs ~2x more than db.t3.micro)
- **Capabilities**: Support for more concurrent connections and higher throughput
- **Use Case**: Larger instances suitable for production workloads, while db.t3.micro is ideal for development/testing

---

## Key Observations

### Technical Insights:
- RDS instances require 10-15 minutes for initial provisioning
- Security groups must be configured before external connections work
- Both engines support similar basic operations but with different syntax
- Connection endpoints remain consistent throughout instance lifecycle

### Cost Considerations:
- **db.t3.micro**: ~$0.017/hour per instance
- **Storage**: ~$0.10/GB/month for GP2
- **Total Lab Cost**: ~$0.034/hour for both instances running

### Security Notes:
- Public accessibility enabled for lab purposes only
- Production environments should use private subnets
- Security groups provide network-level access control
- Database authentication uses master username/password

---

## Deliverables Completed:
- ✅ Screenshots of both RDS instances showing "Available" status
- ✅ Screenshots of successful database connections  
- ✅ Cost estimate documentation
- ✅ Functional testing of both database engines

---

## Lab 1 Key Takeaways:
- **Engine Selection**: Impacts features, syntax, and operational characteristics
- **Instance Classes**: Should match workload requirements and budget constraints
- **Security Groups**: Critical for controlling database access at network level
- **Connection Endpoints**: Provide consistent access point regardless of underlying infrastructure