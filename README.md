# Amazon SNS SMS Detailed Monitoring Dashboard
## Overview

[Amazon Simple Notification Service \(Amazon SNS\)](https://aws.amazon.com/sns/) sends notifications two ways, A2A and A2P\. A2A provides high\-throughput, push\-based, many\-to\-many messaging between distributed systems, microservices, and event\-driven serverless applications\. A2P functionality lets you send messages to your customers with SMS texts, push notifications, and email\. 

In this blog, we will focusing about the monitoring the SMS texts sent using Amazon SNS\. SNS Customers use SMS service for variety of usecases including multi\-factor authentication, OTP messages, transaction alerts, appointment reminders etc\. Different destination Countries keep implmenting different mechanisms to prevent the spread of spam messsages in the country and hence, keep coming up with the different requirements\. An Amazon SNS SMS flow looks like below:

![Amazon SNS SMS Flow](Assets/SNS-SMS-Flow.png?raw=true)


As can be seen above, the delivery of SMS texts depends upon lot of external factors like AWS downstream providers, the mobile carrier itself and finally the receiving handset\. Issues can arise at any of the above mentioned stages resulting in the SMS delivery failure\. Hence, in majority of cases, it becomes crucial to monitor the SMS traffic in real time\. In order to efficiently monitor the SMS deliveries, we need to process and analyze [SNS SMS Delivery Logs and SNS SMS related CloudWatch Metrics](https://docs.aws.amazon.com/sns/latest/dg/sms_stats_cloudwatch.html)\.

With the help of Delivery Logs and CloudWatch metrics, we can analyze delivery rate for the various countries and SNS SMS ProviderResponse patterns over a period of time\. This can help in understanding if there is an increase in any particular ProviderRespnse over the time or if there is a drop in delievry rate in any country\. Such type of real\-time analyse can help in idenitfying issues at an early stage which can be then escalated to AWS Support for further investigation and troubleshooting\.

In this blog, we are discussing about [a CloudWatch Custom Dashboard](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html) which can help in such monitoring by processing and gathering useful insights from the SNS SMS delivery logs and CloudWatch Metrics\. In case of any changes done by the AWS downstream providers to mitigate the issue in the SMS deliveries, the same dashboard can be used to analyze and monitor its impact on the upcoming SMS traffic\. This dashboard can act as a consolidated place to monitor all your SNS SMS related activities\. In the following sections, we will be discussing about the various widgets of this CW Dashboard and how they can be used to analyze the SNS SMS traffic\. 

## Dashboard Features and Widgets:

### CloudWatch Metric based widgets

1. __SNS SMS Monthly Spend \(USD\)__ – This widget uses SNS ‘[SMSMonthToDateSpentUSD](https://docs.aws.amazon.com/sns/latest/dg/sns-monitoring-using-cloudwatch.html#sns-metrics)’ CloudWatch Metric and indicates the charges you have accrued since the start of the current calendar month for sending SMS messages\. If this metric hits the upper limit configured in your [SNS Text Preferences](https://docs.aws.amazon.com/sns/latest/dg/sms_preferences.html#sms_preferences_console) or the upper limit allowed by AWS in any month, SNS will stop sending any new text messages\. By default this upper limit is 1 USD\.
2. __SNS SMS Success Rate Metric for different Countries and Message Type__ – This widget will display the Success Rate for different countries where you’re sending text messages for each message type \(Transactional or Promotional\)\. The metric indicates the rate of successful SMS message deliveries over the selected time period\.
3. __Number Of SMS sent vs delivered \- by Country and Message Type__ – This is a [custom CloudWatch widget](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/add_custom_widget_dashboard.html) backed by [AWS Lambda function](https://aws.amazon.com/lambda/) named *‘CustomWidget\-GetMetricData\-SNS\-SuccessRate’*\. This widget displays on daily basis \(for the selected duration\) the total number of SMS __sent__ vs __succesfully delivered__ in different countries and of different type \(transaction/promotional\)\. This widget gets data from the ‘SMSSuccessRate’ CloudWatch Metric\. SMSSuccessRate metric with Statistic Sample Count indicates the total number of messages sent to a partuclar country of particular message type, while the Statistic Sum indicates the SMS successfully delivered\. The difference between the two can be used to get the number of text messages which were failed to get delivered due to various reasons listed [here](https://docs.aws.amazon.com/sns/latest/dg/sms_stats_cloudwatch.html#sms_stats_delivery_fail_reasons)\. The Statistic Average will give us the Success rate seen in widget \#2\.

![Metric based widget screenshot](Assets/Metric-based-Insights.png?raw=true)

4. __NumberOfMessagesPublished to Direct Phone Numbers__ – This widget indicates the total number of SNS Text messages sent to direct phone numbers from your AWS account using the given AWS Region\. This widget uses data from SNS __NumberOfMessagesPublished__ CloudWatch Metric\. This metric does not include the number of text messages sent via SNS Topics\.
5. __SNS SMS Monthly Spend Limit Alarm State__ – If you’re deploying this CW Dashboard via this blog, it also creates a [CloudWatch Alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html) on SNS ‘SMSMonthToDateSpentUSD’ metric named __‘SNS SMS Monthly Spend Limit Alarm\-\[AWSRegion\]’__\. This widget tells you about the status of this particular CW alarm\. The CloudWatch Alarm on ‘SMSMonthToDateSpentUSD’ metric can help you in notifying well in advance in case your SNS SMS spend crosses a configured threshold\. Then you can timely take required actions and [request higher spend limit from AWS Support](https://docs.aws.amazon.com/sns/latest/dg/channels-sms-awssupport-spend-threshold.html)\. By default, the threshold is configured at 80% of the default 1 USD limit\.

__\[Todo\]:__ [Modify](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Edit-CloudWatch-Alarm.html) the threshold of the above CW Alarm as per your current spending limits\. Also, [configure the required email address](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/US_SetupSNS.html) in the CW alarm which can be used to notify you if the alarm goes into alarming state\.

### CloudWatch Logs based widgets

1. __Fetch SNS SMS Delivery Logs__ – This Custom CW widget can be used to fetch the required SNS SMS delivery logs\. Since, by default the SNS SMS delivery logs for all the countries and message type comes up in the same log group, it becomes difficult to find the required delivery logs to escalate any issue to the AWS Support\. With the help of this widget, you can filter the SNS SMS delivery logs with Destination Country Code, Message Type, Delivery Status, destiantion phone number and SNS Provider Response\. This will help in quickly fetching the required SNS SMS delivery logs\.  For example:
	1. You received OTP delivery failure complaints from your clients in a particular country, you can quickly add filter for the destiantion country, message type \(transactional\), and delivery status to fetch, analyze and escalate the issue to AWS Support \(if required\)\.
	2. You receive a complaint from particular client about the missing OTP, and you would like to find delivery logs for his/her number for further analysis\.
	3. You observe “CarrierBlocked” provider response in the delivery logs and hence, want to fetch more such logs for a particular country to escalate\.

    ![Log based widget 1 screenshot](Assets/Log-Based-Insights-1.png?raw=true)


    This Custom widget is backed by AWS Lambda function named *‘CustomWidget\-Fetch\-SMS\-DeliveryLogs’*\. The Lambda function fetches the required logs by running [CloudWatch Log Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html) query on the SNS SMS delivery log groups \(for successful deliveries \- SNS/\<AWS Region>/\<Account Number>/DirectPublishtoPhoneNumber, Failure log group \- SNS/\<AWS Region>/\<Account Number>/DirectPublishtoPhoneNumber/Failure\)\.

2. __ProviderResponse Insights by Country and Message type__ – This widget will display the number of occurances of various provider responses in the required countries for various message type\. This widget will be helpful in analyzing the pattern of various provdier responses over a period of time\. For example, how was the trend of provider response “Unknown error attempting to reach phone” over last few days and if there is an increase in the same\. Similarly, after any changes performed by AWS downstream providers, how is the new trend going on\.

	![Log based widget 2 screenshot](Assets/Log-Based-Insights-2.png?raw=true)


	This custom CW widget is also backed by AWS Lambda function named *‘CustomWidget\-Fetch\-SMS\-DeliveryLog\-Insights’* and fetch insights by running CloudWatch Insights query on the SNS SMS delivery log groups\.


## Deployment:

You can deploy the above discussed CloudWatch Dashboard for SNS SMS monitoring by using the CloudForamtion template available [here](https://gitlab.aws.dev/swwadhwa/sns-sms-detailed-monitoring-dashboard/-/blob/main/CloudFormationTemplate-SNS-SMS-Monitoring-Dashboard.json)\.

1. Download the [AWS CloudFormation template](https://gitlab.aws.dev/swwadhwa/sns-sms-detailed-monitoring-dashboard/-/blob/main/CloudFormationTemplate-SNS-SMS-Monitoring-Dashboard.json) and navigate to **AWS CloudFormation console** under the AWS region you want to deploy the solution\.
2. Select __Create stack and With new resources__\. Choose __Template is ready__ as __Prerequisite – Prepare template__ and __Upload a template file__ as __Specify template__\. Upload the template downloaded in __step 1__\.
3. Complete the rest of the [steps](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html)\. Once the CloudFormation stack is successfully deployed, you can navigate to the [CloudWatch Custom Dashboard console](https://us-east-1.console.aws.amazon.com/cloudwatch/home?#dashboards) in the concerned region and search for the dashboard __*‘SNS\-SMS\-Detailed\-Monitoring\-Dashboard\-\{AWSRegion\}’\.*__

Note: To access the CW Dashboard, you might need additional permissions listed in this [documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html)\.

### Clean\-up

In case you no longer need the dashboard, you can go ahead and [delete the CloudFormation Stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html) created above\.

## Associated additional charges:

### Amazon CloudWatch – 

Amazon CloudWatch pricing can be found [here](https://aws.amazon.com/cloudwatch/pricing/)\. You may incur additional charges for CloudWatch Dashboard, Logs \(for analyzing logs using Log Insights\), and Metrics \(GetMetricData API used by *‘CustomWidget\-GetMetricData\-SNS\-SuccessRate’ *Lambda function\)\. 

### AWS Lambda – 

Custom widgets “Number Of SMS sent vs delivered \- by Country and Message Type”, “Fetch SNS SMS Delivery Logs”, and “ProviderResponse Insights by Country and Message type” are backed by AWS Lambda function\. A Lambda function will be triggered either when you refresh these widget’s data or they auto\-refresh\. Lambda Pricing can be found [here](https://aws.amazon.com/lambda/pricing/)\.

__Note__: Both the services also offer Free Tier which you can refer in their respective pricing documentations\.

## Customization:

This custom CloudWatch Dashboard can be further customized to gather Cross\-Account and Cross\-Region data for analyzing data from multiple region\(s\) and AWS account\(s\) at a single place\. For more details you may refer to this [documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_xaxr_dashboard.html)\. Custom Widgets backed by Lambda functions are also using a ‘region’ variable which can be modified to support cross region data\.

