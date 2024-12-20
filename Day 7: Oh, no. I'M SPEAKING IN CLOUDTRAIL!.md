By using JQ—a powerful command-line tool for processing JSON data—I was able to sift through large volumes of log data efficiently, uncovering the crucial events that led to the problem.

## Background

The application in question was experiencing sporadic downtime, which seemed to occur without any clear pattern. The first thing I did was start looking at logs to gather as much information as possible. AWS CloudWatch and CloudTrail were the two most obvious places to search for clues.

- **AWS CloudWatch Logs**: These are essential for monitoring the health and performance of AWS resources. In particular, CloudWatch provided the application logs that showed error messages, performance metrics, and other insights related to the app’s state.
  
- **AWS CloudTrail**: This service records API calls made within your AWS account. Every time a change is made—whether it’s an instance stop or a policy modification—CloudTrail logs it. This information is invaluable for tracing the sequence of events that could have led to a problem.

Given the intermittent nature of the downtime, I knew that I would need to filter out unnecessary noise from these logs and focus only on the most relevant events. That's when I decided to use JQ to help with the filtering and analysis of the logs.

## Step 1: Retrieve CloudTrail Logs

The first step in my investigation was to obtain the necessary CloudTrail logs. AWS CloudTrail logs are often stored in an S3 bucket, and I used the AWS CLI to download them. Here’s the command I ran to fetch all the logs related to the timeframe I was interested in:

```bash
aws s3 cp s3://<bucket-name>/<log-prefix>/ ./ --recursive
```

This command recursively downloads all logs from the specified S3 bucket to my local machine, which I could then analyze. The CloudTrail logs were in JSON format, and the files were typically organized by date. Each file contained a series of log records, detailing every API call made within that period.

Once I had the logs, I was able to open them and begin analyzing the JSON structure. Each log entry contains fields such as `eventTime`, `eventSource`, `eventName`, and `userIdentity`, which gave me useful insights into what actions were taken at specific times.

## Step 2: Filter Relevant Events Using JQ

The challenge was sifting through the massive volume of CloudTrail logs to find relevant events. CloudTrail logs can be quite verbose, and you might end up with thousands of records in a single log file. To filter out the noise, I turned to JQ.

JQ allows you to apply powerful filters to JSON data, enabling you to extract only the information you need. In my case, I was particularly interested in events related to EC2 instances, as the downtime seemed to correlate with instance behavior.

For example, I used this JQ command to filter CloudTrail events related to stopping EC2 instances:

```bash
jq '.Records[] | select(.eventSource=="ec2.amazonaws.com" and .eventName=="StopInstances")' cloudtrail-log.json
```

This command did the following:

- `.Records[]`: Iterates over all the records in the CloudTrail log.
- `select(.eventSource=="ec2.amazonaws.com" and .eventName=="StopInstances")`: Filters out events where the source is EC2 and the event is related to stopping instances.

By running this command, I was able to extract only those events that involved stopping EC2 instances. This was particularly useful because I suspected that an instance stop might have contributed to the downtime.

I also used similar filters to look for other types of events, such as instance starts, reboots, or any changes to security groups or auto-scaling policies, which could also potentially explain the downtime.

## Step 3: Analyze CloudWatch Logs

While CloudTrail logs provided insights into the infrastructure-level events (like stopping EC2 instances), I needed more context to understand the application’s behavior during the downtime. This is where CloudWatch Logs came in.

CloudWatch Logs provide detailed records of what’s happening inside your application. This includes error logs, performance metrics, and other system logs that give you a view into how your resources are behaving. I used the AWS CLI to pull logs from CloudWatch within a specific time range:

```bash
aws logs filter-log-events --log-group-name "/aws/lambda/my-function" --start-time <start-time> --end-time <end-time>
```

This command allowed me to pull logs from a specific Lambda function, which was part of the application experiencing downtime. By focusing on the time range that matched the CloudTrail events, I could look at what was happening inside the application during the same period.

From these CloudWatch logs, I found several error messages indicating that the application was failing to connect to the database due to resource limitations. The errors were primarily related to database connection timeouts and request failures, which were triggered by a spike in traffic.

## Step 4: Correlating CloudWatch and CloudTrail Data

With both CloudWatch and CloudTrail data at hand, the next step was to correlate the two. This was crucial because the logs provided insights from two different layers of the stack: the infrastructure layer (CloudTrail) and the application layer (CloudWatch).

Here’s what I discovered:

1. **EC2 Instance Stop**: The CloudTrail logs showed that EC2 instances were being stopped unexpectedly during the time when the application was down.
2. **Application Errors**: At the same time, the CloudWatch logs revealed a spike in application errors, particularly connection timeouts, indicating that the application could no longer reach the database.

From this, it became clear that the root cause of the downtime was the unexpected stopping of EC2 instances, which in turn caused the application to fail due to resource unavailability.

## Step 5: Identifying the Root Cause

The next step was to determine why the EC2 instances were stopped in the first place. I reviewed the CloudTrail logs further and found that the stop events were triggered by an auto-scaling policy. The policy had been misconfigured to scale down the EC2 instances too aggressively during low traffic periods. As a result, the application was left without the necessary resources, causing the intermittent downtime.

The auto-scaling policy was intended to save costs by scaling down instances, but it was overly aggressive, especially during periods of high application load. This misconfiguration led to the failure, and the application could not maintain its usual performance.

## Step 6: Resolution and Conclusion

Once I identified the misconfigured auto-scaling policy, I worked with the infrastructure team to update the scaling configuration. We adjusted the parameters to ensure that the application would retain the necessary resources during peak times and avoid unnecessary stops during periods of moderate traffic.

The downtime was resolved once the scaling policy was adjusted, and I monitored the system over the next few days to ensure that the issue did not recur.

### So...

1. **CloudWatch and CloudTrail**: These two AWS services are incredibly powerful when used together. CloudTrail gives you visibility into infrastructure changes, while CloudWatch provides insight into application behavior. By combining both sets of logs, you can gain a complete picture of what's happening in your AWS environment.
   
2. **JQ for Log Processing**: JQ is a game-changer for analyzing large sets of JSON data. With the right filters, you can extract the exact information you need without manually sifting through massive log files.

3. **Auto-Scaling Misconfigurations**: Always review your auto-scaling policies to ensure they are aligned with your application’s actual needs. Over-aggressive scaling can lead to performance degradation and unexpected downtime.

So basically, by leveraging AWS CloudWatch, CloudTrail, and JQ, I was able to quickly identify and resolve an issue that could have had a significant impact on the application’s uptime. The combination of detailed logs and powerful tools like JQ allowed me to pinpoint the root cause with confidence and resolve the issue swiftly.

**Answers**

*What is the other activity made by the user glitch aside from the ListObject action?* **PutObject**
 
*What is the source IP related to the S3 bucket activities of the user glitch?* **53.94.201.69**
 
*Based on the eventSource field, what AWS service generates the ConsoleLogin event?* **signin.amazonaws.com**
 
*When did the anomalous user trigger the ConsoleLogin event?* **2024-11-28T15:21:54Z**
 
*What was the name of the user that was created by the mcskidy user?* **glitch**
 
*What type of access was assigned to the anomalous user?* **AdministratorAccess**
 
*Which IP does Mayor Malware typically use to log into AWS?* **53.94.201.69**
 
*What is McSkidy's actual IP address?* **31.210.15.79**
 
*What is the bank account number owned by Mayor Malware?* **2394 6912 7723 1294**
