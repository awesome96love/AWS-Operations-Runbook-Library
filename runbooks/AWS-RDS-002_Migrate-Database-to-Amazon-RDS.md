# AWS-RDS-002 — Migrating a Database to Amazon RDS
**Lab execution checklist:** [GitHub Issue #1](https://github.com/awesome96love/AWS-Operations-Runbook-Library/issues/1)
> **AWS Academy Cloud Architecting | Module 5 Challenge Lab**  
> **Status:** Completed and validated (November 7, 2026) | **Environment:** AWS Academy temporary lab  
> **Services:** Amazon RDS (MariaDB), EC2, VPC, Secrets Manager, Systems Manager

> [!IMPORTANT]
> This is an educational lab runbook, not a production migration procedure. Do **not** commit real passwords, secret values, account IDs, or current infrastructure addresses to a public repository. Use the lab instructions to obtain the temporary master password.

## Lab execution worksheet

Copy this table into your private notes for each attempt. Avoid saving secrets in GitHub.

| Value | Fill in during lab |
|---|---|
| Date / attempt | `__________` |
| AWS Region | `__________` |
| CafeServer public IPv4 | `__________` |
| CafeServer private IPv4 | `__________` |
| CafeServer security group / ID | `CafeSG / __________` |
| RDS instance identifier | `cafedatabase` |
| RDS endpoint | `__________` |
| RDS security group / ID | `dbSG / __________` |
| RDS Availability Zone | `__________` |
| RDS subnet Name tag | `__________` |
| Dump file path | `/home/ec2-user/CafeDbDump.sql` |
| Before-migration order number | `__________` |
| After-migration order number | `__________` |
| Final lab grade | `__________` |

> **Interactive checklist note:** GitHub Markdown task boxes in a repository file are edited in Markdown; for click-to-complete tracking, copy the checklist into a GitHub Issue.

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


### 3.1 Start Lab and Open AWS Console — Original Steps 1–3

- [ ] 1. Select Start Lab. Wait for the AWS indicator to turn green; AWS Details then becomes available.
- [ ] 2. Select AWS to open the AWS Management Console. Allow pop-ups if the browser blocks the new tab.
- [ ] 3. Arrange the console alongside the instructions. Do not change the Region unless the lab directs you to.

### 3.2 Create the MariaDB RDS Instance — Original Step 4

**Console-order checklist (original lab Step 4):**

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
- [ ] **Connectivity → Additional configuration:** Expand; database port 3306
- [ ] **RDS Proxy:** Leave unchecked
- [ ] **Monitoring → Enhanced Monitoring:** Expand Additional monitoring settings; UNCHECK Enable Enhanced monitoring
- [ ] **Final action:** Review and select Create database

Navigate: Aurora and RDS → Databases → Create database. The following table follows the tested November 2026 Console order rather than the order in the original PDF.  
Important: Console defaults differed from the lab: Standard class, gp3, 200 GiB, default VPC/subnet group/security group, managed credentials and Enhanced Monitoring were shown initially. Select or deselect every lab-required value explicitly. The exact position of monitoring and connectivity options can vary.  
Verification: The RDS database appears in Databases. Creation may take several minutes; continue with Step 5 while it provisions.  

### 3.3 Examine the Café Application — Original Steps 5–10

- [ ] 5. Open EC2 → Instances; locate the running CafeServer instance.
- [ ] 6. Copy its current Public IPv4 address and construct http://<public-ip-address>/cafe.
- [ ] 7. Open the café URL in a browser tab.
- [ ] 8. Select Menu.
- [ ] 9. Set at least one menu item quantity to 1 and select Submit Order; confirm the order confirmation page.
- [ ] 10. Select Order History and verify the new order and historical orders are displayed.

### 3.4 Open EC2 Session Manager — Original Steps 11–15

- [ ] 11. In EC2 → Instances, select CafeServer.
- [ ] 12. Select Connect from the instance connection menu.
- [ ] 13. Choose Session Manager.
- [ ] 14. Select Connect and verify a browser terminal opens.
- [ ] 15. Enter the following commands separately to switch to ec2-user and its writable home directory:

```bash
sudo su
```


```bash
su ec2-user
```


```bash
whoami
```


```bash
cd /home/ec2-user/
```

Verification: whoami returns ec2-user. The current working directory is /home/ec2-user/. Keep this terminal open for the remainder of the lab.  

### 3.5 Inspect Local MariaDB and Secrets — Original Steps 16–25

- [ ] 16. In the EC2 terminal, check MariaDB service status and client version:

```bash
service mariadb status
```


```bash
mysql --version
```

- [ ] 17. Open AWS Secrets Manager in another Console tab.
- [ ] 18. In the navigation pane select Secrets.
- [ ] 19. Open the /cafe/dbPassword secret.
- [ ] 20. Select Retrieve secret value and copy the dbPassword value. This is the original local database password, NOT the RDS master password.
- [ ] 21. Back in the EC2 terminal, connect to the local database:

```bash
mysql -u root -p
```

- [ ] 22. Paste the local database password when prompted. Confirm the MariaDB prompt appears.
- [ ] 23. Inspect databases, tables, and orders with these SQL commands:

```sql
show databases;
```


```sql
use cafe_db;
```


```sql
show tables;
```


```sql
select * from `order`;
```

- [ ] 24. Inspect individual order line items:

```sql
select * from `order_item`;
```

- [ ] 25. Exit the local SQL client:

```sql
exit;
```


### 3.6 Export the EC2 Database — Original Steps 26–29

- [ ] 26. At the Linux shell as ec2-user in /home/ec2-user/, create the SQL dump:

```bash
mysqldump --databases cafe_db -u root -p > CafeDbDump.sql
```

- [ ] 27. When prompted, enter the original database password from /cafe/dbPassword.
- [ ] 28. Confirm the file exists:

```bash
ls
```

- [ ] 29. Inspect the SQL dump; expect SQL table definitions and INSERT statements:

```bash
cat CafeDbDump.sql
```

Troubleshooting from first attempt: bash: CafeDbDump.sql: Permission denied occurred when exporting as ssm-user from a nonwritable location. Working correction: sudo su → su ec2-user → cd /home/ec2-user/ → rerun mysqldump. Second attempt completed without dump errors. A separate bash “event not found” occurred after an attempted command containing an exclamation mark; this was not the database export failure.  

### 3.7 Check RDS Availability and Answer Questions — Original Steps 30–33

**Question reference (verify against the current lab):**

| Question | Answer / how to verify |
| --- | --- |
| Q1: Most specific location | Availability Zone level (tested us-east-1a); RDS Summary → Region & AZ |
| Q2: Public IPv4 assigned? | No; Connectivity & security → Publicly accessible: No |
| Q3: Subnet Name tag | Private Subnet 1 in tested lab; verify RDS AZ against VPC → Subnets → Name tag |
| Q4: Number of security group rules | Question depends on timing and scope: tested answer selection was 1 after adding the single inbound MySQL rule. Before Step 34, dbSG showed 0 inbound and 1 outbound rule. Inspect current rule counts rather than assuming. |

- [ ] 30. Return to Aurora and RDS → Databases; verify cafedatabase status is Available.
- [ ] 31. At the top of the lab instructions, select AWS Details.
- [ ] 32. Open Access the multiple choice questions.
- [ ] 33. Answer the four questions using the current RDS and VPC console evidence.

### 3.8 Establish EC2-to-RDS Connectivity — Original Steps 34–36

Step 34 is an open-ended challenge. The live lab expands Tips 1–4; the PDF does not contain their expanded text. The procedure below integrates the tested tips.  
- [ ] 34. In Aurora and RDS → Databases → cafedatabase → Connectivity & security, expand Additional configurations. Copy the current RDS endpoint and note port 3306. Select the linked dbSG security group, opening EC2 → Security Groups. On Inbound rules, select Edit inbound rules → Add rule. Set Type to MYSQL/Aurora (TCP 3306), Source to Custom, then choose the security group attached to CafeServer (CafeSG). Save rules. Do not use Anywhere-IPv4 or 0.0.0.0/0.
Tip 1: Connect with mysql -u admin -p --host <rds-endpoint>; enter the RDS master password, not the local database secret. Tip 2: Correct credentials alone are insufficient until dbSG permits inbound TCP 3306. Tip 3: Use CafeSG as the rule source rather than opening port 3306 to all IP addresses. Tip 4: Verify the port from the EC2 Session Manager terminal:  

```bash
nmap -Pn <rds-endpoint>
```

Expected: 3306/tcp open mysql. Tested endpoint example: `<rds-endpoint>` (temporary; copy a fresh endpoint for every lab).  

```bash
mysql -u admin -p --host <rds-endpoint>
```

- [ ] 35. At the RDS MariaDB prompt, list databases. The cafe_db database should NOT yet exist:

```sql
show databases;
```

- [ ] 36. Disconnect from the RDS SQL client:

```sql
exit;
```

Observed troubleshooting: Entering “nmap -Pn” after mysql --host caused a mysql argument parsing error. Run nmap and mysql as separate commands; do not append nmap to the host argument.  

### 3.9 Import and Verify Data in RDS — Original Steps 37–42

- [ ] 37. From the EC2 Linux shell in the directory containing CafeDbDump.sql, import the SQL dump:

```bash
mysql -u admin -p --host <rds-endpoint> < CafeDbDump.sql
```

- [ ] 38. Enter the RDS master password when prompted. No error output generally indicates success.
- [ ] 39. Reconnect to RDS:

```bash
mysql -u admin -p --host <rds-endpoint>
```

- [ ] 40. Enter the RDS master password.
- [ ] 41. Verify the imported database, tables, and orders:

```sql
show databases;
```


```sql
use cafe_db;
```


```sql
show tables;
```


```sql
select * from `order`;
```

Verification: cafe_db exists; tables include order, order_item, product and product_group. Lab expects at least 24 orders; tested import showed 25 orders, including a new order placed before migration.  
- [ ] 42. Exit the RDS SQL client:

```sql
exit;
```


### 3.10 Repoint Café Application to RDS — Original Steps 43–45

**Secrets to update (Step 45):**

| Secret | Required value |
| --- | --- |
| /cafe/dbUrl | <rds-endpoint> (the current RDS instance endpoint, no protocol prefix) |
| /cafe/dbUser | admin |
| /cafe/dbPassword | `<RDS_MASTER_PASSWORD_FROM_LAB>` (lab-only RDS master password) |
| /cafe/dbName | Leave unchanged |
| /cafe/currency | Leave unchanged |
| /cafe/timeZone | Leave unchanged |
| /cafe/showServerInfo | Leave unchanged |

- [ ] 43. Return to the AWS Secrets Manager Console tab.
- [ ] 44. Select Secrets and locate the seven /cafe/ secrets.
- [ ] 45. For each of the three secrets below: open the secret → Retrieve secret value → Edit → replace the value → Save. Update them individually; leave the other four secrets unchanged.
Expanded lab Tips 5–8: Test using http://<public-ip>/cafe/menu.php; no PHP edits or additional network changes are needed after successful Step 34; currency, dbName, timeZone and showServerInfo stay unchanged; dbUrl must equal the RDS endpoint.  

### 3.11 Stop Local MariaDB and Validate Café — Original Steps 46–48

- [ ] 46. In EC2 Session Manager at the Linux shell, stop the original MariaDB service:

```bash
sudo service mariadb stop
```

- [ ] 47. Open http://<public-ip>/cafe/menu.php in a browser and place a new order. Confirm the application still works while local MariaDB is stopped.
- [ ] 48. Select Order History; verify the newly placed order and all earlier orders remain available.
Tested result: Order #26 ($367.50) appeared after migration; Order #25 ($1.00) and older records were preserved. This strongly supports successful application use of the RDS database.  

## 4. Verification and Testing

• RDS status: Available; MariaDB engine, db.t3.micro, correct Availability Zone.  
• Network: dbSG inbound MYSQL/Aurora TCP 3306 sourced from CafeSG; nmap confirms 3306/tcp open mysql.  
• Database: RDS login succeeds; cafe_db and all four application tables appear; migrated orders are present.  
• Application: local MariaDB stopped; menu and new-order workflow still work; Order History retains old and new orders.  
• Grading: user reported full marks after completing the challenge lab.  

## 5. Troubleshooting Reference

| Symptom / observation | Investigation, resolution, verification |
| --- | --- |
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

## 7. Lessons Learned

• Organizing configurations in visible Console order avoids backtracking between engine, template, credentials, storage, connectivity and monitoring.  
• Explicitly disabling storage autoscaling and Enhanced Monitoring prevents silently retaining unsuitable defaults.  
• A security group reference (CafeSG) limits RDS access to the café server rather than opening port 3306 to the internet.  
• Export redirection (>) writes as the current Linux user; the working directory and permissions matter.  
• Secrets Manager decouples application connection settings from PHP source code.  
• Stopping the original database and placing a new order provides an effective end-to-end migration test.  

## 8. Revision History

| Version / date | Change |
| --- | --- |
| 1.0 — 2026-11-07 | Initial full runbook: original steps 1–52, tested Console ordering, expanded Tips 1–8, actual troubleshooting and successful verification. |


## 9. References

• AWS Academy Cloud Architecting — Module 5 Challenge Lab: Migrating a Database to Amazon RDS (original lab PDF).  
• AWS Academy live challenge tips (Steps 34 and 45), captured November 7, 2026.  
• AWS Management Console — Amazon RDS, EC2 Security Groups, VPC, AWS Secrets Manager and Systems Manager Session Manager.  
• AWS-EFS-001_Create-Mount-Monitor-EFS.docx — document structure reference.  

---

## Optional command convenience

At the EC2 Linux shell, set the endpoint once per lab attempt:

```bash
RDS_ENDPOINT="paste-current-rds-endpoint-here"
nmap -Pn "$RDS_ENDPOINT"
mysql -u admin -p --host "$RDS_ENDPOINT"
```

For import (run from `/home/ec2-user`):

```bash
mysql -u admin -p --host "$RDS_ENDPOINT" < CafeDbDump.sql
```

**Never** put a password directly in these commands or commit it to GitHub.

## Execution sign-off

- [ ] All original steps 1–52 completed
- [ ] RDS endpoint and port verified
- [ ] Imported tables and historical orders verified
- [ ] Local MariaDB stopped and café still works
- [ ] New order visible in Order History
- [ ] Submission report reviewed and grade recorded
- [ ] Lab ended after saving permitted notes

**Runbook ID:** AWS-RDS-002 · **Version:** 1.0 · **Reference:** Original AWS Academy lab PDF and observed November 2026 Console workflow
