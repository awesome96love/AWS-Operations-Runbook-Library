# Creating a VPC Peering Connection

AWS Academy | Guided Lab | NETC 3300 — Module 6, Assignment 1

Estimated duration: 30 minutes. Documentation reviewed; live execution and grading pending.

## Session capture — Notepad

Copy into Notepad. Use current-session values only.

```text
AWS Region:
EC2PublicIP:
Database Endpoint:
Lab-Peer ID:
Final Grade:
```

## Access the AWS Management Console

- [ ] 1. Select Start Lab; wait for the AWS indicator to turn green. If more time is needed, select Start Lab again to extend the timer.
- [ ] 2. Select the green AWS indicator to open the console. Allow browser pop-ups if blocked.
- [ ] 3. Arrange the console and instructions side by side. Retain the lab Region.

## Task 1 — Create a VPC peering connection

- [ ] 4. Search for and open VPC.
- [ ] 5. Select Peering connections.
- [ ] 6. Select Create peering connection.
- [ ] 6a. Enter the name.

```text
Lab-Peer
```

- [ ] 6b. Select Lab VPC as requester.
- [ ] 6c. Select My account and This Region if both lab VPCs are in the current account and Region.
- [ ] 6d. Select Shared VPC as accepter.
- [ ] 6e. Select Create peering connection.
- [ ] 7. Select Actions > Accept request; confirm acceptance if prompted.
- [ ] Verify Lab-Peer is Active and capture its peering ID in Notepad.

## Task 2 — Configure route tables

- [ ] 8. Select Route tables.
- [ ] 9. Select Lab Public Route Table (Lab VPC).
- [ ] 10. Open Routes > Edit routes.
- [ ] 10a. Select Add route and enter the destination.

```text
10.5.0.0/16
```

- [ ] 10b. Select Peering Connection as target type, then Lab-Peer.
- [ ] 10c. Save changes.
- [ ] 11. Return to Route tables, clear the previous selection, and select Shared-VPC Route Table.
- [ ] 12. Open Routes > Edit routes.
- [ ] 12a. Select Add route and enter the destination.

```text
10.0.0.0/16
```

- [ ] 12b. Select Peering Connection as target type, then Lab-Peer.
- [ ] 12c. Save changes.
- [ ] Verify both new routes are active and reference the same peering ID. Preserve existing routes.

## Task 3 — Enable VPC Flow Logs

- [ ] 13. Select Your VPCs > Shared VPC.
- [ ] 14. Open the Flow logs tab.
- [ ] 15. Select Create flow log.
- [ ] SCREENSHOT — Step 15: Capture the Create flow log screen for Shared VPC before creating the flow log.
- [ ] 16. Configure the flow log name.

```text
SharedVPCLogs
```

- [ ] 16a. Set Maximum aggregation interval to 1 minute.
- [ ] 16b. Select Send to CloudWatch Logs.
- [ ] 16c. Enter the destination log group name to create a new log group.

```text
ShareVPCFlowLogs
```

- [ ] 16d. Select the existing IAM role.

```text
vpc-flow-logs-Role
```

- [ ] 16e. Verify Filter is All and record format is AWS default (recommended, not specified by guide).
- [ ] 16f. Select Create flow log and check the creation notification.
- [ ] 17. Return to the Flow logs tab and verify SharedVPCLogs exists on Shared VPC.
- [ ] 18. Select ShareVPCFlowLogs under Destination name to open CloudWatch. Keep the tab open.

If the log group does not exist yet, wait a few minutes and refresh. Do not create a duplicate flow log.

## Task 4 — Test the VPC peering connection

- [ ] 19. Open AWS Details in the lab instructions. Copy EC2PublicIP and Endpoint into Notepad.
- [ ] 20. Paste EC2PublicIP into a new browser tab (use HTTP if needed).
- [ ] SCREENSHOT — Step 20: Capture the inventory application displaying “Please configure Settings to connect to database” before changing settings.
- [ ] 21. Open Settings in the inventory application.
- [ ] 21a. Paste the current database Endpoint from Notepad into Endpoint.
- [ ] 21b. Enter the database name.

```text
inventory
```

- [ ] 21c. Enter the username.

```text
admin
```

- [ ] 21d. Enter the lab-provided password from Step 21 of the original guide.
- [ ] 21e. Select Save.
- [ ] Verify inventory records are displayed.

If connectivity fails, verify the peering status, both routes, endpoint and credentials before investigating preconfigured security groups or network ACLs. Do not modify them without evidence.

## Task 5 — Analyze VPC Flow Logs

- [ ] 22. Return to the ShareVPCFlowLogs CloudWatch tab.
- [ ] 23. Open the relevant log stream beginning with eni-.
- [ ] 24. Wait for traffic records to appear; refresh as needed.
- [ ] 25. Examine records containing database port 3306.
- [ ] 25a. Identify account ID, ENI, source and destination private IP addresses, source and destination ports, action and log status.
- [ ] 25b. Verify successful database traffic shows ACCEPT and OK. Look for port 3306 as destination on requests and source on responses.

Default record field order (reference only):

```text
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
```

Log delivery can take several minutes despite the one-minute aggregation interval. If records remain absent, verify flow log status, destination, role and application traffic. Use current-session IPs; the guide's example application IP is inconsistent between its table and screenshot.

## Submit and review

- [ ] 26. In AWS Academy, select Submit > Yes.
- [ ] 27. Review the grade; select Grades if results do not appear after a few minutes. Record the score in Notepad.
- [ ] 28. Select Details > View Submission Report and review feedback. Correct any failed objectives and resubmit; the last submission is recorded.
- [ ] Verify both required NETC 3300 screenshots (Steps 15 and 20) are saved before ending the lab.
- [ ] 29. Select End Lab > Yes. Verify “Ended AWS Lab Successfully.”

## References and validation

- Original AWS Academy guide: Guided Lab: Creating a VPC Peering Connection, pages 1–5 (29 steps).
- https://docs.aws.amazon.com/vpc/latest/peering/create-vpc-peering-connection.html
- https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-routing.html
- https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-cwl-create-flow-log.html
- https://docs.aws.amazon.com/vpc/latest/userguide/flow-log-records.html

All original numbered steps and configuration substeps are represented. Screenshot requirements are user-provided; exact screenshot framing is a suggested interpretation. Console-only execution is retained because authorized CLI access has not been established. Live console behavior, successful execution and grader results remain unverified.
