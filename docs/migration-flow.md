# Migration Platform - End-to-End Agent Flow

This guide documents the working journey from Dashboard through each agent module in the Jenkins to Azure DevOps migration platform.

## 1) Platform Entry

Open `http://localhost:5173` and click **Enter Platform**.

![01 Platform Entry](images/01-platform-entry.png)

---

## 2) Dashboard

On **Platform Dashboard**, confirm:
- Jenkins master is visible and online
- Quick actions are available (**Start New Migration**, **Gateway Admin**)

![02 Dashboard](images/02-dashboard.png)

---

## 3) Define Scope - Connection and Job Source

Go to **Define Scope** and validate Jenkins master details.

Use:
- **Test All Connections**
- **Upload Manifest** (optional)
- **Browse Jenkins**

![03 Define Scope Overview](images/03-define-scope-overview.png)

After testing, confirm toast: **Connection test complete**.

![04 Connection Test Complete](images/04-connection-test-complete.png)

---

## 4) Define Scope - Job Selection

Click **Browse Jenkins**, select target jobs, then submit selected jobs.

![05 Job Browser](images/05-job-browser.png)

![06 Job Selection Ready](images/06-job-selection-ready.png)

---

## 5) Wave Assignment - Agent 1 Intake

Open **Wave Assignment** and start **Agent 1 Intake**.

![07 Wave Intake Start](images/07-wave-intake-start.png)

After processing, verify:
- Total jobs
- Validated count
- Risk scored
- Wave assigned
- Intake checklist complete

![08 Wave Intake Complete](images/08-wave-intake-complete.png)

![09 Wave Risk Summary](images/09-wave-risk-summary.png)

---

## 6) Wave Plan

Review generated wave plan and approval actions.

![10 Wave Plan](images/10-wave-plan.png)

---

## 7) MCP Gateway (Control Layer)

Go to **MCP Gateway** and confirm tool server health and kill switch state.

### 7.1 Kill Switch Active State (blocked writes)
![11 Gateway Active](images/11-gateway-active.png)

### 7.2 Kill Switch OFF (normal operation)
![12 Gateway Normal](images/12-gateway-normal.png)

---

## 8) Migration Execution

Open **Migration** and verify:
- Session state
- Automation target
- Live activity log

![13 Migration Progress](images/13-migration-progress.png)

---

## 9) YAML Validator (Agent 4)

Open **YAML Validator** and review generated output per job.

Validate:
- PASS status and confidence %
- Config diff (Jenkins vs Azure pipeline YAML)
- Download output with **Download YAML**

![14 Validator Job View A](images/14-validator-job-a.png)

![15 Validator Job View B](images/15-validator-job-b.png)

![16 Validator Job View C](images/16-validator-job-c.png)

---

## Completion Checklist

- [ ] Dashboard visible with live Jenkins master
- [ ] Scope connection test passes
- [ ] Jobs selected and submitted
- [ ] Agent 1 intake completes
- [ ] Wave plan generated
- [ ] Migration progress/logs visible
- [ ] YAML validation returns PASS for selected jobs
- [ ] MCP Gateway is healthy and controllable

---

## Notes

- Store screenshots in `docs/images` using the exact filenames referenced above.
- If your filenames differ, update the image paths in this file.
