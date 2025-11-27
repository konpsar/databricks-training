# Databricks Operational Runbook Template

## 1. Incident Overview
- Title:
- Date/Time Detected:
- Detected By: (Monitoring alert / User report / Job failure / System tables)
- Severity: SEV-1 / SEV-2 / SEV-3
- Impact Summary:
  - Affected pipelines:
  - Affected users:
  - Affected data sources/destinations:
  - Business impact:

## 2. Preconditions / Required Access
### Required Roles
- Databricks Workspace Admin
- Unity Catalog Metastore Admin
- Cluster Policy Admin
- (Optional) Account Admin

### Required Tools
- Databricks CLI (profile configured)
- API token or OAuth service principal
- Access to Azure Portal (Storage, Log Analytics, Networking)

### Required Systems Availability
- ADLS Gen2 reachable
- Control Plane status = healthy
- Network endpoints reachable

## 3. Initial Diagnosis Steps

### 3.1 Check Audit / Access Events
SQL:
SELECT * FROM system.access.audit ORDER BY event_time DESC LIMIT 50;

### 3.2 Check Cluster Status
SQL:
SELECT cluster_id, cluster_name, state, last_activity_time
FROM system.compute.clusters
ORDER BY last_activity_time DESC;

### 3.3 Check Recent Job Failures
SQL:
SELECT * FROM system.workspace.jobs
WHERE state = 'FAILED'
ORDER BY start_time DESC
LIMIT 20;

### 3.4 Check Cost / Spikes
SQL:
SELECT * FROM system.billing.usage
WHERE usage_date >= current_date() - 1
ORDER BY usage_date DESC;

### 3.5 Check Cluster & Driver Logs
- UI → Compute → Cluster → Driver Logs
- UI → Job → Run → Output / Error logs

## 4. Recovery Actions (Examples)

- Restart a cluster:
  CLI: databricks clusters restart --cluster-id <id>

- Re-run a failed job:
  CLI: databricks jobs run-now --job-id <job-id>

- Fix permissions:
  SQL: GRANT SELECT ON TABLE silver.table1 TO `finance-readers`;

- Fix missing columns:
  SQL: ALTER TABLE silver.table1 ADD COLUMN new_col STRING;

## 5. Validation & Verification

### Validate Data
SQL:
SELECT COUNT(*) FROM bronze.table1;
SELECT COUNT(*) FROM silver.table1;

### Validate Permissions
SQL:
SHOW GRANTS ON TABLE silver.table1;

### Validate Workflows
- Trigger job with "Run Now"
- Confirm no new errors in logs
- Confirm clusters terminate properly

## 6. Escalation Path
If issue unresolved after 20–30 minutes:
- Data Platform Team:
- Azure Cloud Team:
- Security / IAM:
- Vendor escalation: Databricks Support (SEV-1 for production outage)

## 7. Postmortem Requirements
- Summary of incident
- Timeline (detection → diagnosis → fix → validation)
- Root cause analysis
- Preventive actions
- Update this runbook if needed
- Attach relevant logs & screenshots
