# Runbook: Unity Catalog Permission Mismatch

## 1. Incident Overview
- Title: UC permission mismatch / user cannot access table
- Detected by: User report or failed job
- Severity: SEV-3 (unless impacting production pipelines)
- Impact:
  - User or team cannot read/write specific tables or schemas
  - Job failures due to permission errors (e.g. MISSING_PRIVILEGE, ACCESS_DENIED)

## 2. Preconditions / Required Access
Required Roles:
- Unity Catalog Metastore Admin OR
- Catalog Owner OR
- Schema Owner

Required Tools:
- Databricks workspace
- UI or Databricks SQL access

## 3. Initial Diagnosis

### 3.1 Identify the user and object
Ask/report includes:
- User email
- Table or schema path (for example: catalog.schema.table)

### 3.2 Check permissions using SHOW GRANTS
SQL:
SHOW GRANTS ON TABLE catalog.schema.table;

### 3.3 Check effective user group membership
- Azure AD group membership (Entra ID)
- Databricks workspace groups (SCIM synced)

### 3.4 Check audit log for denied actions
SQL:
SELECT *
FROM system.access.audit
WHERE action_name LIKE '%DENY%' 
  AND resource_name LIKE '%<table_or_schema_name>%'
ORDER BY event_time DESC;

Common causes:
- Missing SELECT on table
- Missing USAGE on schema
- Missing USAGE on catalog
- User belongs to wrong group
- Conflicting explicit DENY

## 4. Recovery Actions

### 4.1 Grant missing privileges
SQL examples:
GRANT SELECT ON TABLE catalog.schema.table TO `group-name`;
GRANT USAGE ON SCHEMA catalog.schema TO `group-name`;
GRANT USAGE ON CATALOG catalog TO `group-name`;

### 4.2 If conflicting DENY privilege exists
SQL:
REVOKE DENY ON TABLE catalog.schema.table FROM `group-name`;

### 4.3 If user must inherit access from a group
- Add user to proper Entra ID group (preferred)
- Sync group via SCIM
- Re-check with SHOW GRANTS

### 4.4 Validate
User runs:
SELECT COUNT(*) FROM catalog.schema.table;

If a job was failing:
- Re-run job manually
- Confirm success

## 5. Escalation Path
If unresolved:
- Escalate to Data Platform (UC admins)
- Escalate to IAM team if SCIM sync is broken
- Escalate to Security if global access model is unclear

## 6. Postmortem
- Root cause found:
- Fix applied:
- Permanent group-based rule needed:
- Runbook updated: Yes/No
