---
tags: ['post']
post_og_image: 'site'
date: '2021-05-27'  
post_title: Scalable CRON job executor for AWS Lambda
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: scalable-cron-job-executor-for-aws-lambda
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

Running scheduled jobs at scale in the cloud is an interesting problem to solve. In this post I will describe an architecture we came up with at [Fusebit](https://fusebit.io) that allows us to execute a large number of arbitrarily scheduled CRON jobs in AWS using Lambda, SQS, and CloudWatch.

### The problem

Say you are building a large-scale, multi-tenant system where tenants define programmatic jobs to be executed in the cloud on a given schedule. You have a large number of such jobs, each with a potentially different execution schedule.

In the context of [Fusebit](https://fusebot.io), we needed a solution to this problem to enable customers of our integration platform to implement and run integration logic on a given schedule. For example, to export new leads from HubSpot to MailChimp every night, or to send a status report based on Jira tickets to Slack every hour.

### The non-scalable CRON solution

We have decided to use AWS Lambda to run the customer-provided logic. While Lambda is not an ideal compute layer for all types of integration workloads, in our case it satisfied the functional requirements while providing a convenient way of scaling and isolating workloads of multiple tenants.

Once the infrastructure was in place to run the customer code, the next step was to trigger it on a customer-defined schedule. There are many ways to trigger execution of a Lambda function in AWS. One of them uses [scheduled CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html). On the surface, CloudWatch Events meet all the requirements. You can define an arbitrary CRON schedule following which a CloudWatch Event will be triggered, and attach a specific Lambda function to execute in response.

<a href="/assets/images/blog/tomek_blog/2021-05-27/0.svg" style="border-bottom:none;"><img src="/assets/images/blog/tomek_blog/2021-05-27/0.svg" class="tj-img-diagram-100" alt="Non-scalable CRON executor for AWS Lambda"></a>

The problem with using CloudWatch Events in a highly scalable system is the limits. Per AWS region and account, one can only support up to 100 scheduled events (as of this writing), which was not sufficient to support the needs of a highly scalable multi-tenant system. We clearly needed a different approach.

### The scalable CRON solution

After looking around for the right building blocks, a lesser known feature of Amazon SQS drew our attention: [SQS delay queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-delay-queues.html). The feature enables a message published to an SQS queue to remain invisible to consumers for a configurable time, up to 15 minutes.

Equipped with this new tool, we have split our CRON processing pipeline by adding SQS delay queues in the middle, and using two additional Lambda functions: a _scheduler_ and an _executor_.

First, we defined a _single_ scheduled CloudWatch Event that triggered the scheduler Lambda function every 10 minutes, starting at minute 8 of an hour. So the scheduler Lambda was running at minute 8, 18, 28, 38 etc. of every hour.

<a href="/assets/images/blog/tomek_blog/2021-05-27/1.svg" style="border-bottom:none;"><img src="/assets/images/blog/tomek_blog/2021-05-27/1.svg" class="tj-img-diagram-100" alt="Scalable CRON executor for AWS Lambda"></a>

The purpose of the scheduler Lambda was to consider all scheduled jobs in the system (possibly millions), and select those that were due for execution in the subsequent whole-10-minute interval. For example, if the scheduler Lambda was running at 3:18, it would determine all scheduled job executions that need to occur in the 3:20-3:30 time span. The scheduler Lambda would then enqueue those job definitions to the SQS queue, setting the delayed delivery for each to correspond to the exact intended moment of execution of that job. For example, if a job was to run at 3:24, the scheduler Lambda running at 3:18 would enqueue that job to SQS setting the delayed execution to 6 minutes. The granularity of delayed execution allows the execution time to be set with 1 second precision.

Lastly, the executor Lambda would consume messages from SQS as they are released following their delayed delivery settings. The executor Lambda would then invoke the appropriate customer-defined Lambda function to execute the intended logic.

With this architecture, we were able to use a single CloudWatch Event to schedule a very large number of jobs and support our scheduled integration needs.

### Shameless plug

This is just one of many interesting technical problems we are continuously solving when building the integration platform at Fusebit. If you enjoy working on developer-centric products and cracking this type of technical problems on a daily basis and in a good company, [we are hiring](https://fusebit.io/careers)!
}