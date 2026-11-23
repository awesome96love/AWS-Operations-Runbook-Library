# Automating Infrastructure with AWS CloudFormation
> AWS Academy guided lab | VS Code interactive Markdown checklist  
> **Validation:** Lab guide reviewed; selected console navigation observed during execution. Final submission score and all individual outcomes were **not** supplied in this conversation.  
> **Source:** *Guided Lab: Automating Infrastructure Deployment with AWS CloudFormation* (46 numbered steps; source skips 13 and repeats 35).

## 📌 Before starting — capture in Notepad

- [ ] Download these three original files from the links in the AWS Academy lab guide (do not create substitutes):
  - [ ] lab-network.yaml
  - [ ] lab-application.yaml
  - [ ] lab-application2.yaml
- [ ] Open Notepad and prepare to capture the following current-session values:

```text
Network stack: lab-network
Application stack: lab-application
Application tag: application=inventory
NetworkStackName: lab-network
VPC ID:
Public subnet ID:
Application URL:
Security group ID:
EBS snapshot ID:
Lab score:
```

## 0. Access AWS Console — original steps 1–3

- [ ] 1. Choose Start Lab in AWS Academy; wait for the lab session to initialize and the AWS status indicator to turn green.
- [ ] 2. Choose AWS to open the Management Console. Allow the pop-up if your browser blocks it.
- [ ] 3. Arrange AWS Console and lab instructions side by side. Do not change Region unless the guide specifically says to.

## 1. Deploy networking layer — original steps 4–12

- [ ] 4. Download lab-network.yaml from the original lab guide. Optionally inspect its YAML in VS Code.
- [ ] 5. Open CloudFormation from the AWS Console search bar.
- [ ] 6. Create the network stack:
  - [ ] Choose Create stack → With new resources (standard).
  - [ ] Select Choose an exsisting template.
  - [ ] Select Upload a template file → Choose file; upload lab-network.yaml.
  - [ ] Choose Next.
  - [ ] Enter stack name: lab-network.
  - [ ] Choose Next.
  - [ ] Under Tags, add Key application, Value inventory.
  - [ ] Choose Next, review, then Submit.
- [ ] 7. Open the Stack info tab.
- [ ] 8. Refresh as needed (approximately every 15 seconds) until CREATE_COMPLETE.
- [ ] 9. Open Resources and inspect the resources CloudFormation created.
- [ ] 10. Open Events and inspect creation events; check for errors.
- [ ] 11. Open Outputs:
  - [ ] Copy VPC ID to Notepad.
  - [ ] Copy PublicSubnet ID to Notepad.
  - [ ] Notice their Export name values, which the application stack will import.
- [ ] 12. Open Template; inspect its resources and Outputs section.

**Success check:** `lab-network` shows `CREATE_COMPLETE`, and both VPC and public subnet outputs are present.

## 2. Deploy application layer — original steps 14–18

> The original guide does not contain a numbered Step 13.

- [ ] 14. Download lab-application.yaml from the original guide.
- [ ] 15. Create the application stack:
  - [ ] Choose Create stack → With new resources (standard).
  - [ ] Select Choose an exsisting template → Upload a template file; choose lab-application.yaml.
  - [ ] Choose Next.
  - [ ] Enter stack name: lab-application.
  - [ ] Confirm parameter NetworkStackName is lab-network.
  - [ ] Choose Next.
  - [ ] Add tag Key application, Value inventory.
  - [ ] Choose Next, review, then Submit.
  - [ ] While the stack builds, inspect Events and Resources.
- [ ] 16. Open Stack info; wait for CREATE_COMPLETE.
- [ ] 17. Open Outputs.
- [ ] 18. Copy the application URL into Notepad; open it in a new browser tab and verify the web page loads.

**Success check:** Both stacks show `CREATE_COMPLETE`, and the application URL works.

## 3. Update application security group — original steps 19–31

- [ ] 19. Open EC2 from the AWS Console search bar.
- [ ] 20. Navigate to Network & Security → Security Groups.
- [ ] 21. Select the security group named lab-application-WebServerSecurityGroup (the console may show a longer generated name).
- [ ] 22. Inspect Inbound rules; before the update, it should permit HTTP (TCP 80), not HTTPS (TCP 443).
- [ ] 23. Return to CloudFormation using the service search or console navigation.
- [ ] 24. Download lab-application2.yaml from the original guide. It adds an inbound rule:

```yaml
- IpProtocol: tcp
  FromPort: 443
  ToPort: 443
  CidrIp: 0.0.0.0/0
```

- [ ] 25. In Stacks, select lab-application.
- [ ] 26. Choose Update → Make a direct update (this was the option shown in the observed console).
  - [ ] Select Replace current template.
  - [ ] Select Upload a template file → Choose file; upload lab-application2.yaml.
- [ ] 27. Choose Next through the remaining wizard screens, preserving existing settings. On the review page, inspect the Change set preview:
  - [ ] Confirm WebServerSecurityGroup is being modified.
  - [ ] Confirm Replacement = False, if shown. Do not submit if the preview unexpectedly replaces unrelated resources.
- [ ] 28. Choose Submit.
- [ ] 29. Refresh Stack info until UPDATE_COMPLETE.
- [ ] 30. Return to EC2 → Security Groups.
- [ ] 31. Select the application web-server security group and inspect Inbound rules:
  - [ ] HTTP: TCP 80 (existing rule).
  - [ ] HTTPS: TCP 443, source 0.0.0.0/0 (new rule).

**Success check:** Stack status is `UPDATE_COMPLETE`; inbound TCP 443 appears. **The lab application still runs on HTTP port 80; do not expect HTTPS to serve the site.**

## 4. Explore the template visually — original steps 32–35 (35 repeated in guide)

> **Observed console difference:** The lab says **Infrastructure Composer**. During our session, the CloudFormation sidebar instead showed **Template studio**. Its **Actions** menu offered **Copy template**, **Download template**, and **Download resources diagram**—not **Open/Import**. **Download template exports the current editor content; it does not import `lab-application2.yaml`.**

- [ ] 32. Return to CloudFormation.
- [ ] 33. In the left navigation, open Template studio if Infrastructure Composer is not listed.
- [ ] 34. Load the updated application YAML for inspection:
  - [ ] Open lab-application2.yaml in VS Code.
  - [ ] Press Ctrl+A, then Ctrl+C to copy the entire file.
  - [ ] In Template studio, click the YAML editor and press Ctrl+A, then Ctrl+V to replace the example content.
  - [ ] Check whether the visual canvas reflects the pasted template. This workaround was discussed but its successful rendering was not confirmed by a subsequent screenshot.
- [ ] 35 (diagram). Examine the visual representation of the template and locate the EC2 instance, security group, and EBS volume if rendered.
- [ ] 35 (exploration). Try the applicable exercises:
  - [ ] Select resources and inspect available properties/details.
  - [ ] Compare visual resources to their definitions in the YAML editor.
  - [ ] Observe how changes to YAML affect the canvas, without deploying them.
  - [ ] Open or paste lab-network.yaml and inspect its networking resources.
  - [ ] Original Composer-only exercise: Drag a resource from the resource-types pane into the canvas, if supported.
  - [ ] Original Composer-only exercise: Drag connector circles between resources, if supported.

**Compatibility note:** Template studio may not support the original Composer drag-and-drop exercises. If the controls are absent, mark those two exercises **not available in this console** rather than claiming to have performed them. The original guide's objective is exploration; do not deploy experimental changes.

**Do not choose `Save and continue` merely to explore.**

## 5. Delete application stack and verify snapshot — original steps 36–42

> **Required cleanup in the guide:** Delete **only** `lab-application`; leave `lab-network` intact. The application template's EBS volume has `DeletionPolicy: Snapshot`.

- [ ] 36. Leave Template studio / Composer and return to the main CloudFormation → Stacks page. Discard unsaved experiments if prompted.
- [ ] 37. Select lab-application (not lab-network).
- [ ] 38. Choose Delete.
- [ ] 39. Confirm Delete in the dialog. Inspect Events while deletion runs; a snapshot event may appear.
- [ ] 40. Wait until lab-application disappears from the active stacks list. Confirm lab-network remains.
- [ ] 41. Open EC2.
- [ ] 42. Navigate to Elastic Block Store → Snapshots:
  - [ ] Locate the recent snapshot named Web Data.
  - [ ] Verify its start time matches this lab session.
  - [ ] Wait for Completed status.
  - [ ] Copy the snapshot ID to Notepad.

**Success check:** Application stack deleted; network stack retained; recent **Web Data** EBS snapshot reaches **Completed**.

## 6. Submit, inspect grade, end lab — original steps 43–46

- [ ] 43. In AWS Academy, choose Submit, then confirm Yes.
- [ ] 44. If results do not appear within a couple of minutes, choose Grades. Re-submit after corrections if necessary; the last submission is recorded.
- [ ] 45. Choose Details → View Submission Report; record the score and any missed objectives in Notepad.
- [ ] 46. Choose End Lab → Yes; verify Ended AWS Lab Successfully.

**Final audit:** Confirm all checked actions were actually completed, especially Step 27 change-set review, Step 31 TCP 443 verification, Step 42 snapshot verification, and Step 45 grader feedback. Any separate college screenshot/Word submission is **outside this AWS Academy guide** and must be checked against that course assignment.
