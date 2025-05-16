# Build a Security Monitoring System in AWS

---

![Image](http://learn.nextwork.org/ecstatic_beige_calm_tarapirohe/uploads/aws-security-monitoring_reghtjy)

---

## Introducing Today's Project!

In this project, I will demonstrate how to set up a monitoring system in AWS using CloudTrail, CloudWatch, and SNS. I am doing this project to demonstrate how security and monitoring services in AWS work, plus have a working system that actually sends us emails too.

### Tools and concepts

Services I used were CloudTrail, CloudWatch, SNS, IAM Roles, Secrects Manager, and S3 Buckets. Key concepts I learned include secret storing, when to use CloudWatch vs CloudTrail, what notifications are and their endpoints (SMS, Email, Lambda, etc.), and how to create CloudWatch filters and alarms (very tedious!)

### Project reflection

This project took me just under 3 hours including demo time. The most challenging part was trying to figure out why the email notification was not delivering in the first test. It was frustrating when the error was happening but there was no error messages. The issue was not that the notification system was irresponsive, the problem was with my configuration and set-up of the when how I was going to be notified. It was most rewarding to compare CloudTrail SNS notification, it really opened my mind as to why CloudWatch + Alarms was as better option for this security notification system.

---

## Create a Secret

Secrets Manager is AWS's security service for storing secrets (database credentials, account ID's, API Keys, and any other sensitive information that would cause damage if leaked. Using Secrets Manager is a great alternative to hard-coding sensitive information within your code/scripts.

To set up for my project, I created a secret called "TopSecretInfo" in Secrets Manager. This secret is a string that contains a special statement written by me. The secret writes "AWS is better than Azure!"

![Image](http://learn.nextwork.org/ecstatic_beige_calm_tarapirohe/uploads/aws-security-monitoring_o5p6q7r8)

---

## Set Up CloudTrail

CloudTrail is a monitoring service. It is used to track events and activities in my AWS account. These logs are very helpful for security (i.e detecting suspicious activites), compliance (i.e. proving that I am following company rules and protocols), and troubleshooting (i.e. investigating why something went wrong in our cloud environment.)

CloudTrail events include types like management, data, insights, and network activity events. In this project I set up the Trail to track managament events becasue accessing the secret falls into that catagory. It is not a data event (which captures high volume actions performed on resources) because all management events are free to track. AWS allows us to track security events like this for free.

### Read vs Write Activity

"Read" API activity involves accessing, reading, and opening a resource. "Write" API activity involves creating, updating, and deleting a resource. For this project, I ticked both options to learn about both types of activities, but I really only need the "write" activity. Accessing the secret is considered a "write" activity because of its importance. By treating secret retrieval as a "Write" operation, it ensures these critical security events are captured in the CloudTrail logs even if someone configures CloudTrail to only log "write" events (which is a common cost-saving measure).

---

## Verifying CloudTrail

I retrieved the secret in two ways: First through the Secrets Manager console where I can easily select the "retreive secrets" button. Second, I used the AWS CLI by running a "get-secret-value" in CloudShell.

To analyze my CloudTrail events, I visited the "Event History" page in CloudTrail. I found that there was a "GetSecretValue" event tracked regardless of whether I performed the action through the console or with the AWS CLI/Cloudshell. This tells me that CloudTrail can definetly see and track whenever anyone opens the Secrets Manager key.

![Image](http://learn.nextwork.org/ecstatic_beige_calm_tarapirohe/uploads/aws-security-monitoring_s8t9u0v1)

---

## CloudWatch Metrics

CloudWatch Logs is a monitoring service that brings together logs from other AWS serviecs including CloudTrail to help me analysze and create alarms. It is important for monitoring because I get to create insights and get alerted based on events that happen in my account.

CloudTrail's Event History is useful for quickly reading management events that happened in the last 90 days, while CloudWatch Logs are better for analyzing logs from different sources, accessing logs for longer than 90 days and advanced filtering.

A CloudWatch metric is a specific way that I count & track events that are in a log group. When setting up a metric, the metric value represents how I increment or "count" an event when it passes my filters (in my case I want to increment the metric value by 1 whenever the Secret is accessed). The Default value is used when the event that we are tracking does not occur.

![Image](http://learn.nextwork.org/ecstatic_beige_calm_tarapirohe/uploads/aws-security-monitoring_a9b0c1d2)

---

## CloudWatch Alarm

A CloudWatch alarm is a feature and alert system in CloudWatch that's designed to "go off"/indicate when certain conditions have been met in the log group. I set my CloudWatch alarm threshold to be about how many times the "GetSecretValue" event happens in a 5 minute period, so the alarm will trigger when the average number of times is above "1".

I created an SNS topic along the way. An SNS topic is like the newsletter or broadcast channel that emails, phone numbers, apps, and Lambda functions can subscribe too, so they get notified when SNS has a new update to share. My SNS topic is set up to send me an email when the Secret gets accessed.

AWS requires email confirmation so it doesn't automatically start emailing addresses that I subscribed to an SNS topic. This helps prevent any unwanted subscriptions for recipients i.e. people who are recieving those emails.

![Image](http://learn.nextwork.org/ecstatic_beige_calm_tarapirohe/uploads/aws-security-monitoring_fsdghstt)

---

## Troubleshooting Notification Errors

To test my monitoring system, I opened and access the Secret once again. The results were not successful, I did not recieve an email or notification about my Secret being accessed.

When troubleshooting the notification issues, I investigated every single part of my monitoring system. I investigated: 1. Whether CloudTrail was picking up on the events that were happening when I access the Secret, 2. Whether CloudTrail was sending the logs to CloudWatch, 3. If the log filter was accidentally rejecting the correct events (False Negative), 4. Whether the triggering of the alarm actually sent an email like it was set up to do.

I initially didn't receive an email before because CloudWatch was configured to use the wrong threshold. Instead of calculating the "AVERAGE"number of times the secret was accessed in a tim period, It should have been calculating the "SUM".

---

## Success!

To validate that my monitoring system can successfully detect and alert when my secret is accessed, I checked my secret's value one more time. I recieved an email within 1 to 2 minutes of the event. My alarm in CloudWatch is also in "Alarm State" as it should be, because the secret has been accessed.

![Image](http://learn.nextwork.org/ecstatic_beige_calm_tarapirohe/uploads/aws-security-monitoring_ageraergearge)

---

## Comparing CloudWatch with CloudTrail Notifications

In a project extension, I enabled SNS notification delivery in CloudTrail because this lets me evaluate CloudTrail vs CloudWatch for notifying about events like my Secret being accessed.

After enabling CloudTrail SNS notifications, my inbox was very quickly filled with new emails from SNS as it was notified by 
CloudTrail. In terms of the usefulness of these emails, I realized that I recieved lots of abitrary logs that were not representative of management events that occured. I only see that new logs have been stored in my bucket.

![Image](http://learn.nextwork.org/ecstatic_beige_calm_tarapirohe/uploads/aws-security-monitoring_d7e8f9g0)

---

---
