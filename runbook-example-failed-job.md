# Runbook: Databricks Job Failure (Standard Workflow)

## 1. Incident Overview
- Title: Job Failure in Scheduled Workflow
- Detected by: Workflow Alert / Email / Monitoring dashboard
- Severity: SEV-2 (affects daily pipeline) or SEV-3 (non-critical)
- Impact:
  - Pipeline did not complete
  - Downstream datasets stale
  - KPIs / dashboards delayed

---

## 2. Preconditions / Required Access
Required Roles:
- Workspace Admin or Job Owner
- UC permissions for relevant tables (SELECT / MODIFY)

Required Tools:
- Access to Workflows UI
- Access to Databricks system tables
- CLI or API (optional for retry automation)

---

## 3. Initial Diagnosis

### 3.1 Identify failing job run
UI:
Workflows → Job Name → Runs → Failed run

System Tables query:
SQL:
SELECT job_id, run_id, state, start_time, end_time, task_key
FROM system.workspace.jobs
WHERE state = 'FAILED'
ORDER BY start_time DESC
LIMIT 20;

### 3.2 Inspect error message
- Open "Run Output"
- Check top-level error: Python/Spark/SQL
- Identify exact task that failed (multi-task jobs)

Common patterns:
- Missing permissions
- Cluster failed to start
- Out-of-memory
- Bad input data
- Schema mismatch
- Lakehouse path missing

### 3.3 Check cluster health (if using job cluster)
SQL:
SELECT cluster_id, cluster_name, state
FROM system.compute.clusters
ORDER BY last_activity_time DESC;

### 3.4 Check recent permission denials (common root cause)
SQL:
SELECT *
FROM system.access.audit
WHERE action_name LIKE '%DENY%' 
ORDER BY event_time DESC;

---

## 4. Recovery Actions

### 4.1 If cluster failed to start
Actions:
- Restart job cluster manually (Run Now)
- Check driver logs (Compute → Cluster → Driver Logs)
- Ensure cluster has access to ADLS (permissions / secret scopes)

### 4.2 If SQL/Python error in notebook
Actions:
- Open the notebook referenced by the task
- Review failing cell
- Identify:
  - Wrong file path
  - Bad schema
  - Null/empty data ingestion
  - Invalid SQL syntax introduced recently

### 4.3 If failure caused by missing permissions
SQL:
GRANT SELECT ON TABLE <table> TO `<group>`;
OR  
GRANT USAGE ON SCHEMA <schema> TO `<group>`;

### 4.4 If data input is corrupted
- Locate input file in ADLS
- Validate schema / JSON structure
- Delete bad file
- Or move it to a quarantine folder

### 4.5 Manual retry
UI:
Workflows → Job → "Run Now"

CLI:
databricks jobs run-now --job-id <id>

---

## 5. Validation

### 5.1 Validate job completion
- Confirm new job run shows "Succeeded"
- Check all downstream tasks (if multi-step workflow)

### 5.2 Validate datasets updated
SQL:
SELECT COUNT(*) FROM silver.table;
SELECT MAX(ingest_date) FROM gold.table;

### 5.3 Validate cluster terminates correctly
- Check for zombie all-purpose clusters left running
- Ensure auto-termination is working

---

## 6. Escalation Path

If still failing:
- Escalate to Data Engineering (pipeline owners)
- Escalate to Cloud Team if cluster provisioning keeps failing
- Escalate to Security if permission errors persist

Trigger Databricks Support if:
- Cluster repeatedly terminates unexpectedly
- DBFS root issues or internal platform errors

---

## 7. Postmortem
- Error Source: Code? Cluster? Permissions? Input?
- Root Cause:
- Fix Applied:
- Should job have retries enabled (YES/NO)?
- Add alert rules if missing
- Update runbook if needed
