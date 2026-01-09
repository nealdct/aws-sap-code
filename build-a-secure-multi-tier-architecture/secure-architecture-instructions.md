# Part 1 - Create Security Groups, KMS Key, ACM Certificate, and SNS Topic

## Create Security Groups

Name: ALBFrontEnd
Inbound rule: HTTPS from anywhere IPv4
Outbound rule: HTTP to the EC2BackEnd SG

Name: EC2BackEnd
Inbound rule: HTTP from the ALBFrontEnd SG
Outbound rule: All traffic to 0.0.0.0/0 (for downloading installation files)

## AWS KMS

1. Create a Symmetric key called MyDataKey
2. Set your individual account as the key administrator
3. For key usage set your individual account and 'AWSServiceRoleForAutoScaling'

## AWS Certificate Manager

1. Request a public certificate
2. Add your domain name, e.g. dctlabs.com and add another name to the certificate for alb.dctlabs.com (replacing with your domain name)
3. Use DNS validation option and submit the validation to create the records in Route 53

## Create an SNS Topic

1. Create a standard topic called "MyNotification"
2. Create an email subscription for your email address

# Part 2 - Create VPC infra, IAM Role, S3 Bucket, and Launch Template

## Create VPC infrastructure

- Create two private subnets in us-east-1 default VPC

Name: Private-1A
CIDR: 172.31.96.0/20
AZ: us-east-1a

Name: Private-1B
CIDR: 172.31.112.0/20
AZ: us-east-1b

- Create a route table

Name: PrivateRT
Associations: the two private subnets

- Create a NAT gateway

Name: my-nat-gw
Subnet: Public subnet in the default VPC
Routes: Add a route to the private route table

## Create IAM Role

Name: ec2-ssm-s3
Use Case: EC2
Permissions policies:
- 'AmazonS3ReadOnlyAccess'
- 'AmazonSSMManagedInstanceCore'

## Create S3 Bucket

Name: source-files-ec2-webapp (add numbers/characters to make it unique)
Objects: Add the following objects from the Code > build-a-secure-multi-tier-architecture > Part-2 directory:
- index.html
- hw.css
- hw.jpeg

## Create a Launch Template

1. Add your bucket name to the user-data-az.md file from the Part-2 directory
2. Create a launch template named MySecureLT
3. Choose the Amazon Linux 2023 AMI
4. Select a t2.micro instance type
5. Don't include a key pair
6. Choose the EC2BackEnd security group
7. Encrypt the EBS volume using the KMS key created earlier
8. Select the 'ec2-ssm-s3' instance profile
9. Paste the user data (make sure bucket name is updated)

# Part 3 - Create Auto Scaling Group, ALB, and CloudFront Distribution

## Create ASG

1. Call it "ASG1"
2. Select the launch template created earlier
3. Select the two private subnets in the default VPC
4. Set group size values to 2 for each option

## Create an ALB

1. Call it "ALB1"
2. Make it internet facing
3. Select the public subnets in us-east-1a and us-east-1b in the default VPC
4. Select the "ALBFrontEnd" security group
5. Configure an HTTPS listener with the public certificate from ACM
6. Forward to a new TG called "TG1"
7. Attach the TG to the ASG

## Create a CloudFront Distribution

1. Select the ALB
2. Set the origin domain to your ALB subdomain, e.g. alb.dctlabs.com
3. Set protocol  to "HTTPS only"
4. Set cache policy to "CachingDisabled"
5. Do not protect with AWS WAF
6. Select the SSL/TLS certificate from ACM
7. Add the apex domain name in the alternate domain name (CNAMEs) field, e.g. dctlabs.com
8. Add the index.html as the default root object


## Create records in Route 53

- Create two records

Name: apex, e.g. dctlabs.com
Type: A
Value/Route traffic to: CloudFront distribution

Name: ALB subdomain, e.g. alb.dctlabs.com
Type: A
Value/Route traffic to: ALB

## Configure Custom Header

1. Edit CloudFront origin
2. For "Origin Custom Headers" add the following header:
- Header Name: X-Custom-Header
- Value: value-123456

## Configure conditional forwarding rule in ALB

1. Add a rule to the listener
2. Use "http header" and set the header name and value
3. Forward to TG1
4. Edit the last rule with a fixed response that states "Access Denied"

# Part 4 - Enable logging, configuration management, and security inspection

## Enable logging for the ALB

1. Create an S3 bucket for logging
2. Add the following bucket policy (modify ONLY the BUCKET-NAME and YOUR-ACCOUNT-ID fields)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::127311923021:root"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::BUCKET-NAME/AWSLogs/YOUR-ACCOUNT-ID/*"
    }
  ]
}
```

3. Edit load balancer attributes
4. Enable access logs
5. Select the S3 bucket and save

## Enable logging for the CloudFront distribution

1. Edit the distrubition
2. Enable standard logging
3. Configure the same bucket as the ALB logging
4. Enable ACLs
5. Specify the prefix "cloudfront"

## Configure rules in AWS Config

1. Create two rules in AWS Config using the following managed rules:

- 's3-bucket-logging-enabled'
- 'alb-waf-enabled'

2. Configure AWS Config to send notifications to the SNS Topic created earlier

## Setup Inspector using Systems Manager

1. In SSM Run Command check that the inspector agent updates have run
2. Go to Amazon Inspector and specify your account ID for delegated administrator account and activate the trial
3. Check the findings for the EC2 instances

# Part 5 - Add AWS WAF

## Create an AWS WAF WebACL

1. Create a WebACL

Name: MyWebACL
Resource type: CloudFront distributions
Associated resources: select the distribution

2. Create a rule for the WebACL

Name: Rate100
Type: Rate-based rule
Rate limit: 100
Action: Block

3. Use the following command on CloudShell to trip the WAF rule (modify the domain name)

```bash
for i in {1..140}; do curl https://YOUR-DOMAIN-NAME/; done
```




