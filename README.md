# lambda-webhook-cloudfront

Solution for listening for Snippet updates via a Lambda webhook and pushing the updated Optimizely snippet to an S3 bucket. This facilitates the ability to self-host the Optimizely snippet in your CloudFront instance. The workflow is as follows:

* An Optimizely user makes an update which triggers a Snippet update
* Optimizely webhook hits the Lambda function's API endpoint URL
* Lambda function is triggered, which:
  * Downloads JS snippet contents
  * Add/replaces the JS snippet in the specified S3 Bucket + path (`Key`)
* The S3 bucket is fronted by CloudFront

---

### Create new Lambda function

* Visit [Lambda console](https://console.aws.amazon.com/lambda/home)
* Create from Blueprint
* From list of Blueprints, choose *`microservice-http-endpoint`* (nodejs or python 3.6)
  * This will automatically set up an API gateway & set it as a trigger to your function
* Name your function, e.g. "optimizelywebhook"
* Assign to a (new) IAM [role](https://console.aws.amazon.com/iam/home?#/roles) that has access to:
  * Amazon S3 (policy: *AmazonS3FullAccess*)
  * Amazon CloudWatch Logs (policy: *AWSLambdaBasicExecutionRole* or *CloudWatchLogsFullAccess*)

![policies](https://github.com/optimizely-solutions/lambda-webhook-cloudfront/blob/master/images/policies.png?raw=true)

* In the Lambda *Inline Code Editor*, replace with [this code](https://gist.github.com/cooperreid-optimizely/bad41f7a28e6c5a45c914404a5836a2c). Notes on code:
  * You must replace "YOUR-AWS-S3-BUCKET-NAME" with your S3 bucket name
  * The second argument in s3.Object is the S3 Key. This is the path that the file will be created in your bucket.
  * There is no security behind this Webhook

### Create Optimizely Webhook

* Visit the Settings // Webhooks page in Optimizely
* Create a new Webhook
* Supply the URL that was generated automatically when the API Gateway trigger was created
* **The Lambda API endpoint URL** can be found by clicking the button on the resource tree and scrolling towards the bottom

![gateway button](https://github.com/optimizely-solutions/lambda-webhook-cloudfront/blob/master/images/gateway.png?raw=true)

* You'll see a module that shows the API endpoint URL:

![tree](https://github.com/optimizely-solutions/lambda-webhook-cloudfront/blob/master/images/endpointurl.png?raw=true)

### Validating your setup

* In the Lambda console, you'll see the following associations if IAM role is properly configured
  * Triggers (left side): API Gateway
  * Resources (right side): Amazon CloudWatch Logs, Amazon S3

![tree](https://github.com/optimizely-solutions/lambda-webhook-cloudfront/blob/master/images/resourcetree.png?raw=true)

* Test function within Lambda console
  * Create a test function that will allow you to manually trigger the Lambda function from within the console. 
  * Build a request payload that looks identical to Optimizely's webhook requests.
  * Modify the following entities within your payload:
    * `"body": "{\"timestamp\":1000000000,\"project_id\":PROJECTID,\"data\":{\"cdn_url\":\"https://cdn.optimizely.com/js/PROJECTID.js\",\"origin_url\":\"https://optimizely.s3.amazonaws.com/js/0PROJECTID.js\",\"revision\":1},\"event\":\"project.snippet_updated\"}"`
    * `"httpMethod": "POST"`
  * After running the Test trigger, you'll see a verbose output that shows both (a) http response & (b) output to CloudWatchLog
  
![success](https://github.com/optimizely-solutions/lambda-webhook-cloudfront/blob/master/images/success.png?raw=true)

### Troubleshooting

* CloudWatch Logs - Monitor the CloudWatch Logs for Lambda function calls. The log group will automatically be generated for your function, as long as it has the proper IAM role. This is located in the [CloudWatch service](https://console.aws.amazon.com/cloudwatch/home)
* Manually dispatch Webhook using cURL on CLI
  * `curl --data "{\"timestamp\":1000000000,\"project_id\":PROJECTID,\"data\":{\"cdn_url\":\"https://cdn.optimizely.com/js/PROJECTID.js\",\"origin_url\":\"https://optimizely.s3.amazonaws.com/js/0PROJECTID.js\",\"revision\":1},\"event\":\"project.snippet_updated\"}" https://UNIQUE.execute-api.us-east-1.amazonaws.com/default/FUNCTIONNAME`
  * The payload should be the same as your Test payload in the Lambda console
  * You should see both 
    1. A log entry for this function call
    2. The snippet showing up in your S3 destination
