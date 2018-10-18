# lambda-webhook-cloudfront
Solution for listening for Snippet updates via a Lambda webhook and pushing new snippet JS to S3 origin

---

### Create new Lambda function

* Create from Blueprint
* From list of Blueprints, choose *`microservice-http-endpoint`* (nodejs or python 3.6)
  * This will automatically set up an API gateway & set it as a trigger to your function
* Name your function, e.g. "optimizelywebhook"
* Assign to a (new) IAM [role](https://console.aws.amazon.com/iam/home?#/roles) that has access to:
  * Amazon S3 (policy: *AmazonS3FullAccess*)
  * Amazon CloudWatch Logs (policy: *AWSLambdaBasicExecutionRole* or *CloudWatchLogsFullAccess*)
* In the Lambda *Inline Code Editor*, replace with [this code](https://gist.github.com/cooperreid-optimizely/bad41f7a28e6c5a45c914404a5836a2c). Notes on code:
  * You must replace "YOUR-AWS-S3-BUCKET-NAME" with your S3 bucket name
  * The second argument in s3.Object is the S3 Key. This is the path that the file will be created in your bucket.
  * There is no security behind this Webhook

### Create Optimizely Webhook

* Visit the Settings // Webhooks page in Optimizely
* Create a new Webhook
* Supply the URL that was generated automatically when the API Gateway trigger was created
* **The Lambda API endpoint URL** can be found by clicking the button on the resource tree and scrolling towards the bottom
* You'll see a module that shows the API endpoint URL:

### Validating your setup

* In the Lambda console, you'll see the following associations if IAM role is properly configured
  * Triggers (left side): API Gateway
  * Resources (right side): Amazon CloudWatch Logs, Amazon S3
* Test function within Lambda console
  * Create a test function that will allow you to manually trigger the Lambda function from within the console. 
  * Build a request payload that looks identical to Optimizely's webhook requests.
  * Modify the following entities within your payload:
  * body, httpMethod
  * After running the Test trigger, you'll see a verbose output that shows both (a) http response & (b) output to CloudWatchLog

### Troubleshooting

* CloudWatch Logs - Monitor the CloudWatch Logs for Lambda function calls. The log group will automatically be generated for your function, as long as it has the proper IAM role. This is located in the [CloudWatch service](https://console.aws.amazon.com/cloudwatch/home)
* Manually dispatch Webhook using cURL on CLI
  * CURL command
  * The payload should be the same as your Test payload in the Lambda console
  * You should see both 
    1. A log entry for this function call
    2. The snippet showing up in your S3 destination