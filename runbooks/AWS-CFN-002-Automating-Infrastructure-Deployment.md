# AWS-CFN-002 — Challenge Lab: Automating Infrastructure Deployment

AWS Academy | Partially validated in AWS | Grader: 24/30 (80%)

Validated November 23, 2026: static site, both pipelines, Virginia dynamic site, Oregon network, and all six questions. Outstanding: Virginia EC2 grading discrepancy and Oregon EC2 explicit IAM denial. Oregon application deployment is not validated.

Use this checklist in VS Code Interactive Checkboxes preview. Check each line when complete. The AWS Academy lab guide is authoritative for requirements; challenge prompts intentionally leave some implementation decisions to you.

## Capture in private Notepad

- [ ] LabIDEURL (from AWS Details)
- [ ] LabIDEPassword (private; never put in GitHub)
- [ ] Default AWS Region
- [ ] Generated S3 bucket name
- [ ] Static website URL
- [ ] CodeCommit clone URL
- [ ] Network stack outputs: subnet ID and VPC ID
- [ ] First EC2 public IPv4 and café URL
- [ ] S3 repository bucket name containing repobucket
- [ ] Oregon EC2 public IPv4 and café URL
- [ ] Answers to lab questions 1–6 (keep in lab question tab)

## Start lab and open IDE — Steps 1–7

- [ ] 1. In AWS Academy, choose Start Lab; wait for the AWS status circle to turn green.
- [ ] 2. Choose the green AWS status circle to open the AWS Management Console.
- [ ] 3. Arrange the console and lab instructions side by side.
- [ ] 4. Open AWS Details.
- [ ] 5a. Copy LabIDEURL into private Notepad.
- [ ] 5b. Copy LabIDEPassword into private Notepad.
- [ ] 6. Open LabIDEURL in a new tab.
- [ ] 7a. Enter LabIDEPassword in the code-server welcome prompt.
- [ ] 7b. Choose Submit; locate the IDE file browser, editor, and Bash terminal.

## Challenge 1 — Static S3 website

### Create the initial template — Steps 8–15

- [ ] 8. In the lab IDE, create and save S3.yaml in /home/ec2-user/environment.
- [ ] 9. Add the template format and Description fields.
- [ ] 10. Add the S3 bucket resource with correct YAML indentation.
- [ ] 11. Set Description to cafe S3 template.
- [ ] 12. Save S3.yaml.

Paste into the lab IDE editor as the initial complete S3.yaml:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: cafe S3 template
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
```

- [ ] 13a. In the lab IDE Bash terminal, check the default AWS Region; record it.
- [ ] 13b. Create the CreateBucket stack from S3.yaml.

```bash
cd /home/ec2-user/environment
aws configure get region
aws cloudformation create-stack --stack-name CreateBucket --template-body file://S3.yaml
```

Success: create-stack returns a StackId. Do not change the default Region.

- [ ] 14. In CloudFormation, inspect CreateBucket: Events, Resources, Outputs, and Template.
- [ ] 15. In S3, find the generated createbucket-s3bucket-... bucket and record its exact name.

### Answer initial questions — Steps 16–18

- [ ] 16. Open AWS Details in the lab instructions.
- [ ] 17. Open Access the multiple choice questions; leave this tab open.
- [ ] 18a. Answer Q1: whether a bucket was created without specifying a name, and its actual name.
- [ ] 18b. Answer Q2: the bucket's Region and why it was used.
- [ ] 18c. Answer Q3: number of lines in the Resources section needed to define the bucket.

Grader-verified answers: Q1 option 3 (stack name, logical ID, random suffix); Q2 option 2 (Region of create-stack); Q3 option 1 (two lines inside Resources). Submit each answer individually.

### Upload website assets and update stack — Steps 19–29

- [ ] 19a. In the lab IDE terminal, download the website ZIP and unzip it into static.

```bash
cd /home/ec2-user/environment
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/15-lab-mod11-challenge-CFn/s3/static-website.zip
unzip static-website.zip -d static
cd static
```

- [ ] 19b. Replace the placeholder below with the bucket name captured in Step 15, then configure ownership controls.

```bash
aws s3api put-bucket-ownership-controls \
  --bucket "BUCKET-NAME" \
  --ownership-controls 'Rules=[{ObjectOwnership=BucketOwnerPreferred}]'
```

- [ ] 19c. Configure the bucket's public access block settings as the lab requires.

```bash
aws s3api put-public-access-block \
  --bucket "BUCKET-NAME" \
  --public-access-block-configuration 'BlockPublicAcls=false,RestrictPublicBuckets=false,IgnorePublicAcls=false,BlockPublicPolicy=false'
```

- [ ] 19d. Upload the website assets from the static directory.

```bash
aws s3 cp --recursive . s3://BUCKET-NAME/ --acl public-read
```

Success: terminal shows upload messages. If denied by account-level restrictions, capture the exact error before changing settings.

- [ ] 20. Open the AWS Resource and Property Types Reference documentation.
- [ ] 21. Select Amazon S3 in the service resource type list.
- [ ] 22. Open AWS::S3::Bucket.
- [ ] 23a. Update S3.yaml to retain the bucket on stack deletion.
- [ ] 23b. Configure static website hosting with index.html as the index document.
- [ ] 24. Add a CloudFormation Output containing the website URL. Use the AWS documentation examples; this is a challenge task.
Use this complete S3.yaml in the editor (reference implementation; exact previous file not retained):

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: cafe S3 template
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      WebsiteConfiguration:
        IndexDocument: index.html
Outputs:
  WebsiteURL:
    Value: !GetAtt S3Bucket.WebsiteURL
```

- [ ] 25. Save S3.yaml in the lab IDE editor.
- [ ] 26. Return to the template directory and validate S3.yaml.

```bash
cd /home/ec2-user/environment
aws cloudformation validate-template --template-body file://S3.yaml
```

Success: validation returns template information without a ValidationError. Correct and revalidate any YAML issues before continuing.

- [ ] 27. Update CreateBucket from the saved template.

```bash
aws cloudformation update-stack --stack-name CreateBucket --template-body file://S3.yaml
```

- [ ] 28a. In CloudFormation, refresh CreateBucket until status is UPDATE_COMPLETE.
- [ ] 28b. If the update fails, inspect the first relevant UPDATE_FAILED event and its Status reason; fix the template and retry only after identifying the cause.
- [ ] 29a. Open the Outputs tab and verify the website URL output.
- [ ] 29b. Open the URL and verify the static café website loads.
- [ ] 29c. If needed, confirm the S3 bucket contains index.html, css/, and images/.
- [ ] 29d. If needed, check object public-read permissions and bucket Properties > Static website hosting.

## Challenge 2 — CodeCommit version control

### Clone the supplied repository — Steps 30–34

- [ ] 30. In CodeCommit, find the existing CFTemplatesRepo repository.
- [ ] 31. Open CFTemplatesRepo > Clone URL > Clone HTTPS (GRC); capture the URL.
- [ ] 32. Return to the lab IDE.
- [ ] 33. From /home/ec2-user/environment, replace CODECOMMIT-CLONE-URL with the copied URL and clone the repository.

```bash
cd /home/ec2-user/environment
git clone "CODECOMMIT-CLONE-URL"
```

- [ ] 34. Enter the repository and check Git status.

```bash
cd /home/ec2-user/environment/CFTemplatesRepo
git status
```

Success: working tree is clean. This is the lab's CodeCommit repository, not your personal GitHub portfolio.

## Challenge 3 — Automated network and application deployments

### Network stack via pipeline — Steps 35–54

- [ ] 35. In the lab IDE, expand CFTemplatesRepo/templates.
- [ ] 36. Duplicate template1.yaml in the same templates directory.
- [ ] 37. Name the copy cafe-network.yaml.
```bash
cd /home/ec2-user/environment/CFTemplatesRepo
cp -i templates/template1.yaml templates/cafe-network.yaml
ls templates
```

- [ ] 38a. Set its description to Network layer for the cafe and save.
- [ ] 38b. Review the seven resources defined in the template.
- [ ] 39. Open CodePipeline in the AWS Console.
- [ ] 40. Choose Pipelines; locate CafeAppPipeline and CafeNetworkPipeline.
- [ ] 41. Open CafeNetworkPipeline; inspect its Source action.
- [ ] 42. Choose View details for SourceAction.
- [ ] 43. Open Configuration; confirm the source repository is CFTemplatesRepo.
- [ ] 44. Choose Done to return to the pipeline.
- [ ] 45. Inspect the Deploy stage and its CloudFormation action.
- [ ] 46. Open RunChangeSet > View details; confirm stack update-cafe-network and template cafe-network.yaml.
- [ ] 47. Return to the lab IDE.
- [ ] 48. Run git status; confirm cafe-network.yaml is untracked.

```bash
cd /home/ec2-user/environment/CFTemplatesRepo
git status
```

- [ ] 49. Add and commit the network template.

```bash
git add templates/cafe-network.yaml
git commit -m 'initial commit of network template' templates/cafe-network.yaml
```

- [ ] 50. Run git status; confirm the branch is ahead of origin/main by one commit.
- [ ] 51. Push to CodeCommit.

```bash
git status
git push
```

- [ ] 52a. In CodePipeline, watch CafeNetworkPipeline Source and Deploy stages.
- [ ] 52b. Confirm both stages succeed and reference the new commit.
- [ ] 52c. If a stage fails, inspect execution details before retrying; avoid deleting stacks unless the error state requires it.
- [ ] 53a. In CloudFormation, verify update-cafe-network is CREATE_COMPLETE or UPDATE_COMPLETE.
- [ ] 53b. Inspect Outputs; no outputs are expected yet.
- [ ] 54. In VPC, verify Cafe VPC and Cafe Public Subnet exist.

### Export network values — Steps 55–58

- [ ] 55a. Append the exact Outputs section below to templates/cafe-network.yaml.
- [ ] 55b. Save the edited file before committing.

```yaml
Outputs:
  PublicSubnet:
    Description: The subnet ID to use for public web servers
    Value:
      Ref: PublicSubnet
    Export:
      Name:
        'Fn::Sub': '${AWS::StackName}-SubnetID'
  VpcId:
    Description: The VPC ID
    Value:
      Ref: VPC
    Export:
      Name:
        'Fn::Sub': '${AWS::StackName}-VpcID'
```

Do not add a second Outputs key if one already exists; inspect the template before pasting.

- [ ] 56. Commit and push the updated network template.

```bash
cd /home/ec2-user/environment/CFTemplatesRepo
git add templates/cafe-network.yaml
git commit -m 'export network stack outputs'
git push
```

- [ ] 57. Verify CafeNetworkPipeline updates the stack successfully.
- [ ] 58a. Verify CloudFormation Outputs includes PublicSubnet, exported as update-cafe-network-SubnetID.
- [ ] 58b. Verify Outputs includes VpcId, exported as update-cafe-network-VpcID; capture both values.

### Application stack via pipeline — Steps 59–72

- [ ] 59. Duplicate templates/template2.yaml and rename the copy cafe-app.yaml.
```bash
cd /home/ec2-user/environment/CFTemplatesRepo
cp -i templates/template2.yaml templates/cafe-app.yaml
ls templates
```

- [ ] 60a. Review LatestAmiId and CafeNetworkParameter in Parameters.
- [ ] 60b. Review RegionMap in Mappings.
- [ ] 60c. Review the existing CafeSG security group and WebServerPublicIP output.
- [ ] 60d. Correct the starter AMI default to Amazon Linux 2023 before deployment.

```bash
sed -i 's|amzn2-ami-hvm-x86_64-gp2|al2023-ami-kernel-default-x86_64|g' templates/cafe-app.yaml
```

The original starter AMI was Amazon Linux 2, but the supplied script requires dnf. Actual logs confirmed dnf was missing; switching to AL2023 and redeploying restored the website.

- [ ] 61. Consult the AWS CloudFormation documentation example for defining a template parameter.
- [ ] 62a. Add an instance-type parameter with allowed values t2.micro, t2.small, t3.micro, t3.small.
- [ ] 62b. Set its default to t2.small and give it an accurate description.
- [ ] 62c. Save cafe-app.yaml.
Insert under Parameters (description added to meet Step 62; not separately deployed):

```yaml
  InstanceTypeParameter:
    Type: String
    Default: t2.small
    Description: Choose t2.micro, t2.small, t3.micro, or t3.small.
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro
      - t3.small
```

- [ ] 63. Open the CloudFormation documentation for EC2 instance resource syntax.
- [ ] 64a. Define an EC2 resource with logical ID CafeInstance.
- [ ] 64b. Set ImageId to reference LatestAmiId.
- [ ] 64c. Set InstanceType to reference the parameter from Step 62.
- [ ] 64d. Set KeyName to use RegionMap as specified in the complete resource below.

- [ ] 64e. Set IamInstanceProfile to CafeRole.
- [ ] 64f. Configure a network interface with public IPv4, imported subnet ID, and CafeSG as specified in the complete resource below.

- [ ] 64g. Add a Name tag with value Cafe Web Server.
- [ ] 64h. Add the lab-provided UserData script inside CafeInstance Properties; preserve YAML indentation.

The supplied script successfully deployed the Virginia website after the AL2023 AMI correction. Oregon remains unvalidated.

For faster execution, insert the following complete resource under Resources, alongside CafeSG, then verify each of the preceding configuration items. Do not duplicate CafeInstance.

```yaml
  CafeInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref InstanceTypeParameter
      KeyName: !FindInMap [RegionMap, !Ref "AWS::Region", keypair]
      IamInstanceProfile: CafeRole
      NetworkInterfaces:
        - DeviceIndex: '0'
          AssociatePublicIpAddress: 'true'
          SubnetId: !ImportValue
            'Fn::Sub': '${CafeNetworkParameter}-SubnetID'
          GroupSet:
            - !Ref CafeSG
      Tags:
        - Key: Name
          Value: Cafe Web Server
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash
            dnf -y update
            dnf install -y mariadb1011-server httpd php8.4 php8.4-fpm php8.4-mysqlnd wget unzip
            systemctl enable --now php-fpm
            systemctl enable httpd
            systemctl start httpd
            systemctl enable mariadb
            systemctl start mariadb
            wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/15-lab-mod11-challenge-CFn/s3/cafe-app.sh
            chmod +x cafe-app.sh
            ./cafe-app.sh
```

- [ ] 65. Save the completed cafe-app.yaml.
- [ ] 66. Validate the application template from the lab IDE Bash terminal.

```bash
aws cloudformation validate-template --template-body file:///home/ec2-user/environment/CFTemplatesRepo/templates/cafe-app.yaml
```

Success: JSON output includes the three template parameters. If validation fails, correct the saved YAML and rerun validation.

- [ ] 67. Add, commit, and push cafe-app.yaml to CodeCommit.

```bash
cd /home/ec2-user/environment/CFTemplatesRepo
git add templates/cafe-app.yaml
git commit -m 'add cafe application template'
git push
```

Note: The original guide jumps from Step 67 to Step 69; it contains no Step 68.

- [ ] 69a. Open CafeAppPipeline and watch Source and Deploy complete.
- [ ] 69b. If deployment fails, inspect pipeline execution details and the first CloudFormation failure event before retrying.
- [ ] 70. Verify update-cafe-app stack has CREATE_COMPLETE status.
- [ ] 71a. In EC2, verify the instance and security group were created.
- [ ] 71b. Wait until the instance passes both status checks.
- [ ] 72a. Capture the EC2 public IPv4 address; open the café application.

```text
http://<PUBLIC-IPV4>/cafe
```

- [ ] 72b. Allow time for UserData initialization; verify the café page and its Region/Availability Zone details.

If the site fails, check the current stack parameters and outputs before reusing an IP:

```bash
aws cloudformation describe-stacks \
  --stack-name update-cafe-app --region us-east-1 \
  --query 'Stacks[0].{Status:StackStatus,Parameters:Parameters,Outputs:Outputs}' \
  --output json --no-cli-pager
```

For a failed HTTP connection, inspect the current instance system log (replace INSTANCE_ID):

```bash
aws ec2 get-console-output --region us-east-1 \
  --instance-id INSTANCE_ID --query Output --output text --no-cli-pager \
  | grep -Ei 'dnf|command not found|failed|not found'
```

Observed failure: dnf, httpd and mysql were unavailable with the original AL2 AMI. The AL2023 correction above resolved the Virginia website. If correcting an existing stack, commit/push, wait for the pipeline, confirm the new AMI parameter and capture the replacement public IP. Do not assume a successful stack means UserData completed.

### Answer application questions — Step 73

- [ ] 73a. Answer Q4 using LatestAmiId in the update-cafe-app stack Parameters tab.
- [ ] 73b. Answer Q5 using the stack's IAM role ARN in Stack info.
- [ ] 73c. Answer Q6 by opening a CodeCommit commit ID and observing its contents.

Grader-verified answers: Q4 option 4 (AL2023 SSM parameter path); Q5 option 1 (CustomCFnRole); Q6 option 3 (line-level changes in commit details). Submit each answer individually.

## Challenge 4 — Duplicate infrastructure in Oregon

### Second Region — Steps 74–92

- [ ] 74. From the lab IDE, create the network stack in us-west-2 using the existing network template.

```bash
aws cloudformation create-stack \
  --stack-name update-cafe-network \
  --template-body file:///home/ec2-user/environment/CFTemplatesRepo/templates/cafe-network.yaml \
  --region us-west-2
```

Success: a StackId is returned.

- [ ] 75a. In CloudFormation, switch to US West (Oregon), us-west-2.
- [ ] 75b. Refresh until update-cafe-network reaches CREATE_COMPLETE.
- [ ] 76. In the Oregon VPC console, confirm the network resources were created.
- [ ] 77. Open the EC2 console and confirm Region us-west-2.
- [ ] 78. Choose Key Pairs in EC2 navigation.
- [ ] 79. Choose Create key pair.
- [ ] 80. Enter the key pair name cafe-oregon.
- [ ] 81. Choose Create key pair. The guide says the private key is not needed for this lab; handle any downloaded key securely.
- [ ] 82a. Reopen cafe-app.yaml in the lab IDE.
- [ ] 82b. Verify RegionMap maps us-east-1 to vockey and us-west-2 to cafe-oregon.
- [ ] 82c. Confirm the instance type parameter defaults to t2.small, but can be overridden.
- [ ] 83a. In S3, identify the actual bucket whose name contains repobucket; record it.
```bash
aws s3 ls | grep repobucket
```

- [ ] 83b. From the CFTemplatesRepo directory, replace REPOBUCKET-BUCKETNAME with the captured bucket name and upload the template.

```bash
cd /home/ec2-user/environment/CFTemplatesRepo
aws s3 cp templates/cafe-app.yaml s3://REPOBUCKET-BUCKETNAME/
```

- [ ] 84. In S3, select the uploaded cafe-app.yaml object and copy the Object URL from Properties (not the S3 URI).
- [ ] 85. In CloudFormation, confirm Region is Oregon (us-west-2).
- [ ] 86. Choose Create stack > With new resources (standard).
- [ ] 87. Supply the copied Amazon S3 object URL as the template source.
- [ ] 88. Choose Next.
- [ ] 89a. Enter update-cafe-app (name used during execution).
- [ ] 89b. Set InstanceType to t3.micro.
- [ ] 90. Continue through the remaining stack creation screens using the lab's defaults and submit the stack.
- [ ] 91. Verify the Oregon application stack reaches CREATE_COMPLETE.
- [ ] 92a. In Oregon EC2, inspect the created instance and its key pair and instance type.
- [ ] 92b. Wait for both instance status checks and application initialization.
- [ ] 92c. Capture Oregon's public IPv4; load the café website.

```text
http://<OREGON-PUBLIC-IPV4>/cafe
```

- [ ] 92d. Verify the website reports us-west-2; compare with the first website in us-east-1.

### Oregon deployment blocker — observed, unresolved

The Oregon network and key pair were created successfully. The application stack failed on CafeInstance with an explicit identity-based policy deny for ec2:RunInstances on an EBS volume resource. The AL2023 AMI inspection showed an 8 GiB gp3 root volume; whether gp3 caused the denial is unconfirmed. Do not change IAM policies or blindly retry. Inspect stack events and the template's storage settings before considering any permitted correction. Steps 91–92 remain unvalidated; submit partial work if blocked.

## Submit and end lab — Steps 93–96

- [ ] 93. In AWS Academy, choose Submit and confirm Yes.
- [ ] 94. If results do not appear after a few minutes, open Grades.
- [ ] 95. Open Submission Report and inspect detailed feedback; record any incomplete objectives.
- [ ] 96. When grading and evidence capture are complete, choose End Lab and confirm Yes.

## Validation record (not execution steps)

- Documentation reviewed against original guide, Steps 1–96; Step 68 is absent in the source.
- Actual AWS execution: static site, network and application pipelines, Virginia website after AMI correction, Oregon network and key pair succeeded.
- Grader: 24/30; all six questions correct. Task 6B R1 EC2 and Task 7 R2 instance each scored 0/3. All other objectives received full credit.
- Virginia EC2 grading cause unknown; do not claim the grader is defective. Oregon IAM denial unresolved.
- No required screenshot in the lab guide; separate college submission requirements must be checked independently.
- End Lab completion not confirmed. If resuming, check whether the original lab session and resources still exist before executing anything.
- Keep this canonical Markdown file; update it when remaining objectives are resolved.
