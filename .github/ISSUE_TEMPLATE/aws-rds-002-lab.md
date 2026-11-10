---
name: AWS-RDS-002 Lab Execution
about: Reusable checklist for migrating the café database to Amazon RDS.
title: 'Lab Execution — AWS-RDS-002 — Database Migration'
---
# AWS-RDS-002 — Lab execution checklist

**Runbook:** [AWS-RDS-002 — Migrating a Database to Amazon RDS](../blob/main/runbooks/AWS-RDS-002_Migrate-Database-to-Amazon-RDS.md)  
**Purpose:** Work top-to-bottom. Check each item only after completing it. Keep changing IPs, endpoints, IDs, and all passwords in **private notes**, not this public Issue.  
**Resume here:** Update this line in the Issue whenever you stop: `Next step: ___`.

> [!IMPORTANT]
> This repository is public. **Do not paste passwords, secret values, account IDs, live IP addresses, or RDS endpoints into this Issue or its comments.** Use the runbook's private worksheet. Commands below use placeholders.

## 1 — Start the lab (original steps 1–3)
- [ ] **1.** Select **Start Lab**; wait for the green AWS indicator.
- [ ] **2.** Select **AWS** to open the AWS Console; allow the pop-up if needed.
- [ ] **3.** Arrange the lab instructions and Console side by side; keep the assigned Region.

## 2 — Create RDS (original step 4; actual Console order)
Open **Aurora and RDS → Databases → Create database**.
- [ ] **4a.** Engine type → **MariaDB**.
- [ ] **4b.** Creation method → **Full configuration**.
- [ ] **4c.** Template → **Dev/Test**.
- [ ] **4d.** Engine version → **MariaDB 10.6.25**.
- [ ] **4e.** DB instance identifier → **CafeDatabase**.
- [ ] **4f.** Master username → **admin**.
- [ ] **4g.** Credentials management → **Self managed** (explicitly select).
- [ ] **4h.** Enter and confirm the **lab-provided RDS master password**; record it only in private notes.
- [ ] **4i.** Instance class → **Burstable classes (t classes)**; instance type → **db.t3.micro**.
- [ ] **4j.** Storage type → **General Purpose SSD (gp2)**; allocated storage → **20 GiB**.
- [ ] **4k.** Expand **Additional storage configuration** → **uncheck Enable storage autoscaling**.
- [ ] **4l.** Availability → **Do not create a standby instance** (Single-AZ).
- [ ] **4m.** Compute resource → **Don't connect to an EC2 compute resource**; network type → **IPv4**.
- [ ] **4n.** VPC → **Lab VPC**; DB subnet group → **lab-db-subnet-group**.
- [ ] **4o.** Public access → **No**.
- [ ] **4p.** VPC security group → **Choose existing**; select **dbSG** and **remove default**.
- [ ] **4q.** Availability Zone → first listed zone ending in **a** (verify the lab's Region).
- [ ] **4r.** Expand **Connectivity → Additional configuration** → database port **3306**.
- [ ] **4s.** Leave **Create an RDS Proxy** unchecked.
- [ ] **4t.** Expand **Additional monitoring settings**; **uncheck Enable Enhanced monitoring** per tested runbook (verify against current lab instructions).
- [ ] **4u.** Review all settings; select **Create database**.

**STOP & VERIFY:** RDS instance appears in Databases (it may take several minutes to become Available). Do not assume Console defaults match the lab.

## 3 — Test existing café (original steps 5–10)
- [ ] **5.** EC2 → Instances → locate running **CafeServer**.
- [ ] **6.** Copy its current public IPv4 to **private notes**; build `http://<public-ip>/cafe`.
- [ ] **7.** Open the café page in a browser.
- [ ] **8.** Select **Menu**.
- [ ] **9.** Set an item quantity to 1 → **Submit Order**; confirm success.
- [ ] **10.** Open **Order History**; verify new and historical orders; note the latest order number privately.

## 4 — Open EC2 terminal (original steps 11–15)
- [ ] **11.** EC2 → Instances → select **CafeServer**.
- [ ] **12.** Select **Connect**.
- [ ] **13.** Choose **Session Manager**.
- [ ] **14.** Select **Connect**; confirm terminal opens.
- [ ] **15a.** Run `sudo su`.
- [ ] **15b.** Run `su ec2-user`.
- [ ] **15c.** Run `whoami` → expect `ec2-user`.
- [ ] **15d.** Run `cd /home/ec2-user/`.

## 5 — Inspect original database (original steps 16–25)
- [ ] **16.** Run `service mariadb status` and `mysql --version`.
- [ ] **17.** Open **Secrets Manager** in another Console tab.
- [ ] **18.** Select **Secrets**.
- [ ] **19.** Open `/cafe/dbPassword`.
- [ ] **20.** **Retrieve secret value**; retain original local database password privately.
- [ ] **21.** At EC2 terminal run `mysql -u root -p`.
- [ ] **22.** Enter original local DB password; confirm MariaDB prompt.
- [ ] **23a.** Run `show databases;`.
- [ ] **23b.** Run `use cafe_db;`.
- [ ] **23c.** Run `show tables;`.
- [ ] **23d.** Run ``select * from `order`;``.
- [ ] **24.** Run ``select * from `order_item`;``.
- [ ] **25.** Run `exit;` to return to Linux.

## 6 — Export local database (original steps 26–29)
- [ ] **26.** From `/home/ec2-user/`, run `mysqldump --databases cafe_db -u root -p > CafeDbDump.sql`.
- [ ] **27.** Enter **original local database** password.
- [ ] **28.** Run `ls` → verify `CafeDbDump.sql` exists.
- [ ] **29.** Run `cat CafeDbDump.sql` → verify SQL definitions and INSERT statements.

**STOP & VERIFY:** Dump file exists and contains SQL. If `Permission denied`, verify `whoami` is `ec2-user` and you are in `/home/ec2-user/`; do not export as `ssm-user` from a protected directory.

## 7 — Check RDS and answer lab questions (original steps 30–33)
- [ ] **30.** RDS → Databases → verify `cafedatabase` status **Available**.
- [ ] **31.** Lab instructions → **AWS Details**.
- [ ] **32.** Select **Access the multiple choice questions**.
- [ ] **33.** Answer questions using **current** RDS/VPC evidence (AZ, public access, subnet Name tag, security group rule count). **Do not rely on an earlier attempt's rule count.**

## 8 — Establish private connectivity (original steps 34–36)
- [ ] **34a.** RDS → Databases → `cafedatabase` → **Connectivity & security** → **Additional configurations**.
- [ ] **34b.** Copy current RDS endpoint into **private notes**; verify port **3306**.
- [ ] **34c.** Click linked **dbSG** → EC2 **Security Groups**.
- [ ] **34d.** **Inbound rules → Edit inbound rules → Add rule**.
- [ ] **34e.** Type **MYSQL/Aurora**, protocol **TCP**, port **3306**.
- [ ] **34f.** Source **Custom** → select **CafeSG** (confirm it's attached to CafeServer). **Never choose Anywhere-IPv4**.
- [ ] **34g.** Select **Save rules**.
- [ ] **34h.** In EC2 Session Manager, run `nmap -Pn <rds-endpoint>` with your private endpoint substituted.
- [ ] **34i.** Confirm `3306/tcp open mysql`.
- [ ] **34j.** Run `mysql -u admin -p --host <rds-endpoint>`; enter **RDS master password** (not old local password).
- [ ] **34k.** Confirm `MariaDB [(none)]>` prompt.
- [ ] **35.** Run `show databases;` → `cafe_db` should **not** exist yet.
- [ ] **36.** Run `exit;`.

**STOP & VERIFY:** Port 3306 open **and** successful MariaDB login. `nmap` and `mysql` are **two separate commands**; don't append `nmap -Pn` to `mysql --host`.

## 9 — Import and verify (original steps 37–42)
- [ ] **37.** From Linux in directory containing dump: `mysql -u admin -p --host <rds-endpoint> < CafeDbDump.sql`.
- [ ] **38.** Enter **RDS master password**; confirm no import error.
- [ ] **39.** Reconnect: `mysql -u admin -p --host <rds-endpoint>`.
- [ ] **40.** Enter **RDS master password**.
- [ ] **41a.** `show databases;` → `cafe_db` now exists.
- [ ] **41b.** `use cafe_db;`.
- [ ] **41c.** `show tables;` → `order`, `order_item`, `product`, `product_group`.
- [ ] **41d.** ``select * from `order`;`` → historical orders present.
- [ ] **42.** `exit;`.

**STOP & VERIFY:** Imported tables and orders are present before changing application secrets.

## 10 — Point application at RDS (original steps 43–45)
- [ ] **43.** Return to **Secrets Manager** tab.
- [ ] **44.** Select **Secrets**; locate seven `/cafe/` secrets.
- [ ] **45a.** Open `/cafe/dbUrl` → **Retrieve secret value → Edit** → set to **current RDS endpoint only** (no `http://`) → **Save**.
- [ ] **45b.** Open `/cafe/dbUser` → **Retrieve secret value → Edit** → set to `admin` → **Save**.
- [ ] **45c.** Open `/cafe/dbPassword` → **Retrieve secret value → Edit** → set to **RDS master password** → **Save**. Do not paste the password in this Issue.
- [ ] **45d.** Confirm `/cafe/dbName`, `/cafe/currency`, `/cafe/timeZone`, `/cafe/showServerInfo` are **unchanged**.

**STOP & VERIFY:** All three updated secrets saved. **No PHP code edits** required.

## 11 — Prove migration works (original steps 46–48)
- [ ] **46.** EC2 terminal → `sudo service mariadb stop`.
- [ ] **47a.** Browser → `http://<public-ip>/cafe/menu.php` using current EC2 public IP from private notes.
- [ ] **47b.** Place a **new** order; confirm success while local MariaDB is stopped.
- [ ] **48.** Open **Order History**; verify both new and historical orders.

**STOP & VERIFY:** New order + historical orders visible **after local database was stopped**.

## 12 — Submit and close (original steps 49–52)
- [ ] **49.** AWS Academy lab → **Submit** → **Yes**.
- [ ] **50.** After grading, open **Grades**; record score in private notes. Only the last submission score is retained.
- [ ] **51.** Review **Submission Report** and resolve any issues before final submission.
- [ ] **52.** Save permitted notes/screenshots → **End Lab → Yes**; confirm **Ended AWS Lab Successfully**.

## Final sign-off
- [ ] All original lab steps **1–52** accounted for.
- [ ] RDS and security group verified.
- [ ] SQL dump imported and records checked.
- [ ] Café works with local MariaDB stopped.
- [ ] New and old orders visible.
- [ ] Grade and report reviewed; lab ended.

**Troubleshooting notes (no secrets or live addresses):** Add a comment to this Issue with symptom → investigation → fix → verification, or update the permanent runbook after confirming a new finding.
