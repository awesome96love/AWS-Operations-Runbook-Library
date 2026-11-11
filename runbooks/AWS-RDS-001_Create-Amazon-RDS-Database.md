# AWS RUNBOOK — Creating an Amazon RDS Database

| Field | Value |
|---|---|
| Runbook ID | AWS-RDS-001 |
| Primary service | Amazon RDS (MySQL) |
| Related services | EC2, VPC, security groups, Secrets Manager, IAM |
| Environment | AWS Academy / Vocareum lab |
| Courses | NETC 3300 / ITNW 2475 |
| Tested | November 6, 2026 |
| Status | Completed in training lab; earlier failures documented |
| Source | AWS Academy, *Guided Lab: Creating an Amazon RDS Database* |

> **Training-only procedure.** Values such as the sample DB identifier, database name, instance class, and security group are specific to this AWS Academy lab. Do not reuse its permissive settings or credentials in production. This document paraphrases the source procedure; consult your authorized course materials for the lab password.

## 1. Purpose

Create an RDS MySQL database in the provided VPC, connect the predeployed EC2-hosted Inventory application, and verify database-backed create/update/delete operations.

## 2. Prerequisites

- Active AWS Academy session; lab role and console access initialized.
- Lab VPC, DB subnet group, `DB-SG`, and application EC2 instance provisioned by the lab.
- Browser pop-ups permitted; lab AWS Details panel accessible.
- Keep temporary passwords, public IPs, endpoints, and account IDs **out of GitHub**.

## 3. Architecture and key configuration

EC2 Inventory web application → private-network MySQL RDS database. The application stores connection details using AWS Secrets Manager.

| Setting | Lab-required value |
|---|---|
| DB identifier | `inventory-db` |
| Engine | MySQL |
| Template | Free tier |
| Availability | Single-AZ DB instance |
| Credentials | Self managed; master username `admin`; password from authorized lab PDF |
| Class | `db.t3.micro` |
| Storage | gp2, 20 GiB; autoscaling disabled |
| VPC | Lab VPC |
| DB subnet group | Lab-provided default selection |
| VPC security group | `DB-SG` only; remove `default` |
| Initial database | `inventory` |
| MySQL port | TCP 3306 (verify if shown in current console) |

## 4. Procedure — original AWS lab step mapping

**Console-order note:** The live AWS form may reorder Engine options, Templates, Availability, Credentials, and Additional configuration. Follow the screen's actual section order, but verify every numbered requirement below before choosing Create database. The source PDF does not explicitly instruct changing the port; 3306 is an execution observation.

### Access AWS Academy and Console

#### Original Step 1 — Start lab

Select **Start Lab** in AWS Academy; wait for the timer, green AWS indicator, and AWS Details. To extend time, select Start Lab before the timer expires.

#### Original Step 2 — Open console

Choose the **AWS** link above the lab terminal. Allow pop-ups if blocked.

#### Original Step 3 — Arrange workspace

Keep the lab instructions and AWS Management Console accessible together. **Do not change Region** unless instructed.


### Task 1 — Create Amazon RDS Database

#### Original Step 4 — Open RDS

AWS Console search → **RDS** → **Aurora and RDS**.

#### Original Step 5 — Create database

Choose **Create database**.

#### Original Step 6 — Choose engine

Under **Engine options**, select **MySQL** (not Aurora).

#### Original Step 7 — Template and availability

Under **Templates**, choose **Free tier**. Under **Availability and durability**, verify **Single-AZ DB instance**; select it manually if necessary.

#### Original Step 8 — Database settings

Under **Settings**, set **DB instance identifier** = `inventory-db` and **Master username** = `admin`.

#### Original Step 9 — Credentials

Under **Credentials management**, select **Self managed**. Enter the **lab-provided password** in both password fields. Retrieve the exact value from the authorized course instructions; do not store it in a public issue.

#### Original Step 10 — Instance class

Under **Instance configuration**, choose **Burstable classes (includes t classes)** → **db.t3.micro**.

#### Original Step 11 — Storage

Under **Storage**, select **General Purpose SSD (gp2)** and **Allocated storage** = `20` GiB.

#### Original Step 12 — Autoscaling

Expand **Storage autoscaling** and **deselect Enable storage autoscaling**.

#### Original Step 13 — Connectivity

Under **Connectivity**, select **Lab VPC**; keep the default **DB subnet group**. For **VPC security group (firewall)** choose **Choose existing**, select **DB-SG**, and **remove `default`**. Verify only DB-SG remains. If a DB port field appears, verify **3306** for MySQL; this is an observed console location, not a separate numbered requirement in the source PDF.

#### Original Step 14 — Monitoring

Confirm **Database Insights – Standard** is selected under **Monitoring** as described in the lab; record any console differences.

#### Original Step 15 — Additional configuration

Expand **Additional configuration** and set **Initial database name** = `inventory`. Leave other options at lab defaults.

#### Original Step 16 — Create and wait

Choose **Create database**. Wait until `inventory-db` status is **Available** before proceeding. Close Suggested add-ons if prompted.

**STOP & VERIFY:** Database status is **Available**; engine MySQL; Single-AZ; DB-SG only; endpoint visible.


### Task 2 — Connect Inventory Application

#### Original Step 17 — Get application IP

In AWS Academy **AWS Details**, copy **AppServerPublicIP** into a private scratchpad (not a public issue).

#### Original Step 18 — Open application

Open a new browser tab and navigate to the lab-provided AppServerPublicIP. Verify the Inventory web app loads.

#### Original Step 19 — Open settings

In the Inventory application, choose **Settings**.

#### Original Step 20 — Return to AWS

Return to the AWS Console **without closing** the Inventory application tab.

#### Original Step 21 — Open RDS again

AWS Console → **Aurora and RDS**.

#### Original Step 22 — Find databases

Left navigation → **Databases**.

#### Original Step 23 — Open instance

Select **inventory-db**.

#### Original Step 24 — Copy endpoint

Open **Connectivity & security** and copy the actual **Endpoint**. Do not use the example hostname in the PDF.

#### Original Step 25 — Save connection

Return to Inventory **Settings**. Enter the actual endpoint; **Database** = `inventory`; **Username** = `admin`; **Password** = lab-provided value. Choose **Save**. The app stores connection data in AWS Secrets Manager.

#### Original Step 26 — Test CRUD

In Inventory, **add**, **edit**, and **delete** inventory records. Confirm the changes appear.

#### Original Step 27 — Maintain five records

Ensure the inventory table has **five or more records** before submitting. Add replacement records if a deletion reduced the count.

**STOP & VERIFY:** Settings saved; Inventory app shows at least five records and CRUD operations work.


### Submit and End Lab

#### Original Step 28 — Submit

Return to AWS Academy instructions → **Submit** → confirm **Yes**.

#### Original Step 29 — Check grade

Wait for results. If they do not display after a few minutes, choose **Grades**. The latest submission is the recorded result.

#### Original Step 30 — Read report

Choose **Details** → **View Submission Report**; inspect any unsuccessful checks.

#### Original Step 31 — End lab

When finished and after saving permitted evidence, choose **End Lab** → **Yes**; confirm the success message.

**STOP & VERIFY:** Submission report reviewed and lab ended only after evidence was saved.

## 5. Verification and testing

- [ ] DB instance exists and status is Available.
- [ ] DB configuration matches engine, class, storage, Single-AZ, VPC, subnet group, and DB-SG requirements.
- [ ] Inventory app saves settings and loads records.
- [ ] Add/edit/delete actions succeed; five or more records remain.
- [ ] Grade and detailed submission report reviewed.

## 6. Troubleshooting reference

### RDS-001 — Inventory application connection error

**Observed:** “Check your Database connection details and try again.” During an earlier attempt, disabling AT&T ActiveArmor did not immediately resolve the error; clearing and re-entering application connection settings was followed by a successful connection. **Root cause not confirmed.**

**Check in order:** RDS Available → actual endpoint copied without extra characters → database `inventory` → username `admin` → password from lab → DB-SG membership → re-enter application settings. If browser/network protection is suspected, investigate without assuming that local protection controls EC2-to-RDS traffic. Restore any temporarily disabled protection.

### RDS-002 — IAM AccessDenied on create

**Observed:** `rds:CreateDBInstance` denied because no identity-based policy permitted the action. This is an AWS-side authorization failure, not proof of a browser/cache problem.

**Check:** correct course lab, active/ready session, correct lab-provided console role. If still denied, save the error and escalate to course/lab administrator. Do not try to change IAM policies outside lab permissions.

### Console discrepancies from tested attempts

- **Single-AZ:** Was not automatically selected during a tested attempt; select and verify it explicitly.
- **Engine/template order:** The console layout may not match the PDF sequence.
- **Autoscaling:** Explicitly uncheck storage autoscaling after selecting allocated storage.
- **Security group:** Remove `default`; keep only `DB-SG`.
- **Port 3306:** May appear under additional connectivity settings; verify MySQL port if visible, without changing unrelated defaults.

## 7. Lessons learned

A working EC2 web page does not itself prove RDS connectivity; successful inventory CRUD does. Distinguish AWS IAM authorization failures from connection errors and from browser/network interference. Recheck actual console defaults rather than relying on wording that says a value is automatically chosen.

## 8. Revision history

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-11-06 | Completed Word runbook following lab attempts |
| 1.1 | 2026-11-11 | GitHub Markdown edition; full original-step mapping, public-safe placeholders, verified corrections |

## 9. References

- AWS Academy, *Guided Lab: Creating an Amazon RDS Database* (course-provided PDF; original steps 1–31).
- Personal execution observations and AWS-RDS-001 Word runbook (November 2026).
- AWS documentation: [Amazon RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html).
