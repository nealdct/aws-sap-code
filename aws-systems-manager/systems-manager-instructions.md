# Launch EC2 Managed Instances

## Create role for Systems Manager
- EC2 use case
- Permissions: `AmazonSSMManagedInstanceCore` policy
- Name: SSMInstanceProfile

1. Launch one Windows and one Linux instance using the IAM role
2. Add a tag to each instance: Environment | Production
3. Check Systems Manager agent version (on Linux): 

```bash
yum info amazon-ssm-agent
```

## Configure Default Host Management

- Agent version must be 3.2.582.0 of later
- IMDSv2 must be enabled

1. In Fleet Manager choose the Account Management dropdown
2. Turn on Enable Default Host Management Configuration

# Systems Manager Automation

1. In Systems Manager click "Documents"
2. Search for "AWS-StopEC2Instance"
3. Choose "Execute automation" and select the "Simple execution"
4. Select both instances manually and execute the automation
5. Monitor the progress and ensure the instances are stopped
6. Find and run the "AWS-StartEC2Instance" document

# Systems Manager Run Command and Patch Manager

## Run Command

1. In Systems Manager click "Run Command"
2. Search for the command "AWS-FindWindowsUpdates"
3. Change "Update level" to "All"
4. For target selection, use tags and enter the tag added to the instances
5. Run the command and monitor the progress. What happened to the Linux instance?
6. Search for and run "AWS-InstallMissingWindowsUpdates"
7. Select the Windows instance and run the command

## Patch Manager

1. Go to Patch Manager and click "Create patch policy"
2. Select "Scan and install"
3. Select "Current account" and enter the environment tag for the instances
4. Select an S3 bucket for the output
5. Create the policy
6. Monitor the tasks

# Systems Manager Configuration Compliance

1. In Systems Manager go to "Compliance" and check the compliance status of your instances
2. Update "INSTANCE_ID" below with one of your instance IDs

```bash
aws ssm put-compliance-items --resource-id INSTANCE_ID --resource-type ManagedInstance --compliance-type Custom:CorporateSoftware --execution-summary ExecutionTime=1687802409 --items Id=Version-2.0,Title=CorporateSoftware,Severity=CRITICAL,Status=NON_COMPLIANT --region us-east-1
```

3. Run the command using AWS CloudShell

4. Refresh the Compliance console and view the updated status information

