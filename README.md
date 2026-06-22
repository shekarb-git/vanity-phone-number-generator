# Amazon Connect Vanity Number Generator

This project deploys a serverless Amazon Connect solution that:

* Converts a give phone number into possible vanity numbers.
* Returns vanity number suggestions to the Amazon Connect contact flow.
* Stores the best vanity numbers in Amazon DynamoDB.
* Provides a simple web page to display the vanity numbers generated for the given phone numbers.

## Prerequisites

Before you begin, ensure you have:

* AWS CLI v2 installed and configured
* An AWS account
* An Amazon Connect instance
* An Amazon S3 bucket in the target AWS Region
* Permission to create IAM, Lambda, DynamoDB, and CloudFormation resources

---

## Step 1 – Download Project Files

Download the following files into a local working directory:

* `generateVanityNumbers.zip`
* `displayVanityNumbers.zip`
* `template.yaml`

---

## Step 2 – Configure AWS CLI

Open a terminal (or Command Prompt) and change to the project directory.

Verify your AWS configuration:

```bash
aws login
aws configure list
```

If necessary, switch to the desired AWS Region:

```bash
set AWS_REGION=us-east-1
```

> **Note:** Replace `us-east-1` with the Region where your Amazon Connect instance resides.

---

## Step 3 – Upload Lambda Packages to Amazon S3

The Lambda deployment packages must be uploaded to Amazon S3 before deploying the CloudFormation stack.

```bash
aws s3 cp generateVanityNumbers.zip s3://<your-s3-bucket>

aws s3 cp displayVanityNumbers.zip s3://<your-s3-bucket>
```

Replace `<your-s3-bucket>` with the name of your S3 bucket. The `template.yaml` file assumes that above files are uploaded to Root Directory of s3 bucket.

---

## Step 4 – Deploy the CloudFormation Stack

The `template.yaml` file provisions all required AWS resources.

### Required Parameters

| Parameter            | Description                                         |
| -------------------- | --------------------------------------------------- |
| `ConnectInstanceARN` | ARN of your Amazon Connect instance                 |
| `S3BucketName`       | S3 bucket containing the Lambda deployment packages |

Example:

```text
ConnectInstanceARN
arn:aws:connect:us-east-1:123456789012:instance/a1b2c3d4-1234-1e34-4567-fg8hi9j0kl1m

S3BucketName
my-demo-bucket
```

Deploy the stack:

```bash
aws cloudformation create-stack \
--stack-name aws-connect-vanity-number-generator \
--template-body file://template.yaml \
--parameters \
ParameterKey=ConnectInstanceARN,ParameterValue=<your-connect-instance-arn> \
ParameterKey=S3BucketName,ParameterValue=<your-s3-bucket> \
--capabilities CAPABILITY_NAMED_IAM
```

---

## Step 5 – Verify Stack Creation

Open the AWS Management Console and navigate to **CloudFormation**.

Locate the stack:

```
aws-connect-vanity-number-generator
```

Wait until the stack status is:

```
CREATE_COMPLETE
```

Open the **Outputs** tab and note the generated resources, including:

* GenerateVanityNumbersFunctionFunctionArn
* DisplayVanityNumbersFunctionArn
* DisplayVanityNumbersFunctionURL
* DynamoDBTableArn
* VanityNumbersContactFlow

Save the **DisplayVanityNumbersFunctionURL** for later use.

---

## Step 6 – Associate the Contact Flow

Sign in to your Amazon Connect instance.

Navigate to:

```
Channels
    → Phone Numbers
```

Either:

* Associate an existing phone number with the deployed **Vanity Number Contact Flow**, or
* Claim a new phone number and associate it with the deployed **Vanity Number Contact Flow**.

---

## Step 7 – Test the Solution

Call the associated Amazon Connect phone number.


---

## Step 8 – View the Stored Vanity Numbers

Open the **DisplayVanityNumbersFunctionURL** obtained from the CloudFormation Outputs from Step 5 above.

The web page displays the vanity numbers generated for the given phone numbers.

---

## Step 9 – Clean Up Resources

Delete all resources created by this project:

```bash
aws cloudformation delete-stack --stack-name aws-connect-vanity-number-generator
aws s3 rm s3://<your-s3-bucket>/displayVanityNumbers.zip
aws s3 rm s3://<your-s3-bucket>/generateVanityNumbers.zip
```

---

## Step 10 – Run Unit Tests

Extract the contents of `generateVanityNumbers.zip`.

Run the unit tests for `vanity_converter.py` using below command from Command Prompt on the root directory of extracted archive.

```bash
python -m pytest tests/ -v 2>&1
```

---

## Resources Created

The CloudFormation template provisions the following AWS resources:

* Amazon DynamoDB table
* Amazon Connect Contact Flow
* AWS Lambda – Vanity Number Generator
* AWS Lambda – Display Vanity Numbers
* Lambda Function URL
* IAM Roles and Policies required by the Lambda functions
