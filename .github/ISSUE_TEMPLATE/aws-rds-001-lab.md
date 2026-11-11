---
name: "AWS-RDS-001 — Create Amazon RDS Database"
about: "Fresh, repeatable execution checklist for the AWS Academy RDS lab (31 original steps)"
title: "[AWS-RDS-001] Lab attempt — YYYY-MM-DD"
labels: []
assignees: []
---

# AWS-RDS-001 — Lab execution checklist

**Permanent runbook:** [AWS-RDS-001](../../runbooks/AWS-RDS-001_Create-Amazon-RDS-Database.md)

> **Security:** This issue is visible to anyone who can access the repository. **Do not paste** real passwords, endpoint hostnames, public IPs, account numbers, screenshots containing secrets, or access tokens. Retrieve the password from the authorized course PDF when needed.

## Session status

- **Course:** NETC 3300 / ITNW 2475 (choose one)
- **Attempt date:** YYYY-MM-DD
- **Progress:** Not started / In progress / Blocked / Submitted
- **Resume at original step:** 1
- **Next action:** Start Lab
- **Issue/blocker (sanitized):** None

## Private worksheet — keep OFF GitHub

Use a local, untracked note or temporary scratchpad. **Do not enter actual values in this issue.**

| Private value | Where to find it |
|---|---|
| Lab account / Region | AWS Console |
| AppServerPublicIP | AWS Details |
| RDS endpoint | RDS → Databases → inventory-db → Connectivity & security |
| Lab password | Authorized lab instructions |
| Submission result / evidence | AWS Academy Grades / submission report |

## Execution checklist — all original steps

**If the AWS form presents fields in a different order, configure them as they appear and use this checklist to verify every required selection before creating the database.**

### Access AWS Academy and Console

- [ ] **Step 1 — Start lab:** Select **Start Lab** in AWS Academy; wait for the timer, green AWS indicator, and AWS Details. To extend time, select Start Lab before the timer expires.
- [ ] **Step 2 — Open console:** Choose the **AWS** link above the lab terminal. Allow pop-ups if blocked.
- [ ] **Step 3 — Arrange workspace:** Keep the lab instructions and AWS Management Console accessible together. **Do not change Region** unless instructed.

### Task 1 — Create Amazon RDS Database

- [ ] **Step 4 — Open RDS:** AWS Console search → **RDS** → **Aurora and RDS**.
- [ ] **Step 5 — Create database:** Choose **Create database**.
- [ ] **Step 6 — Choose engine:** Under **Engine options**, select **MySQL** (not Aurora).
- [ ] **Step 7 — Template and availability:** Under **Templates**, choose **Free tier**. Under **Availability and durability**, verify **Single-AZ DB instance**; select it manually if necessary.
  - [ ] **Explicitly check Single-AZ**; it may not be preselected.
- [ ] **Step 8 — Database settings:** Under **Settings**, set **DB instance identifier** = `inventory-db` and **Master username** = `admin`.
- [ ] **Step 9 — Credentials:** Under **Credentials management**, select **Self managed**. Enter the **lab-provided password** in both password fields. Retrieve the exact value from the authorized course instructions; do not store it in a public issue.
  - [ ] **Self managed** selected; both password fields match the authorized lab value (never paste it here).
- [ ] **Step 10 — Instance class:** Under **Instance configuration**, choose **Burstable classes (includes t classes)** → **db.t3.micro**.
- [ ] **Step 11 — Storage:** Under **Storage**, select **General Purpose SSD (gp2)** and **Allocated storage** = `20` GiB.
  - [ ] Storage type gp2; allocated storage 20 GiB.
- [ ] **Step 12 — Autoscaling:** Expand **Storage autoscaling** and **deselect Enable storage autoscaling**.
  - [ ] Storage autoscaling disabled.
- [ ] **Step 13 — Connectivity:** Under **Connectivity**, select **Lab VPC**; keep the default **DB subnet group**. For **VPC security group (firewall)** choose **Choose existing**, select **DB-SG**, and **remove `default`**. Verify only DB-SG remains. If a DB port field appears, verify **3306** for MySQL; this is an observed console location, not a separate numbered requirement in the source PDF.
  - [ ] Lab VPC; default DB subnet group; **DB-SG only**, `default` removed; verify TCP 3306 if shown.
- [ ] **Step 14 — Monitoring:** Confirm **Database Insights – Standard** is selected under **Monitoring** as described in the lab; record any console differences.
- [ ] **Step 15 — Additional configuration:** Expand **Additional configuration** and set **Initial database name** = `inventory`. Leave other options at lab defaults.
- [ ] **Step 16 — Create and wait:** Choose **Create database**. Wait until `inventory-db` status is **Available** before proceeding. Close Suggested add-ons if prompted.

**STOP & VERIFY — RDS:** DB is **Available**; MySQL, Single-AZ, db.t3.micro, gp2/20 GiB, autoscaling off, Lab VPC, DB-SG only.


### Task 2 — Connect Inventory Application

- [ ] **Step 17 — Get application IP:** In AWS Academy **AWS Details**, copy **AppServerPublicIP** into a private scratchpad (not a public issue).
- [ ] **Step 18 — Open application:** Open a new browser tab and navigate to the lab-provided AppServerPublicIP. Verify the Inventory web app loads.
- [ ] **Step 19 — Open settings:** In the Inventory application, choose **Settings**.
- [ ] **Step 20 — Return to AWS:** Return to the AWS Console **without closing** the Inventory application tab.
- [ ] **Step 21 — Open RDS again:** AWS Console → **Aurora and RDS**.
- [ ] **Step 22 — Find databases:** Left navigation → **Databases**.
- [ ] **Step 23 — Open instance:** Select **inventory-db**.
- [ ] **Step 24 — Copy endpoint:** Open **Connectivity & security** and copy the actual **Endpoint**. Do not use the example hostname in the PDF.
- [ ] **Step 25 — Save connection:** Return to Inventory **Settings**. Enter the actual endpoint; **Database** = `inventory`; **Username** = `admin`; **Password** = lab-provided value. Choose **Save**. The app stores connection data in AWS Secrets Manager.
  - [ ] Verify endpoint is from this run, not a prior attempt.
- [ ] **Step 26 — Test CRUD:** In Inventory, **add**, **edit**, and **delete** inventory records. Confirm the changes appear.
- [ ] **Step 27 — Maintain five records:** Ensure the inventory table has **five or more records** before submitting. Add replacement records if a deletion reduced the count.

**STOP & VERIFY — APP:** Settings saved, CRUD tested, **at least 5 records**.


### Submit and End Lab

- [ ] **Step 28 — Submit:** Return to AWS Academy instructions → **Submit** → confirm **Yes**.
- [ ] **Step 29 — Check grade:** Wait for results. If they do not display after a few minutes, choose **Grades**. The latest submission is the recorded result.
- [ ] **Step 30 — Read report:** Choose **Details** → **View Submission Report**; inspect any unsuccessful checks.
- [ ] **Step 31 — End lab:** When finished and after saving permitted evidence, choose **End Lab** → **Yes**; confirm the success message.

**STOP & VERIFY — SUBMISSION:** Grade checked; detailed report reviewed; lab ended only after saving permitted notes.

## Final validation

- [ ] All 31 original steps checked or a blocker recorded.
- [ ] Database and Inventory application verification passed.
- [ ] At least five records at submission.
- [ ] Grade and submission details reviewed.
- [ ] Sensitive values excluded from this issue.
- [ ] New troubleshooting findings recorded without credentials.

## Troubleshooting notes (sanitized)

- **Step where blocked:**
- **Exact error text (redact account IDs, ARNs, IPs, endpoints, secrets):**
- **What I checked:**
- **What fixed it / not yet resolved:**
- **Resume at step:**

## Outcome

- **Result:** Passed / Needs retry / Blocked
- **Lessons learned:**
- **Next attempt (if needed):**
