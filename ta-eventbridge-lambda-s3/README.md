# S3 Bucket Privatizer

This pattern utilizes AWS Trusted Advisor to execute an AWS Lambda Function via Amazon EventBridge, to remove public access to an S3 Bucket once it has been flagged as public. 
Incorrectly configured S3 buckets can be the source of a data leak in your organization. After deploying this serverless pattern, you will be automating the lock-down of your 
s3 buckets to prevent any accidental data leaks. If an S3 bucket meets the use-case for being open to the public, simply "Exclude & Refresh" the specific resource in 
the [AWS Trusted Advisor Console](https://console.aws.amazon.com/trustedadvisor/home) and they will be ignored by the 'S3 Bucket Privatizer'. 

The 'Amazon S3 Bucket Permissions' Trusted Advisor check is only available to AWS Accounts subscribed to [Business or Enterprise support plans](https://aws.amazon.com/premiumsupport/plans/)

Learn more about this pattern at Serverless Land Patterns: https://serverlessland.com/patterns/ta-eventbridge-lambda-s3

Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the [AWS Pricing page](https://aws.amazon.com/pricing/) for details. 
You are responsible for any AWS costs incurred. No warranty is implied in this example.

## Requirements

* [Create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) if you do not already have one and log in. The IAM user that you use must have sufficient permissions to make necessary AWS service calls and manage AWS resources.
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) installed and configured
* [Git Installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [Python 3.6 or later](https://www.python.org/downloads/)
* [pip and virtualenv](https://docs.aws.amazon.com/cdk/latest/guide/work-with-cdk-python.html#python-prerequisites)
* [AWS Cloud Development Kit](https://docs.aws.amazon.com/cdk/latest/guide/cli.html) (AWS CDK) installed

## Deployment Instructions

1. Create a new directory, navigate to that directory in a terminal and clone the GitHub repository:
    ```bash
    git clone https://github.com/aws-samples/serverless-patterns
    ```
2. Change directory to the pattern directory:
    ```bash
    cd ta-eventbridge-lambda-s3
    ```
3. From the command line, set Environment variables for your account and region:
    ```bash
    export CDK_DEFAULT_ACCOUNT=123456789012
    export CDK_DEFAULT_REGION=us-east-1
    ```
4. Activate your virtual environment by running the following command from the command line.
    ```bash
    source .venv/bin/activate
    ```
5. Bootstrap your CDK environment in the destination account [Learn More about bootstrapping](https://docs.aws.amazon.com/cdk/latest/guide/bootstrapping.html#bootstrapping-howto-cli)
    ```bash
    cdk bootstrap
    ```
6. Deploy the CDK App. 
    ```bash
    cdk deploy --parameters notificationemailaddress=validEmailAddress@hostingProvider.com --parameters profiling=TRUE --parameters tracing=TRUE --parameters trustedadvisorrefreshminutes=7
    ```
   **Parameter Definitions:**
   * **notificationemailaddress:** **[Required]** The E-mail address you wish to send notifications to when a WARN or ERROR flag is identified
   * **profiling:** TRUE or FALSE. If set to true, [CodeGuru Profiler](https://docs.aws.amazon.com/codeguru/latest/profiler-ug/what-is-codeguru-profiler.html) will be enabled. **Default:** FALSE
   * **tracing:** TRUE or FALSE. If set to true, [AWS X-Ray Tracing](https://aws.amazon.com/xray/) will be enabled. **Default:** FALSE
   * **trustedadvisorrefreshminutes:** Integer value between 5 and 1440. The number of minutes you wish to schedule a Trusted Advisor Check Refresh for S3 Bucket Permissions. **Default:** 7
   

7. When Asked: ```Do you wish to deploy these changes (y/n)?``` , Type ```y```, then press "Return"
## How it works

[AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/) periodically evaluates resources in your AWS accounts. 
Once a Trusted Advisor check is flagged as 'WARN' or 'ERROR' for the Trusted Advisor check 'Amazon S3 Bucket Permissions', 
an event is sent to [Amazon EventBridge](https://aws.amazon.com/eventbridge/), which triggers a serverless 
[AWS Lambda](https://aws.amazon.com/lambda/) function to remove the 
[bucket policy](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html) 
on the affected [Amazon S3](https://aws.amazon.com/s3/) Bucket, and send an e-mail via 
[Amazon Simple Notification Service](https://aws.amazon.com/sns/) (Amazon SNS)
to subscribers, notifying them of the S3 bucket policy removal.


## Testing

1. Login to your AWS account
2. [Create a new Amazon S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)
3. [Unblock public access to this Amazon S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/configuring-block-public-access-bucket.html)
4. [Add a bucket policy to the new S3 Bucket to allow Public access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/add-bucket-policy.html)
   
   Use the following Amazon S3 Bucket Policy, replacing 'bucket-name' with the name of the test bucket
   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicRead",
               "Effect": "Allow",
               "Principal": "*",
               "Action": [
                   "s3:GetObject",
                   "s3:GetObjectVersion"
               ],
               "Resource": "arn:aws:s3:::<bucket-name>/*"
           }
       ]
   }
   ```
5. [Navigate to the Trusted Advisor Security Category](https://console.aws.amazon.com/trustedadvisor/home#/category/security) or 
wait for the trusted advisor check to refresh based on the cdk parameter of trustedadvisorrefreshminutes=[number of minutes] 
(The number of minutes to wait until the trusted advisor check is refreshed)
   1. If you wish to manually trigger the trusted Advisor Check Refresh, click the circular refresh icon to the right of the
   Trusted Advisor check labeled "Amazon S3 Bucket Permissions". If the Last refreshed time is less than 5 minutes, you 
   will need to wait until 5 minutes has passed since the last refresh.
6. After the Trusted Advisor check refreshes, you will see the open bucket policy removed, and recieve an e-amil to the notification e-mail address


## Cleanup
 
1. Delete the CDK stack
    ```bash
    cdk destroy
    ```

----
Copyright 2021 Amazon.com, Inc. or its affiliates. All Rights Reserved.

SPDX-License-Identifier: MIT-0