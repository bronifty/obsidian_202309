![](./media/aws_fundamentals_1.png)

### ARN
```
arn:<partition>:<service>:<region>:<account-id>:<resource-id>
arn:aws:s3:::my-bucket
arn:aws:dynamodb:us-east-1:123456789012:table/my-table

s3 arn:aws:s3:::my-bucket
ec2 arn:aws:ec2:us-east-1:123456789012:instance/i-012345abcdef
rds arn:aws:rds:us-east-1:123456789012:db:mydatabase
dynamo arn:aws:dynamodb:us-east-1:123456789012:table/mytable
```
- arn's are used in iam and iac templates and service apis; they enable you to specify and authorize access to resources in a secure and standardized manner
	- For example, the ARN for an Amazon DynamoDB table includes the table name (mytable) after the table/ prefix. This allows you to uniquely identify the table and authorize access to it using IAM policies.

### IAM v SSO
- a root account in IAM is an analog to a management account in SSO for an Organization in that it enables SSO (eg via Okta for all member accounts who login and have access to business apps and apis) and manages permission via Identity Center via a resource hierarchy
- Identity Center for an Org in SSO is an analog to IAM for a root account 

- root account is like IE -whose only purpose is to download Chrome- in that it is meant only for track billing and to create a new account for provisioning resources and interacting with their services
- create a budget with an alarm to your email with the root account 
	- CloudWatch* metric can send alarms to SNS topic to which you can subscribe with text or email as well
- create a user account to provision and interact with services
- 

*Cloudwatch. every service reports its metrics and logs to Cloudwatch (it's a central logging utility for AWS). Log event/stream/group. Log insights to isolate a specific app with SQL-like query. Lambda function named thumbnail in group /aws/lambda/thumbnail. To be able to query logs efficiently you need to use a structured logger (JSON logger).


### AWS Orgs
- global service allows to manage multiple accounts
- main acct is mgt; others are member accts
- Org:member account is 1:M (member can only be a member of 1 org)
- consolidated billing for all accts (single payment method, volume discount)
![](./media/aws_fundamentals_2.png)
#### Advantages of Orgs / OUs
- use org and OU to enforce tagging standards for billing 
- enable CloudTrail on all accounts send logs to central S3 bucket
- send CloudWatch logs to central logging account
- administer RBAC for all member accts from the root/mgt acct
#### Security: Service Control Policies (SCP)
- IAM policies applied to OU or Accounts to restrict Users and Roles
- Mgt account exempted
- Requires explicit allow like IAM

aws:PrincipalOrgID condition to allow access to a resource from any account
- allow from this org id (predicate)
```yaml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::example-bucket",
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalOrgID": "o-123456789012"
                }
            }
        }
    ]
}
```

### Tag Policies
- enforce tagging / standardize tagging
- cost allocation / abac
- CloudWatch to monitor tagged resources or resources without tags

