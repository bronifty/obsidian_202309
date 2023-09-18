![](./media/aws_fundamentals_1.png)

### IAM
- root account is like IE -whose only purpose is to download Chrome- in that it is meant only for track billing and to create a new account for provisioning resources and interacting with their services
- create a budget with an alarm to your email with the root account 
	- CloudWatch* metric can send alarms to SNS topic to which you can subscribe with text or email as well
- create a user account to provision and interact with services
- 

*Cloudwatch. every service reports its metrics and logs to Cloudwatch (it's a central logging utility for AWS). Log event/stream/group. Log insights to isolate a specific app with SQL-like query. Lambda function named thumbnail in group /aws/lambda/thumbnail. To be able to query logs efficiently you need to use a structured logger (JSON logger).

