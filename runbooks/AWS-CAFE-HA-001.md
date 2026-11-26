# AWS-CAFE-HA-001 — Scalable and Highly Available Café

> AWS Academy challenge lab · Original steps 1–22 · Console instructions + Session Manager commands
>
> Use only resources from the **current lab session**. AWS Academy permissions and the live console may differ from standard AWS documentation.

## Notepad — capture as you go

Copy this block into Notepad; replace blanks with **current-session** values.

```text
Region:
Lab VPC ID / IPv4 CIDR:
Public Subnet 1 ID / AZ:
Public Subnet 2 ID / AZ:
Private Subnet 1 ID / AZ:
Private Subnet 2 ID / AZ:
Private Subnet 2 route table ID:
NAT gateway 2 ID:
CafeSG ID:
Launch template version:
Launch template name:
New key pair name (key file stored safely):
Auto Scaling group name:
ALB security group name / ID:
Target group name:
Load balancer name:
Load balancer DNS:
Application URL: http://<LOAD-BALANCER-DNS>/cafe
```

## Access the lab

### Step 1 — Start the lab

- [ ] AWS Academy instructions → Start Lab.
- [ ] Wait for the status indicator beside AWS to turn green. The session timer starts.
- [ ] If needed, Start Lab again before the timer reaches zero to refresh the session length.

### Step 2 — Open the AWS Console

- [ ] Select the AWS link above the lab terminal.
- [ ] If blocked, allow pop-ups and open the new console tab.

### Step 3 — Arrange the workspace

- [ ] Keep the lab instructions and AWS Console accessible side by side.

## Task 1 — Inspect the environment

### Step 4 — Inspect the existing network

- [ ] VPC → Your VPCs: locate Lab VPC; capture VPC ID and IPv4 CIDR.
- [ ] VPC → Subnets: identify both public and both private subnets; capture their IDs and Availability Zones.
- [ ] VPC → Route tables: inspect the route table associated with each subnet and its default route. Identify the existing NAT gateway under NAT gateways.
- [ ] EC2 → Security Groups: select CafeSG and inspect inbound/outbound rules; capture its ID in Notepad.
- [ ] EC2 → Instances: inspect CafeWebAppServer, its subnet, and public IP (if any).
- [ ] EC2 → AMIs: locate the lab-created image. Do not create a new AMI.

  **Check:** Identify the subnet/route/security-group facts needed for Step 7. Do not modify resources yet.

### Step 5 — Open AWS Details

- [ ] Return to the lab instructions → AWS Details.

### Step 6 — Open the questions

- [ ] Select Access the multiple choice questions.

### Step 7 — Answer and submit six questions

- [ ] Which ports are open in CafeSG?
- [ ] Can you connect from the internet to instances in Public Subnet 1?
- [ ] Should an instance in Private Subnet 1 reach the internet?
- [ ] Should an instance in Private Subnet 2 reach the internet?
- [ ] Can you connect to CafeWebAppServer from the internet?
- [ ] What is the name of the AMI?
- [ ] Choose Submit on the question page. Use observed configuration, not guessed answers.

## Task 2 — NAT gateway and second Availability Zone

### Step 8 — Create NAT gateway 2

- [ ] VPC → NAT gateways → Create NAT gateway.
- [ ] Enter a name (suggested: CafeNAT-2).
- [ ] Select Public Subnet 2 in the second Availability Zone. Match its ID to Notepad.
- [ ] Before creation, verify the selected subnet is Public Subnet 2, NOT Private Subnet 2.
- [ ] Select a public NAT gateway. If an availability mode is offered, select Zonal to place it in Public Subnet 2.
- [ ] Allocate an Elastic IP as prompted.
- [ ] Create the NAT gateway; wait for Available. Capture its ID.
- [ ] Check that Public Subnet 2's route table has internet access through an internet gateway. If not, investigate before continuing.

  **Check:** NAT gateway 2 is **Available** in Public Subnet 2.

> **Known failure:** A public NAT gateway created in Private Subnet 2 can show Available but fail the grader. In the repeat lab, creating CafeNAT-2 in Public Subnet 2 was followed by a full-credit resubmission. A NAT gateway cannot be moved; create a replacement, verify its public subnet and update the private route before removing the incorrect gateway, if permitted. Cleanup and route changes during that recovery were not independently confirmed.

### Step 9 — Route Private Subnet 2 through NAT gateway 2

- [ ] VPC → Subnets → select Private Subnet 2; open its Route table details. Capture the route table ID.
- [ ] VPC → Route tables → select that exact route table → Subnet associations. Verify which subnets use it.
- [ ] If it also serves Private Subnet 1, do not redirect both subnets. Create a new route table in Lab VPC and explicitly associate Private Subnet 2 with it. Preserve its local route and any other required routes from the original table.
- [ ] On the route table associated with Private Subnet 2, open Routes → Edit routes.
- [ ] Add the IPv4 default route, or edit its target if the route already exists:

     ```text
     Destination: 0.0.0.0/0
     Target type: NAT Gateway
     Target:       <NAT-GATEWAY-2-ID>
     ```

- [ ] Save changes. Recheck Private Subnet 1's route table; do not change its intended NAT routing.

  **Check:** Private Subnet 2 routes internet-bound IPv4 traffic through NAT gateway 2. A private subnet does **not** use an internet gateway as its direct default-route target.

## Task 3 — Launch template

### Step 10 — Create the café launch template

- [ ] EC2 → Launch Templates → Create launch template.
- [ ] Enter the launch template name:

     ```text
     CafeWebLaunchTemplate
     ```
- [ ] Application and OS Images → My AMIs: select Cafe WebServer Image (search Cafe if needed).
- [ ] Select the instance type:

     ```text
     t2.micro
     ```
- [ ] Key pair (login) → Create new key pair; download its private key, return to the template, and select the new key pair. Never upload the key file to GitHub.
- [ ] Network settings → Subnet and Availability Zone: leave both out of the template; Auto Scaling selects the private subnets.
- [ ] Firewall → Select existing security group → choose CafeSG in Lab VPC (the generated group name may contain CafeSG rather than equal it).
- [ ] Match the selected security group ID to CafeSG in the Security Groups console; capture the ID in Notepad. Do not select default or CafeALB-SG.
- [ ] Resource tags → Add new tag (not merely template tags):

     ```text
     Key:           Name
     Value:         webserver
     Resource type: Instances
     ```

- [ ] Advanced details → IAM instance profile:

     ```text
     CafeRole
     ```
- [ ] Review the selected AMI, instance type, key pair, CafeSG, instance tag, and CafeRole; Create launch template.

- [ ] Open the saved launch template version and verify the security group ID matches CafeSG. If a network interface is configured, inspect its security groups as well.
- [ ] Capture the launch template name and version in Notepad.
- [ ] NETC 3300: Capture the Step 10 screenshot of the completed launch template configuration.

  **Check:** The launch template contains every required setting.

## Task 4 — Auto Scaling group

### Step 11 — Create the Auto Scaling group

- [ ] EC2 → Auto Scaling Groups → Create Auto Scaling group.
- [ ] Enter the Auto Scaling group name.
- [ ] Choose the Step 10 launch template and its correct default version.
- [ ] Choose Next.
- [ ] Choose instance launch options: keep the launch template's t2.micro; choose Lab VPC and Private Subnet 1 + Private Subnet 2 (different AZs).
- [ ] In the observed console, leave Availability Zone distribution = Balanced best effort and Capacity Reservation preference = Default → Next.
- [ ] Integrate with other services: do not attach a load balancer or target group yet. Skip advanced options → Next.
- [ ] Configure group size and scaling:

     ```text
     Desired capacity: 2
     Minimum capacity: 2
     Maximum capacity: 6
     Scaling policy:  Target tracking scaling policy
     Metric type:     Average CPU utilization
     Target value:    25
     Instance warmup: 60 seconds
     ```

- [ ] Set 60 in instance warmup, not in the health-check grace period. Continue past optional notifications and tags without unnecessary changes.
- [ ] Review → Create Auto Scaling group.

  **Check:** Group has desired 2, minimum 2, maximum 6 and CPU target tracking at 25 with warmup 60 seconds.

### Step 12 — Verify two launched instances

- [ ] EC2 → Instances → Refresh; allow startup time.
- [ ] Identify two new instances named webserver, one in each private subnet/AZ. Distinguish these from the original CafeWebAppServer.

- [ ] Select each new webserver → Security tab → verify its attached security group ID matches CafeSG in Notepad.

  **Check:** Two new `webserver` instances are running with CafeSG.

  **Known failure:** Instances launched with the default security group caused target health-check timeouts in the repeat lab. Creating template version 2 and setting it as default succeeded, but this does not change existing instances. Direct instance security group changes and StartInstanceRefresh were denied by Academy permissions. Do not repeat these denied actions or assume existing instances were corrected.

## Task 5 — Application Load Balancer

### Step 13 — Create the load balancer and target group

- [ ] EC2 → Load Balancers → Create load balancer → Application Load Balancer → Create.
- [ ] Enter a name (suggested: CafeALB); select Internet-facing and IPv4 for the lab's IPv4 HTTP architecture.
- [ ] Network mapping: select Lab VPC and both public subnets (Public Subnet 1 and Public Subnet 2), not the private subnets. Verify subnet IDs.
- [ ] Security groups → Create new security group (opens a separate page/tab). Select Lab VPC. Configure the new security group:

     ```text
     Name: CafeALB-SG
     Description: Allows inbound HTTP traffic from the internet to the Cafe Application Load Balancer.
     ```

- [ ] Add the inbound HTTP rule:

     ```text
     Type:    HTTP
     Port:    TCP 80
     Source:  0.0.0.0/0
     ```

- [ ] Save the security group. Return to the ALB creation tab, refresh the security-group selection if necessary, select the new group, and remove any unintended default selection.
- [ ] Listeners and routing: keep HTTP:80 and Forward to target groups. Click create target group beside the target-group selector; a separate page/tab may open.
- [ ] On the Create target group page, configure:

     ```text
     Target type:       Instances
     Target group name: CafeTargetGroup
     Protocol:          HTTP
     Port:              80
     IP address type:   IPv4
     VPC:               Lab VPC
     Health check:      HTTP (start with default path /)
     ```

- [ ] Continue to Register targets; do not register instances manually. Choose Create target group.
- [ ] Return to the ALB creation tab. Click the refresh icon beside the target-group dropdown, then select the new target group as the listener's default forward action. Keep stickiness off; skip HTTPS settings.
- [ ] Review the ALB's public subnets, security group, HTTP listener, and target group → Create load balancer.
- [ ] EC2 → Load Balancers → select the new ALB; wait for Active. Capture its DNS name in Notepad.

  **Check:** ALB is **Active**, listener HTTP:80 forwards to the new target group, and the ALB uses both **public** subnets.

### Step 14 — Attach target group to Auto Scaling

- [ ] EC2 → Auto Scaling Groups → select the Step 11 group → Integrations.
- [ ] Under Load balancing, choose Edit and attach the existing target group created in Step 13. Save/update. Do not create another load balancer.
- [ ] EC2 → Target Groups → select CafeTargetGroup → Targets. Wait for Auto Scaling to register the two webserver instances and for health checks to run.

  **Check:** Two registered targets become **Healthy**. If not, inspect the Targets tab health status details, verify the actual instance security groups (Step 12), the target port, ALB outbound rules, and whether `CafeSG` allows HTTP from the ALB; make only evidence-based corrections. The lab does not prescribe a health-check path, so `/` is a starting default, not a verified application-specific value.

## Task 6 — Test the application and scaling

### Step 15 — Open the café application

- [ ] EC2 → Load Balancers → select the ALB → Details → DNS name; copy the DNS name.
- [ ] Replace the placeholder and open the complete URL in your browser:

     ```text
     http://<LOAD-BALANCER-DNS>/cafe
     ```

- [ ] Confirm the café application loads. If not, inspect: NAT gateway 2, Private Subnet 2 route table, CafeRole in the launch template, ALB public subnet mapping, Auto Scaling private subnet mapping, HTTP security-group rules, and target health.

  **Check:** Café loads through the ALB URL. Do not use an individual instance's IP.

### Step 16 — Connect through Session Manager

- [ ] EC2 → Instances → select one running Auto Scaling webserver instance → Connect → Session Manager → Connect.
- [ ] Use the AWS Session Manager terminal, not a local PowerShell/SSH/VS Code terminal.
- [ ] If Session Manager is unavailable, inspect the instance's SSM readiness, network connectivity, and assigned CafeRole before attempting fixes.

### Step 17 — Run the CPU stress test

- [ ] In the Session Manager terminal, install the tool:

     ```bash
     sudo dnf install stress -y
     ```

- [ ] Then run the lab's 10-minute CPU workload:

     ```bash
     stress --cpu 1 --timeout 600
     ```

  **Check:** Stress runs without command errors. Leave the session running while monitoring scaling.

### Step 18 — Verify scale-out

- [ ] In another AWS Console tab, open EC2 → Auto Scaling Groups → select the group → Activity.
- [ ] Also open EC2 → Instances and refresh periodically.
- [ ] Look for additional webserver instances launched beyond the original two. Allow time for CPU metric collection, scaling evaluation, and startup.
- [ ] If no scale-out occurs, check group average CPU and scaling activities/alarms; stressing one of two instances does not guarantee an immediate group-average breach of 25%.

- [ ] NETC 3300: Capture the Step 18 screenshot showing more than two Auto Scaling webserver instances.

  **Check:** Additional instances launch during load. The guide requires scale-out verification, not a separate scale-in demonstration.

## Submit and finish

### Step 19 — Submit the lab

- [ ] Return to AWS Academy instructions → Submit → Yes.

### Step 20 — View grades

- [ ] If results do not appear after a couple of minutes, choose Grades at the top of the instructions.
- [ ] If you make verified corrections, submit again; the last submission is recorded.

### Step 21 — Review the submission report

- [ ] Open Submission Report; inspect detailed feedback and any missed objectives.
- [ ] Verify required screenshot evidence from Steps 10 and 18 is saved separately for coursework.

### Step 22 — End the lab

- [ ] Only after submission and required evidence: End Lab → Yes.
- [ ] Confirm the message Ended AWS Lab Successfully.

---

### Reference and validation notes

- Source: AWS Academy, Challenge Lab: Creating a Scalable and Highly Available Environment for the Café. Reviewed original steps 1–22, six questions, required settings, stress commands, submission and end-lab instructions. NETC 3300 screenshot reminders retained at Steps 10 and 18.
- AWS execution: Earlier attempt demonstrated scale-out (user report). Repeat attempt exposed incorrect NAT subnet placement and default instance security groups. Screenshots confirmed replacement NAT gateway creation in Public Subnet 2 and launch template version 2 set as default. Instance refresh was denied. Actual recovery of target health, final route association and removal of incorrect NAT gateway were not independently confirmed.
- Grader verification: NETC 3300 initially 26/31 (Task 2 failed); after NAT correction, user confirmed 31/31. Task 6 was not manually repeated in that attempt despite full grader credit. Keep Steps 15–18 as required execution checks; grader credit alone is not proof of application or scaling behavior.
- Documentation audit: Steps 1–22 reconciled with guide; individual action checkboxes, configuration values, session-specific placeholders, execution contexts and screenshot reminders retained. Revised checklist has not been executed end-to-end in AWS or retested in the VS Code extension.
- AWS references: [NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) · [Route tables](https://docs.aws.amazon.com/vpc/latest/userguide/WorkWithRouteTables.html) · [Launch templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html) · [ALB creation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-application-load-balancer.html) · [Target groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-target-group.html) · [Attach target group](https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html) · [Target tracking](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html). References not freshly checked online during this revision.
