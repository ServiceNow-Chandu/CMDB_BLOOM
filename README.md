# CMDB Bloom – ServiceNow

## Project Overview

CMDB Bloom is a lightweight enterprise CMDB implementation developed in ServiceNow to demonstrate Configuration Management Database (CMDB) architecture, CI relationship modeling, identification and reconciliation strategies, controlled data ingestion, and governance reporting.

The project was designed using scoped application development practices integrated with GitHub source control to simulate enterprise CMDB engineering and governance workflows.

This implementation demonstrates how organizations maintain accurate infrastructure visibility, prevent duplicate Configuration Items (CIs), and establish service-to-infrastructure dependency mapping.

---

# Business Requirement

The organization required a centralized CMDB solution capable of:

* Modeling infrastructure relationships between services, servers, and databases
* Maintaining clean and governed CI records
* Preventing duplicate CI creation
* Importing infrastructure data through controlled pipelines
* Supporting dependency visualization
* Improving CMDB health and governance visibility

---

# Technologies Used

| Technology            | Purpose                  |
| --------------------- | ------------------------ |
| ServiceNow CMDB       | Configuration Management |
| Scoped Applications   | Modular development      |
| Import Sets           | Data ingestion           |
| Transform Maps        | CI transformation        |
| Identification Rules  | Duplicate prevention     |
| Business Rules        | Governance enforcement   |
| GlideRecord           | Relationship creation    |
| GitHub Source Control | Version management       |
| Dashboards & Reports  | Governance reporting     |

---

# Application Details

| Item              | Value              |
| ----------------- | ------------------ |
| Application Name  | CMDB Bloom         |
| Scope             | x_ns_cmdb_bloom    |
| Version           | 1.0.0              |
| Development Style | Scoped Application |
| Source Control    | GitHub Integrated  |

---

# CMDB Architecture

The CMDB was designed using enterprise-aligned CI hierarchy modeling.

---

## CI Relationship Structure

```text id="6j3b8g"
Business Service
      ↓
Application Service
      ↓ depends on
Server
      ↓ runs on
Database
```

---

# CMDB Classes Extended

# 1. Application Service

Extended from:

```text id="3w4q5u"
cmdb_ci_service_auto
```

---

## Custom Fields Added

| Field Label   | Field Name      | Type                       |
| ------------- | --------------- | -------------------------- |
| Service Owner | u_service_owner | Reference (sys_user)       |
| Support Group | u_support_group | Reference (sys_user_group) |
| Service Tier  | u_service_tier  | Choice                     |

---

# 2. Server CI

Extended from:

```text id="7mq8yv"
cmdb_ci_server
```

---

## Custom Fields Added

| Field Label | Field Name    | Type   |
| ----------- | ------------- | ------ |
| Environment | u_environment | Choice |
| OS Version  | u_os_version  | String |
| Build Date  | u_build_date  | Date   |

---

# 3. Database CI

Extended from:

```text id="oqt9gh"
cmdb_ci_database
```

---

## Custom Fields Added

| Field Label | Field Name   | Type       |
| ----------- | ------------ | ---------- |
| DB Engine   | u_db_engine  | Choice     |
| DB Version  | u_db_version | String     |
| HA Enabled  | u_ha_enabled | True/False |

---

# CI Relationship Modeling

Implemented enterprise-standard CI dependency relationships.

---

## Relationship Types Used

| Relationship          | Purpose              |
| --------------------- | -------------------- |
| Depends on :: Used by | Application → Server |
| Runs on :: Runs       | Server → Database    |

---

## Relationship Example

```text id="rvjij5"
Application Service
    ↓ depends on
Application Server
    ↓ runs on
Database
```

---

# Automated Relationship Logic

A Business Rule was designed to automatically create CI relationships between Server and Database records.

---

## Business Rule Purpose

* Automatically create relationships
* Reduce manual CI linking
* Improve dependency mapping consistency

---

## Relationship Automation Script

```javascript id="wn9f24"
(function executeRule(current, previous) {

  if (!current.hosted_on)
      return;

  var rel = new GlideRecord('cmdb_rel_ci');

  rel.initialize();

  rel.parent = current.hosted_on;

  rel.child = current.sys_id;

  rel.type = 'runs_on';

  rel.insert();

})();
```

---

# Identification & Reconciliation Strategy

Implemented Identification Rules to prevent duplicate Configuration Items and maintain CMDB integrity.

---

# Server Identification Rule

Target Table:

```text id="67w1fk"
cmdb_ci_server
```

---

## Identifiers Used

* Name
* IP Address
* Serial Number

---

# Database Identification Rule

Target Table:

```text id="7ujfd3"
cmdb_ci_database
```

---

## Identifiers Used

* Name
* DB Engine
* Version

---

# Duplicate Prevention Strategy

Duplicate prevention was implemented using:

| Method                 | Purpose                          |
| ---------------------- | -------------------------------- |
| Identification Rules   | CI uniqueness                    |
| Transform Map Coalesce | Update existing records          |
| Controlled Imports     | Prevent uncontrolled CI creation |

---

# Import Set Pipeline

Implemented a controlled Import Set process for onboarding server infrastructure records into the CMDB.

---

# Import Process Flow

```text id="jy6s3n"
CSV File
    ↓
Import Set Table
    ↓
Transform Map
    ↓
CMDB Server CI
```

---

# Sample Import Data

```csv id="gk72qx"
name,ip_address,u_environment,u_os_version
app-server-01,10.0.0.10,Prod,RHEL 8
app-server-02,10.0.0.11,QA,Ubuntu 22
```

---

# Transform Map Configuration

| Source Field  | Target Field  |
| ------------- | ------------- |
| name          | name          |
| ip_address    | ip_address    |
| u_environment | u_environment |
| u_os_version  | u_os_version  |

---

# Coalesce Strategy

The `name` field was used as the coalesce key to ensure:

* Existing records update correctly
* Duplicate servers are not created
* Infrastructure records remain synchronized

---

# Governance & CMDB Health

Implemented governance validation and reporting mechanisms to improve CMDB data quality.

---

# Required Field Governance

Implemented Business Rules to enforce mandatory CI metadata.

---

## Required Fields

| Field       | Validation |
| ----------- | ---------- |
| Name        | Mandatory  |
| Environment | Mandatory  |
| OS Version  | Mandatory  |

---

## Governance Validation Script

```javascript id="pnqz2y"
(function executeRule(current, previous) {

    var missing = [];

    if (gs.nil(current.name))
        missing.push('Name');

    if (gs.nil(current.u_environment))
        missing.push('Environment');

    if (gs.nil(current.u_os_version))
        missing.push('OS Version');

    if (missing.length > 0) {

        gs.addErrorMessage(
            'Missing required CMDB fields: ' +
            missing.join(', ')
        );

        current.setAbortAction(true);
    }

})();
```

---

# CMDB Governance Dashboard

Created governance reporting dashboards to visualize CMDB health metrics.

---

## Dashboard Widgets

| Widget                    | Purpose                |
| ------------------------- | ---------------------- |
| Servers by Environment    | Environment visibility |
| Missing Environment       | Data quality check     |
| Recently Imported Servers | Import tracking        |

---

# Dependency Visualization

Verified CI relationships using CMDB Dependency View.

---

## Dependency Validation

```text id="0zn6mq"
Application Service
      ↓ depends on
Server
      ↓ runs on
Database
```

---

# Testing Scenarios

# Scenario 1 – Import Validation

### Test

Re-import identical CSV data.

### Expected Result

* Existing records updated
* No duplicate servers created

---

# Scenario 2 – Governance Validation

### Test

Create Server CI with missing fields.

### Expected Result

* Record creation blocked
* Validation message displayed

---

# Scenario 3 – Dependency Validation

### Test

Open Dependency View.

### Expected Result

* CI hierarchy visible
* Relationships displayed correctly

---

# Scenario 4 – Dashboard Validation

### Test

Open Governance Dashboard.

### Expected Result

* Governance metrics visible
* Reports populated successfully

---

# GitHub Repository Structure

```text id="64n0rq"
04-cmdb-bloom/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── ci-relationships.md
│   ├── identification-reconciliation.md
│   ├── import-process.md
│   ├── governance-rules.md
│   ├── cmdb-health.md
│   └── demo-scenarios.md
│
├── sample-data/
│   └── server_import.csv
│
├── media/
│
└── update_sets/
```

---

# Enterprise Concepts Demonstrated

| Concept                 | Implemented |
| ----------------------- | ----------- |
| CMDB Modeling           | Yes         |
| CI Relationships        | Yes         |
| Dependency Mapping      | Yes         |
| Identification Rules    | Yes         |
| Reconciliation Strategy | Yes         |
| Controlled Imports      | Yes         |
| Governance Enforcement  | Yes         |
| CMDB Health Reporting   | Yes         |
| Duplicate Prevention    | Yes         |

---

# Technical Challenges Solved

## Challenge 1

Preventing duplicate CI creation.

### Solution

Implemented Identification Rules and Transform Map coalesce logic.

---

## Challenge 2

Maintaining accurate CI dependency mapping.

### Solution

Configured CI relationship modeling and automated linking.

---

## Challenge 3

Ensuring CMDB data quality.

### Solution

Developed governance validation Business Rules.

---

## Challenge 4

Importing infrastructure data in a controlled manner.

### Solution

Designed Import Set and Transform Map pipeline.

---

# Project Outcome

The CMDB Bloom implementation successfully demonstrated enterprise-style CMDB architecture, governance, CI relationship management, dependency visualization, and controlled infrastructure onboarding.

The project improved:

* Infrastructure visibility
* CMDB data quality
* Governance consistency
* Duplicate prevention
* Service dependency mapping

---
