# AWS-RDS-002 — Migrating a Database to Amazon RDS

> **AWS Academy Cloud Architecting | Module 5 Challenge Lab**  
> **Status:** Completed and validated (November 7, 2026) | **Environment:** AWS Academy temporary lab  
> **Services:** Amazon RDS (MariaDB), EC2, VPC, Secrets Manager, Systems Manager

> [!IMPORTANT]
> This is an educational lab runbook, not a production migration procedure. Do **not** commit real passwords, secret values, account IDs, or current infrastructure addresses to a public repository. Use the lab instructions to obtain the temporary master password.

## Before starting — Open a private Notepad scratchpad

- [ ] Open **Windows Notepad** and create a temporary note **outside the GitHub repository**.


```text
LOCAL DB PASSWORD (/cafe/dbPassword):
RDS MASTER PASSWORD (from lab instructions):
RDS ENDPOINT (copy at Step 34):
CAFE SERVER PUBLIC IP (copy at Step 6):
```

**Passwords:** Use the **local DB password** for inspection/export; use the **RDS master password** for RDS login/import and the updated `/cafe/dbPassword`.

**Endpoint:** Copy it to Notepad at Step 34. Reuse it for terminal commands and `/cafe/dbUrl` (no protocol or port).

**Security:** Keep this scratchpad private and temporary. Do not save it in the repository, screenshots, or GitHub Issues. Delete the note when the lab is finished.

> **NETC 3300 submission:** Capture the required screenshots at **Steps 9 and 48**. Save them to your coursework submission document, not the public runbook repository.


---

## 1. Purpose and Learning Objectives

Migrate the café application’s MariaDB database from an Amazon EC2 instance to Amazon RDS. Export the original data with mysqldump, establish private database connectivity, import the data, update Secrets Manager, and confirm that the application continues to operate after the EC2-hosted database is stopped.  
• Create a MariaDB RDS instance with the lab-prescribed configuration.  
• Inspect the existing café application and database.  
• Export the database to a SQL dump and import it into RDS.  
• Restrict database access to the café EC2 server security group.  
• Update application connection secrets without modifying PHP code.  
• Verify new and historical orders while the original database service is stopped.  

## 2. Prerequisites and Value Handling

• Active AWS Academy lab session and AWS Management Console access.  
• Preprovisioned CafeServer EC2 instance, Lab VPC, lab-db-subnet-group, dbSG and CafeSG.  
• AWS Systems Manager Session Manager access to CafeServer.  
• Existing Secrets Manager secrets under /cafe/.  
• Use the current lab’s public EC2 IP, RDS endpoint, subnet IDs, and security group IDs. These are environment-specific and change between attempts.  
• The example master password in this lab is a classroom-only credential; do not reuse it in production or commit live secrets to GitHub.  

## 3. Configuration and Procedures

### How to use command blocks in VS Code

Use **Markdown: Open Preview to the Side** (`Ctrl+K`, then `V`). In a preview that supports code-block copying, use its copy icon; otherwise select and copy the complete fenced block from the editor.

**Copy one block at a time.** **Linux shell:** `$` or `#` (either `ec2-user` or `root` is fine when permissions allow). **SQL client:** `MariaDB [...]>`. Never paste a Linux `sudo`, `mysql`, or `mysqldump` command into the MariaDB prompt. Password prompts require manual entry; typed passwords will not appear.

Check **STOP & VERIFY** before continuing. Replace placeholders; copy only commands.

> **Security:** Keep live passwords out of Markdown and Git. Use the lab's current secret values and RDS endpoint.




### 3.1 Start Lab and Open AWS Console — Original Steps 1–3

- [ ] 1. Select Start Lab. Wait for the AWS indicator to turn green; AWS Details then becomes available.
- [ ] 2. Select AWS to open the AWS Management Console. Allow pop-ups if the browser blocks the new tab.
- [ ] 3. Arrange the console alongside the instructions. Do not change the Region unless the lab directs you to.

### 3.2 Create the MariaDB RDS Instance — Original Step 4

**Navigate:** Aurora and RDS → Databases → Create database.

**Step 4 — Configure from top to bottom:**

- [ ] **Engine options → Engine type:** MariaDB
- [ ] **Choose database creation method:** Full configuration (current Console label)
- [ ] **Templates:** Dev/Test
- [ ] **Settings → Engine version:** MariaDB 10.6.25
- [ ] **DB instance identifier:** CafeDatabase (displayed later as cafedatabase)
- [ ] **Master username:** admin
- [ ] **Credentials management:** Self managed — explicitly select
- [ ] **Master password and confirmation:** `<RDS_MASTER_PASSWORD_FROM_LAB>` (lab-only)
- [ ] **Instance configuration → DB instance class:** Burstable classes (includes t classes)
- [ ] **Instance type:** db.t3.micro
- [ ] **Storage → Storage type:** General Purpose SSD (gp2), not gp3
- [ ] **Allocated storage:** 20 GiB, not 200 GiB
- [ ] **Additional storage configuration:** Expand; UNCHECK Enable storage autoscaling
- [ ] **Availability & durability:** Do not create a standby instance (Single-AZ)
- [ ] **Connectivity → Compute resource:** Don’t connect to an EC2 compute resource
- [ ] **Network type:** IPv4 (leave unchanged)
- [ ] **VPC:** Lab VPC (not Default VPC)
- [ ] **DB subnet group:** lab-db-subnet-group (not default)
- [ ] **Public access:** No
- [ ] **VPC security group:** Choose existing → dbSG; REMOVE default
- [ ] **Availability Zone:** First listed zone ending in a (tested us-east-1a)
- [ ] **RDS Proxy:** Leave unchecked
- [ ] **Connectivity → Additional configuration:** Expand; database port 3306
- [ ] **Monitoring → Enhanced Monitoring:** Expand Additional monitoring settings; UNCHECK Enable Enhanced monitoring
- [ ] **Final action:** Review and select Create database

**VERIFY:** Database appears in Databases. Continue with Step 5 while it provisions.  

### 3.3 Examine the Café Application — Original Steps 5–10

- [ ] 5. Open EC2 → Instances; locate the running CafeServer instance.
- [ ] 6. Copy its current Public IPv4 address into your private Notepad note and construct `http://<public-ip-address>/cafe`.
- [ ] 7. Open the café URL in a browser tab.
- [ ] 8. Select Menu.
- [ ] 9. Set at least one menu item quantity to 1 and select Submit Order; confirm the order confirmation page. **📸 NETC 3300: Take the required Step 9 screenshot now, before navigating away.**
- [ ] 10. Select Order History and verify the new order and historical orders are displayed.

### 3.4 Open EC2 Session Manager — Original Steps 11–15

- [ ] 11. In EC2 → Instances, select CafeServer.
- [ ] 12. Select Connect from the instance connection menu.
- [ ] 13. Choose Session Manager.
- [ ] 14. Select Connect and verify a browser terminal opens.
- [ ] **15a–15d.** At the **Linux shell**, switch to `ec2-user`, then enter its home directory. Paste this **single block**:

```bash
sudo su
su ec2-user
whoami
cd /home/ec2-user/
pwd
```

**VERIFY:** `pwd` shows `/home/ec2-user`. Use the Linux shell (`$` or `#`), not `MariaDB >`.

### 3.5 Inspect Local MariaDB and Secrets — Original Steps 16–25

- [ ] **16.** Check the original MariaDB service and client in one **Linux shell** block:

```bash
service mariadb status
mysql --version
```

**STOP & VERIFY:** The local MariaDB service is running and the client is installed.

- [ ] 17. Open AWS Secrets Manager in another Console tab.
- [ ] 18. In the navigation pane select Secrets.
- [ ] 19. Open the /cafe/dbPassword secret.
- [ ] 20. Select Retrieve secret value and record `dbPassword` in your **private Notepad note**. This is the original local database password, NOT the RDS master password.
- [ ] 21. Back in the EC2 terminal, connect to the local database:

```bash
mysql -u root -p
```

- [ ] 22. Paste the local database password when prompted. Confirm the MariaDB prompt appears.
- [ ] **23–25.** At the **MariaDB SQL prompt**, paste the following SQL as **one block** to inspect the original data, then exit:

```sql
SHOW DATABASES;
USE cafe_db;
SHOW TABLES;
SELECT * FROM `order`;
SELECT * FROM `order_item`;
EXIT;
```

**STOP & VERIFY:** Confirm the `cafe_db` database, expected tables and existing orders are visible before proceeding. After `EXIT;`, the prompt must be the **Linux shell** again. The backticks around `order` are required because it is a SQL keyword.

### 3.6 Export the EC2 Database — Original Steps 26–29

- [ ] **26.** At the Linux shell (`$` or `#`), go to the dump directory:

```bash
cd /home/ec2-user/
```

- [ ] **27.** Export; enter the **local DB password** when prompted:

```bash
mysqldump --databases cafe_db -u root -p > CafeDbDump.sql
```

**VERIFY:** Returns to the Linux prompt without an error. `ERROR 1045`? Recheck the local password.

- [ ] **28–29. Verify export.** Only after export succeeds, paste this **single Linux shell block**:

```bash
ls -lh CafeDbDump.sql
grep -m 5 -E 'CREATE TABLE|INSERT INTO' CafeDbDump.sql
tail -n 5 CafeDbDump.sql
```

**VERIFY:** `ls` shows a nonempty file; `grep` shows `CREATE TABLE` / `INSERT INTO`; `tail` shows `Dump completed on ...`.


### 3.7 Check RDS Availability and Answer Questions — Original Steps 30–33

**Question reference (verify against the current lab):**

| Question | Answer / how to verify |
| --- | --- |
| Q1: Most specific location | Availability Zone level (tested us-east-1a); RDS Summary → Region & AZ |
| Q2: Public IPv4 assigned? | No; Connectivity & security → Publicly accessible: No |
| Q3: Subnet Name tag | Private Subnet 1 in tested lab; verify RDS AZ against VPC → Subnets → Name tag |
| Q4: Security group rule count | Inspect the current `dbSG` rules; answer using the lab's requested scope. |

- [ ] 30. Return to Aurora and RDS → Databases; verify cafedatabase status is Available.
- [ ] 31. At the top of the lab instructions, select AWS Details.
- [ ] 32. Open Access the multiple choice questions.
- [ ] 33. Answer the four questions using the current RDS and VPC console evidence.

### 3.8 Establish EC2-to-RDS Connectivity — Original Steps 34–36

- [ ] **34a.** RDS → Databases → `cafedatabase` → Connectivity & security. Copy the **endpoint** into private Notepad.
- [ ] **34b.** Open the linked **dbSG** in EC2 → Security Groups.
- [ ] **34c.** Inbound rules → Edit inbound rules → Add rule.
- [ ] **34d.** Set **MySQL/Aurora | TCP | 3306 | Source: CafeSG** (the security group attached to CafeServer).
- [ ] **34e.** Save rules. Do **not** use `0.0.0.0/0`.
- [ ] **34f.** In the **Linux shell**, set your endpoint (replace the text inside the quotes):

```bash
RDS_ENDPOINT="paste-current-rds-endpoint-here"
```

- [ ] **34g.** Test the port:

```bash
nmap -Pn -p 3306 "$RDS_ENDPOINT"
```

**STOP & VERIFY:** `3306/tcp open mysql`. If `filtered`, recheck **dbSG inbound source = CafeSG** before proceeding.

- [ ] **35.** Connect to RDS from the Linux shell:

```bash
mysql -u admin -p --host "$RDS_ENDPOINT"
```

Enter the **RDS master password**.

- [ ] **36.** At the **MariaDB SQL prompt**, check the empty RDS instance and return to Linux:

```sql
SHOW DATABASES;
EXIT;
```

**VERIFY:** `cafe_db` is **not** present yet. Terminal returns to `$` or `#`.

### 3.9 Import and Verify Data in RDS — Original Steps 37–42

- [ ] **37–38. Import.** At the Linux shell (`$` or `#`), run this block. Keep the same terminal session so `$RDS_ENDPOINT` is set:

```bash
cd /home/ec2-user/
test -s CafeDbDump.sql && mysql -u admin -p --host "$RDS_ENDPOINT" < CafeDbDump.sql
```

**VERIFY:** Enter the **RDS master password**. The command returns without an error. If nothing happens, check `CafeDbDump.sql` exists; if you reopened the terminal, set `RDS_ENDPOINT` again.

- [ ] **39–40. Reconnect.** At the **Linux shell**:

```bash
mysql -u admin -p --host "$RDS_ENDPOINT"
```

Enter the RDS master password.

- [ ] **41–42. Verify and exit.** At the **RDS MariaDB SQL prompt**, paste **one SQL block**:

```sql
SHOW DATABASES;
USE cafe_db;
SHOW TABLES;
SELECT COUNT(*) AS order_count FROM `order`;
SELECT * FROM `order` ORDER BY order_number DESC LIMIT 5;
EXIT;
```

**VERIFY:** `cafe_db` and all four tables exist; order count is at least 24. The latest pre-migration order appears. `EXIT;` returns to Linux.


### 3.10 Repoint Café Application to RDS — Original Steps 43–45

- [ ] **43.** Open **AWS Secrets Manager → Secrets**.
- [ ] **44.** Locate the seven `/cafe/` secrets.
- [ ] **45a.** `/cafe/dbPassword` → update to the **RDS master password**.
- [ ] **45b.** `/cafe/dbUser` → update to `admin`.
- [ ] **45c.** `/cafe/dbUrl` → update to the **current RDS endpoint** (no `https://` or `:3306`).
- [ ] **45d.** Confirm `/cafe/dbName` remains unchanged.

**For each update:** Open secret → Retrieve secret value → Edit → Save.

**Leave unchanged:** `/cafe/currency`, `/cafe/timeZone`, `/cafe/showServerInfo`.

**STOP & VERIFY:** All three connection secrets saved.

### 3.11 Stop Local MariaDB and Validate Café — Original Steps 46–48

- [ ] 46. At the **Linux shell** (not inside MariaDB), stop the original service and immediately verify its status:

```bash
sudo service mariadb stop
sudo service mariadb status
```

**VERIFY:** Local MariaDB reports `stopped` or `inactive`.

- [ ] 47. Open http://<public-ip>/cafe/menu.php in a browser and place a new order. Confirm the application still works while local MariaDB is stopped.
- [ ] 48. Select Order History; verify the newly placed order and all earlier orders remain available. **📸 NETC 3300: Take the required Step 48 screenshot now, before navigating away.**

## 4. Verification and Testing

• RDS status: Available; MariaDB engine, db.t3.micro, correct Availability Zone.  
• Network: dbSG inbound MYSQL/Aurora TCP 3306 sourced from CafeSG; nmap confirms 3306/tcp open mysql.  
• Database: RDS login succeeds; cafe_db and all four application tables appear; migrated orders are present.  
• Application: local MariaDB stopped; menu and new-order workflow still work; Order History retains old and new orders.  
• Grading: user reported full marks after completing the challenge lab.  

## 5. Quick Troubleshooting

| Symptom / observation | Investigation, resolution, verification |
| --- | --- |
| mysqldump: ERROR 1045 / Access denied | Check the original `/cafe/dbPassword` secret; distinguish it from the RDS master password. A mistyped character caused a failed attempt; retry carefully without changing the secret. |
| `->` continuation prompt in MariaDB | You are inside the SQL client or have an unfinished SQL statement. Finish/cancel the statement, then use `EXIT;` to return to the Linux shell before running `sudo service mariadb stop`. |
| mysqldump: Permission denied | Observed as ssm-user; switch to ec2-user and /home/ec2-user; rerun export; verify with ls. |
| RDS port inaccessible | Inspect dbSG inbound rules; allow MYSQL/Aurora TCP 3306 from CafeSG only; verify with nmap -Pn <rds-endpoint>. |
| mysql: Unknown suffix “n” for port | nmap -Pn was accidentally appended to mysql --host. Run separate nmap and mysql commands. |
| RDS login fails | Confirm correct current endpoint, username admin and RDS master password; do not use old local DB secret. |
| cafe_db absent before import | Expected at Step 35. Import SQL dump in Step 37 and recheck. |
| Café fails after local DB stopped | Check all three Secrets Manager values and current RDS endpoint; verify dbSG rule and RDS connection. |
| Console order differs from PDF | Use tested top-to-bottom configuration order in §3.2; expand Additional storage configuration, Connectivity Additional configuration and Additional monitoring settings. |


## 6. Lab Completion and Submission — Original Steps 49–52

- [ ] 49. At the top of the AWS Academy lab instructions, select Submit; when prompted, select Yes.
- [ ] 50. After a few minutes, select Grades. You may resubmit; only the last submission score is retained.
- [ ] 51. Select Submission Report to review detailed feedback and resolve any remaining issues.
- [ ] 52. After saving notes/screenshots and confirming grading, select End Lab → Yes; wait for the “Ended AWS Lab Successfully” message.

## 7. Revision History

| Version / date | Change |
| --- | --- |
| 1.3 — 2026-11-11 | Streamlined execution notes, corrected export and connectivity flow, removed duplicate SQL exit, added per-secret checkboxes, and separated maintenance commentary. |
| 1.2 — 2026-11-11 | Replaced oversized worksheet with private Notepad setup, highlighted endpoint reuse and NETC 3300 screenshot checkpoints (Steps 9 and 48). |
| 1.1 — 2026-11-11 | Grouped safe Linux and SQL command blocks for fewer copy/paste operations; added STOP & VERIFY checkpoints, shell/SQL prompt guidance, targeted dump inspection, and troubleshooting from repeat lab. |
| 1.0 — 2026-11-07 | Initial full runbook: original steps 1–52, tested Console ordering, expanded Tips 1–8, actual troubleshooting and successful verification. |


## 9. References

• AWS Academy Cloud Architecting — Module 5 Challenge Lab: Migrating a Database to Amazon RDS (original lab PDF).  
• AWS Academy live challenge tips (Steps 34 and 45), captured November 7, 2026.  
• AWS Management Console — Amazon RDS, EC2 Security Groups, VPC, AWS Secrets Manager and Systems Manager Session Manager.  
• AWS-EFS-001_Create-Mount-Monitor-EFS.docx — document structure reference.  


## Execution sign-off

- [ ] All original steps 1–52 completed
- [ ] RDS endpoint and port verified
- [ ] Imported tables and historical orders verified
- [ ] Local MariaDB stopped and café still works
- [ ] New order visible in Order History
- [ ] Submission report reviewed and grade recorded
- [ ] Lab ended after saving permitted notes

**Runbook ID:** AWS-RDS-002 · **Version:** 1.3 · **Reference:** Original AWS Academy lab PDF and observed November 2026 Console workflow
