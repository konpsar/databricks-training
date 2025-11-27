# Runbook: Cluster Policy Violation (User Cannot Create or Start Cluster)

## 1. Incident Overview
- Title: User cannot create/start a cluster due to policy restrictions
- Detected by: User report / error popup in UI
- Severity: SEV-3 (non-critical but blocks productivity)
- Impact:
  - User blocked from running notebooks
  - Training / development work interrupted

---

## 2. Preconditions / Required Access
Roles needed:
- Workspace Admin
- Cluster Policy Admin

Tools:
- Databricks UI
- Cluster Policies page

---

## 3. Initial Diagnosis

### 3.1 Identify the exact error message
Typical messages:
- "Cluster creation failed due to policy violation"
- "Instance type not allowed by policy"
- "Min workers > Max workers"
- "This Databricks Runtime version is not permitted"
- "This cluster size exceeds allowed limits"

Ask user for:
- Screenshot OR
- Cluster name OR
- Cluster ID

### 3.2 Inspect the Policy attached to the cluster
UI path:
Compute → Create Cluster → Policy dropdown (e.g., “Standard-Restricted”)

Check for:
- Allowed instance families
- Min/max workers
- Allowed runtime versions
- Disabled features (DBFS, Init Scripts, Secret Scopes)

---

## 4. Recovery Actions

### 4.1 If user is missing the correct policy
Assign user to proper group:
Workspace Admin → User → Assign to group (e.g., “data-engineers-policy”)

### 4.2 If policy is too restrictive
Open:
Workspace Admin → Compute → Policies → Select Policy

Modify specific fields, for example:
- Allow more instance types
- Increase max_workers
- Allow job clusters
- Allow interactive clusters

### 4.3 If policy is wrong for user
Change policy for this cluster creation:
User creates cluster and selects correct allowed policy.

### 4.4 If Databricks Runtime version is disallowed
Update policy:
Add allowed DBR versions (e.g., 13.3 LTS, 14.0)

---

## 5. Validation

### 5.1 User tries creating the cluster again
- If successful → issue resolved
- If not → read new error message (often another restriction)

### 5.2 Admin smoke test
Try:
spark.range(10).count()
(if cluster starts successfully)

---

## 6. Escalation Path
Escalate to Data Platform Team if:
- Policy logic is unclear
- Multiple conflicting policies exist
- Policies belong to another team or business unit

---

## 7. Postmortem
- Was policy too restrictive?
- Did we need a new policy for this team?
- Should this user group have a different default policy?
- Update policy documentation or runbook if needed
