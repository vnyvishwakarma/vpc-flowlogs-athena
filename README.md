# Introduction

Monitoring network traffic is a critical component of security best practices to meet compliance requirements, investigate security incidents, track key metrics, and configure automated notifications. AWS VPC Flow Logs captures information about the IP traffic going to and from network interfaces in your VPC. In this hands-on lab, we will set up and use VPC Flow Logs published to Amazon CloudWatch, create custom metrics and alerts based on the CloudWatch logs to understand trends and receive notifications for potential security issues, and use Amazon Athena to query and analyze VPC Flow Logs stored in S3.

## Solution Implementation 

Log in to the Cloud AWS Account.

Once inside the AWS account, make sure you are using eu-west-1 (ireland) as the selected region.

## Create CloudWatch Log Group and VPC Flow Logs to CloudWatch and S3

Create VPC Flow Log to S3

1. Navigate to VPC and select Your VPCs.
2. Select the Desired VPC.
3. At the bottom of the screen, select the Flow logs tab.
4. Click Create flow log, and set the following values:

  * Filter: All
  * Maximum aggregation interval: 1 minute
  * Destination: Send to an Amazon S3 bucket
  * S3 bucket ARN:

     1. Navigate to S3 in a new browser tab.
     2. Select the provided bucket (it should have vpcflowlogsbucket in its name).
     3. Click Copy ARN.
     4. Return to the VPC tab and paste in the value.

5. Leave the rest as their defaults and click Create flow log.
6. Select the Flow logs tab and verify the flow log shows an Active status.
7. In the S3 browser tab, click to open the bucket.
8. Click the Permissions tab.
9. Scroll down to Bucket policy.
10. Notice the bucket path in the policy includes AWSLogs.
Note: It can take 5–15 minutes before logs appear, so let's move on while we wait for that to happen.

## Create CloudWatch Log Group

* In a new browser tab, navigate to CloudWatch and select Log groups.
* Click Create log group.
* In Log group name, enter "VPCFlowLogs".
* Click Create.

## Create VPC Flow Log to CloudWatch

1. Back in the VPC browser tab, click Your VPCs in the left-hand menu.
2. Select the Desired VPC .
3. Select the Flow Logs tab.
4. Click Create flow log, and set the following values:
      * Filter: All
      * Maximum aggregation interval: 1 minute
      * Destination: Send to CloudWatch Logs
      * Destination log group: VPCFlowLogs
      * IAM role: Select the role with DeliverVPCFlowLogsRole in the name.

5. Leave the rest as their defaults and click Create flow log.
6. Under the Flow logs tab, verify the flow log shows an Active status.
7. In the CloudWatch browser tab, click the VPCFlowLogs log group to open it.

Note: It can take 5–15 minutes before logs start to show up, so let's move on while we wait for that to happen.

## Generate Traffic
In a new browser tab, navigate to EC2.
Under Resources on the EC2 dashboard, select Instances (running).
Select the provisioned Web Server instance.
At the bottom under Details, copy the public IPv4 address to the clipboard.
Open a terminal session and log in to the EC2 instance via SSH (the password is provided on the lab page):

ssh ec2_user@<PUBLIC_IP>
Exit the terminal:

### logout
Return to the EC2 dashboard.

With Web Server selected, click Actions > Security > Change security groups.
Under Associated security groups, click Remove to remove the attached security group.
In the search bar, search for and select the security group with HTTPOnly in the name.
Click Add security group.
Click Save.
Return to the terminal and attempt to connect to the EC2 instance again via SSH using the provided lab credentials.
Note: We expect this connection to time out since we just selected a security group with no SSH access.

After about 15 seconds, press Ctrl-C to cancel the SSH command.
Return to the EC2 dashboard.
With Web Server selected, click Actions > Security > Change security groups.
Under Associated security groups, click Remove to remove the HTTPOnly security group.
In search bar, search for and select the security group with HTTPAndSSH in the name.
Click Add security group.
Click Save.
Attempt to log in to the EC2 instance again via SSH using the credentials provided. This time, it should work.
Exit the terminal:

logout

Create CloudWatch Filters and Alerts

Create CloudWatch Log Metric Filter
Navigate to CloudWatch and select Log groups.
Select the VPCFlowLogs log group. You should now see a log stream.
Note: If you don't see a log stream listed yet, wait a few more minutes and refresh the page until the data appears.

Go back to the VPCFLowLogs page and select the Metric filters tab.
Click Create metric filter.
Enter the following in the Filter pattern field to track failed SSH attempts on port 22:

[version, account, eni, source, destination, srcport, destport="22", protocol="6", packets, bytes, windowstart, windowend, action="REJECT", flowlogstatus]
In the Select log data to test dropdown, select Custom log data.

Paste the following in the text box:

```sh

2 086112738802 eni-0d5d75b41f9befe9e 61.177.172.128 172.31.83.158 39611 22 6 1 40 1563108188 1563108227 REJECT OK
2 086112738802 eni-0d5d75b41f9befe9e 182.68.238.8 172.31.83.158 42227 22 6 1 44 1563109030 1563109067 REJECT OK
2 086112738802 eni-0d5d75b41f9befe9e 42.171.23.181 172.31.83.158 52417 22 6 24 4065 1563191069 1563191121 ACCEPT OK
2 086112738802 eni-0d5d75b41f9befe9e 61.177.172.128 172.31.83.158 39611 80 6 1 40 1563108188 1563108227 REJECT OK

```

Click Test pattern.

Click Next.
Set the following values:
Filter name: dest-port-22-rejects
Metric namespace: VPC Flow Logs
Metric name: ssh-rejects
Metric value: 1
Click Next.
Click Create metric filter.
Create Alarm Based on the Metric Filter
Once created, select the metric filter and click Create alarm.
Change Period to 1 minute.
Scroll down to Conditions and set Whenever SSHRejects is... to Greater/Equal than 1.
Click Next.
In the Notification section, set the following values:
Select an SNS topic: Create new topic
Create a new topic...: Default
Email endpoints that will receive the notification...: Your email address
Click Create topic.
Click Next.
In Alarm name, type "SSH Rejects" and click Next.
Click Create alarm.
Open your email inbox and click the Confirm Subscription link in the received SNS email.
Generate Traffic for Alerts
In the terminal, log in to the Web Server instance via SSH using the lab credentials.
Exit the terminal:

logout
In a new browser tab, navigate to EC2 and select Instances(running).

Select the Web Server instance.
Click Actions > Security > Change Security Groups.
Under Associated security groups, click Remove to remove the attached security group.
In the search bar, search for and select the HTTPOnly security group.
Click Add security group.
Return to the terminal and attempt to connect to the EC2 instance via SSH.
Note: We expect this to time out since we just selected a security group with no SSH access.

Press Ctrl-C to cancel the SSH command.
Return to EC2 dashboard.
With the Web Server instance still selected, click Actions > Security > Change Security Groups.
Click Remove to remove the HTTPOnly security group.
Select again the HTTPAndSSH security group and click Add security group.
Click Save.
Go back to CloudWatch > Alarms. We should see our SSHRejects alarm enter an In alarm state shortly.
Note: If you don't see the alarm listed yet, wait a few more minutes and refresh the page until it appears.

Use CloudWatch Insights
Navigate to CloudWatch and select Logs > Insights.
In the Select log group(s) search window, select VPCFlowLogs.
In the right-hand pane, select Queries.
Under Sample queries, click VPC Flow Logs > Top 20 source IP addresses with highest number of rejected requests.
Click Apply.

Observe the query changes.
Click Run query. After a few moments, we'll see some data start to populate.


## Analyze VPC Flow Logs Data in Athena

Record Reference Information to Be Used in Athena Queries

Note: Before attempting to run a query in Athena, you have to specify an S3 bucket for the results to be saved.

In a new browser tab, navigate to S3.
Select the provisioned bucket to open it.
Navigate through the bucket folder structure: AWSLogs > {ACCOUNT_ID} > vpcflowlogs > us-east-1 > {YEAR} > {MONTH} > {DAY}.
At the top right, click Copy S3 URI.
Create the Athena Table
Navigate to Athena.
If you come to the Amazon Athena landing page, click Get Started. Otherwise, skip this step.
Above the new query window, click set up a query result location in Amazon S3.
In Query result location, enter s3:// and paste in the S3 bucket path previously copied with a forward slash at the end (e.g., s3://{BUCKET_NAME}/AWSLogs/{ACCOUNT_ID}/vpcflowlogs/us-east-1/{YEAR}/{MONTH}/{DAY}/).
Click Save.


Paste the following DDL code in the new query window, replacing {your_log_bucket} and {account_id} with your unique values (You can obtain them from the bucket path you've been using.):

```sh

CREATE EXTERNAL TABLE IF NOT EXISTS default.vpc_flow_logs (
  version int,
  account string,
  interfaceid string,
  sourceaddress string,
  destinationaddress string,
  sourceport int,
  destinationport int,
  protocol int,
  numpackets int,
  numbytes bigint,
  starttime int,
  endtime int,
  action string,
  logstatus string
)
PARTITIONED BY (dt string)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ' '
LOCATION 's3://{your_log_bucket}/AWSLogs/{account_id}/vpcflowlogs/us-east-1/'
TBLPROPERTIES ("skip.header.line.count"="1");

```

Click Run query. Once executed, a Query successful message should display.

Create Partitions to Be Able to Read the Data
Click the + icon to open a new query window.
Paste the following code:

```sh

ALTER TABLE default.vpc_flow_logs
    ADD PARTITION (dt='{Year}-{Month}-{Day}')
    location 's3://{your_log_bucket}/AWSLogs/{account_id}/vpcflowlogs/us-east-1/{Year}/{Month}/{Day}';
Update the following variables with your unique values (You can obtain them from the bucket path you've been using.):

```

{your_log_bucket}
{account_id}
{Year}
{Month}
{Day}
Click Run query. A Query successful message should display.

Analyze VPC Flow Logs Data in Athena
Open a new query window and paste in the following:

```sh

SELECT day_of_week(from_iso8601_timestamp(dt)) AS
     day,
     dt,
     interfaceid,
     sourceaddress,
     destinationport,
     action,
     protocol
   FROM vpc_flow_logs
   WHERE action = 'REJECT' AND protocol = 6
   order by sourceaddress
   LIMIT 100;

```

Click Run query. Our formatted data should appear underneath.