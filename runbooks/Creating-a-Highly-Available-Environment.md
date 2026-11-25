# Creating a Highly Available Environment

AWS Academy guided lab | Estimated duration: 40 minutes

## Capture in Notepad — current session only

```text
AMI ID:
Load balancer DNS name:
Original application instance IDs / AZs:
Surviving instance ID / AZ:
Replacement instance ID / AZ:
Final grade:
```

All actions are in the AWS console unless specified. Optional tasks are not required by the guide.

## Start lab
- [ ] 1. Start Lab; wait for the green readiness indicator.
- [ ] 2. Open the AWS console using the AWS link. Allow pop-ups if blocked.
- [ ] 3. Arrange console and instructions side by side. Stay in the assigned Region.

## Task 1 — Inspect the VPC
- [ ] 4. Open VPC.
- [ ] 5. Filter by VPC: Lab VPC.
- [ ] 6. Open Your VPCs.
- [ ] 7. Select Lab VPC; verify IPv4 CIDR 10.0.0.0/16.
- [ ] 8. Open Subnets; locate Public Subnet 1 and verify VPC, CIDR 10.0.0.0/24 and Availability Zone.
- [ ] 9. Select Public Subnet 1.
- [ ] 10. Open Route table; verify local 10.0.0.0/16 and 0.0.0.0/0 to internet gateway.
- [ ] 11. Open Network ACL; review default inbound and outbound allow rules.
- [ ] 12. Open Internet gateways; verify Lab IG is attached to Lab VPC.
- [ ] 13. Open Security groups.
- [ ] 14. Select Inventory-DB.
- [ ] 15. Open Inbound rules; verify MySQL/Aurora port 3306 from 10.0.0.0/16.
- [ ] 16. Open Outbound rules; review default allow-all rule.

## Task 2 — Application Load Balancer
- [ ] 17. Open EC2.
- [ ] 18. Open Load Balancers.
- [ ] 19. Choose Create load balancer; review the available types.
- [ ] 20. Choose Application Load Balancer > Create.
- [ ] 21. Enter load balancer name: Inventory-LB.
- [ ] 22. Configure network mapping.

    - Scheme: Internet-facing
    - IP address type: IPv4
    - VPC: Lab VPC
    - Mappings: Public Subnet 1 and Public Subnet 2, in different AZs.
- [ ] 23. Choose Create a new security group (opens another tab).
- [ ] 24. Configure security group basic details.

    - Name: Inventory-LB
    - Description: Enable web access to load balancer
    - VPC: Lab VPC
- [ ] 25. Add inbound HTTP rule; source Anywhere-IPv4.
- [ ] 26. Add inbound HTTPS rule; source Anywhere-IPv4.
- [ ] 27. Create security group.


> Original guide skips step 28; no action is missing.
- [ ] 29. Return to load balancer tab; refresh security groups, select Inventory-LB and remove the default group.
- [ ] 30. Under Listeners and routing, choose Create target group (new tab).
- [ ] 31. Configure target group and advanced health checks.

    - Target type: Instances
    - Name: Inventory-App
    - VPC: Lab VPC
    - Healthy threshold: 2
    - Interval: 10 seconds
- [ ] 32. Choose Next; skip registering targets.
- [ ] 33. Review and create target group; confirm success.
- [ ] 34. Return to load balancer tab.
- [ ] 35. Refresh Listeners and routing.
- [ ] 36. Select Inventory-App as the default action for the HTTP listener.
- [ ] 37. Create load balancer.
- [ ] 38. Choose View load balancer.

## Task 3.1 — Create AMI
- [ ] 39. Open EC2.
- [ ] 40. Open Instances.
- [ ] 41. Wait for Web Server 1 to pass status checks (guide: 2/2); refresh as needed.
- [ ] 42. Select Web Server 1.
- [ ] 43. Actions > Image and templates > Create image.
- [ ] 44. Enter image name and description.

    - Image name: Web Server AMI
    - Image description: Lab AMI for Web Server
- [ ] 45. Create image; copy the new AMI ID into Notepad.

## Task 3.2 — Launch template and Auto Scaling group
- [ ] 46. Open Launch Templates.
- [ ] 47. Choose Create launch template.
- [ ] 48. Configure launch template and paste user data.

    - Name: Inventory-LT
    - Auto Scaling guidance: Enable guidance for EC2 Auto Scaling.
    - AMI: My AMIs > Web Server AMI (match captured AMI ID).
    - Instance type: t2.micro
    - Key pair: vockey
    - Firewall: Select existing security group > Inventory-App
    - Advanced details > IAM instance profile: Inventory-App-Role
    - Detailed CloudWatch monitoring: Enable
    - User data: paste the following script into the console field, not a terminal.

```bash
#!/bin/bash
# Install Apache Web Server and PHP
dnf install -y httpd php8.4 php8.4-fpm php8.4-mysqlnd mariadb1011 wget unzip
systemctl enable --now php-fpm
# Download Lab files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/12-lab-mod10-guided-Scaling/s3/scripts/al2023-inventory-app.zip -O inventory-app.zip
unzip inventory-app.zip -d /var/www/html/
# Download and install the AWS SDK for PHP
wget https://docs.aws.amazon.com/aws-sdk-php/v3/download/aws.zip
unzip -o aws.zip -d /var/www/html
# Turn on web server
systemctl enable httpd
service httpd start
```
- [ ] 49. Create launch template.
- [ ] 50. Open the Inventory-LT link in the success message.
- [ ] 51. Actions > Create Auto Scaling group.
- [ ] 52. Set group name Inventory-ASG; confirm launch template Inventory-LT.
- [ ] 53. Choose Next.
- [ ] 54. Select Lab VPC, Private Subnet 1 and Private Subnet 2.
- [ ] 55. Choose Next.
- [ ] 56. Configure load balancing, health checks and monitoring.

    - Attach to an existing load balancer > Inventory-App target group.
    - Turn on Elastic Load Balancing health checks.
    - Health check grace period: 90 seconds.
    - Enable group metrics collection within CloudWatch.
- [ ] 57. Choose Next.
- [ ] 58. Configure group size and scaling policies.

    - Desired: 2
    - Minimum: 2
    - Maximum: 2
    - No scaling policies.

> The guide's narrative says 2–6; its explicit settings are all 2.
- [ ] 59. Choose Next.
- [ ] 60. Skip notifications; choose Next.
- [ ] 61. Add tag: Name = Inventory-App; ensure it applies to launched instances.
- [ ] 62. Choose Next.
- [ ] 63. Review and create Auto Scaling group.
- [ ] 64. Refresh after approximately one minute; verify two running instances across two Availability Zones.


> Initializing status checks can be normal. Proceed with security group configuration; verify target health in Task 5.

## Task 4.1 — Load balancer security group

Already configured in Task 2; no additional action.

## Task 4.2 — Application security group
- [ ] 65. Open EC2 > Security Groups.
- [ ] 66. Select Inventory-App.
- [ ] 67. Open Inbound rules.
- [ ] 68. Choose Edit inbound rules.
- [ ] 69. Add HTTP rule from the load balancer security group.

    - Type: HTTP (80)
    - Source: Custom > search sg > Inventory-LB (security group, not CIDR).
    - Description: Traffic from load balancer
- [ ] 70. Save rules.

## Task 4.3 — Database security group
- [ ] 71. Select only Inventory-DB.
- [ ] 72. Edit inbound rules; delete the existing VPC-CIDR rule and add the application security group rule.

    - Type: MySQL/Aurora (3306)
    - Source: Custom > search sg > Inventory-App (security group, not CIDR).
    - Description: Traffic from application servers
- [ ] 73. Save rules.

## Task 5 — Test application
- [ ] 74. Open EC2 > Target Groups.
- [ ] 75. Select Inventory-App.
- [ ] 76. Open Targets; verify two registered application instances.
- [ ] 77. Refresh until both targets are healthy.


> If targets remain unhealthy, inspect their health reason, listener/target group association, security groups and instance startup. EC2 running status alone does not establish application health.
- [ ] 78. Open Load Balancers > Inventory-LB.
- [ ] 79. Copy DNS name from Details into Notepad.
- [ ] 80. Open http://<LOAD_BALANCER_DNS> in a new browser tab; verify Inventory application loads and note instance ID and AZ.
- [ ] 81. Refresh several times; verify instance IDs and AZs sometimes alternate.

## Task 6 — Test high availability
- [ ] 82. Return to EC2 console; keep application tab open.
- [ ] 83. Open Instances.
- [ ] 84. Select exactly ONE Inventory-App instance (not Web Server 1).
- [ ] 85. Instance state > Terminate instance.
- [ ] 86. Confirm Terminate.
- [ ] 87. Refresh application several times; verify it remains available and shows the surviving AZ.
- [ ] 88. Refresh EC2 Instances about every 30 seconds until replacement appears; verify replacement becomes healthy and application resumes alternating.


> Check both EC2 status checks and target group health. Record replacement ID and AZ. A draining old target is expected during replacement.

## Optional task 1 — Multi-AZ RDS
- [ ] 89. Open RDS.
- [ ] 90. Open Databases.
- [ ] 91. Select inventory-db; review details.
- [ ] 92. Choose Modify.
- [ ] 93. Availability & durability > Multi-AZ deployment > Create a standby instance.
- [ ] 94. DB instance class: db.t3.small.
- [ ] 95. Allocated storage: 20 GiB; leave other settings unchanged.
- [ ] 96. Choose Continue.
- [ ] 97. Choose Apply immediately.
- [ ] 98. Choose Modify DB instance; Modifying is expected. No need to wait.

## Optional task 2 — Second NAT gateway
- [ ] 99. Open VPC.
- [ ] 100. Open NAT gateways; review existing gateway.
- [ ] 101. Choose Create NAT gateway.
- [ ] 102. Configure the second NAT gateway using Zonal availability mode.

    - Name: NatGateway2
    - Availability mode: Zonal (Regional mode hides the subnet selector).
    - Subnet: Public Subnet 2
    - Connectivity type: Public
    - Allocate Elastic IP.
- [ ] 103. Create NAT gateway.
- [ ] 104. Open Route tables.
- [ ] 105. Choose Create route table.
- [ ] 106. Name: Private Route Table 2; VPC: Lab VPC.
- [ ] 107. Create route table.
- [ ] 108. Open Routes; verify local route.
- [ ] 109. Choose Edit routes.
- [ ] 110. Add route: destination 0.0.0.0/0; target NAT Gateway > NatGateway2.
- [ ] 111. Save changes.
- [ ] 112. Open Subnet associations.
- [ ] 113. Choose Edit subnet associations.
- [ ] 114. Select Private Subnet 2.
- [ ] 115. Save associations; verify Private Subnet 2 uses the new route table.


> Verify the NAT gateway becomes Available and the default route is active.

## Submit and end lab
- [ ] 116. Choose Submit in AWS Academy; confirm Yes.
- [ ] 117. Open Grades if results are delayed; resubmit after corrections if necessary.
- [ ] 118. Choose Details > View Submission Report; review all graded objectives.


> Prior execution: Task 5 initially scored 0/5 despite healthy targets. After confirming the HTTP application loaded, resubmission returned 75/75. If this recurs, verify target health and browser access, then resubmit before changing working resources.
- [ ] 119. After reviewing results and saving any required evidence, choose End Lab > Yes; confirm successful termination.

## Validation and save
- [ ] Save the Markdown file as Creating-a-Highly-Available-Environment.md in the runbooks folder.

Source: supplied AWS Academy PDF and HTML guide; original numbering preserved (28 absent). Previous lab execution and reported grader result: 75/75. This revised Markdown has been reviewed against the guide but has not itself been executed. Optional task completion and continuous availability during the failure test were not independently established. No separate college screenshot rubric was supplied.
