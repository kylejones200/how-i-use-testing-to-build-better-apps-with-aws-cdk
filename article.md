---
author: "Kyle Jones"
date_published: "October 18, 2024"
date_exported_from_medium: "November 10, 2025"
canonical_link: "https://medium.com/@kyle-t-jones/how-i-use-testing-to-build-better-apps-with-aws-cdk-eccf0cf13476"
---

# How I use testing to build better apps with AWS CDK Testing is essential to any software development process, even with
infrastructure code. The need for testing has grown significantly as...

### How I use testing to build better apps with AWS CDK
Testing is essential to any software development process, even with infrastructure code. The need for testing has grown significantly as infrastructure becomes more complex and dynamic with the use of Infrastructure as Code (IaC). In AWS CDK (Cloud Development Kit), writing tests for your infrastructure is not just about making sure it works today, it's about ensuring it continues to work as expected in the future, even as things change.

### Writing tests for IAC
Like with application code, you can write two main types of tests for your infrastructure code: unit tests and integration tests.

#### **Unit Tests**
Unit tests focus on testing individual components or pieces of your CDK code. In this case, you're not deploying any cloud resources; instead, you're testing the logical structure of your CDK constructs. This could involve verifying that a particular resource, such as an S3 bucket or a Lambda function, is defined correctly.

**Why Unit Tests?**

They are fast to execute because they don't interact with AWS services.

They help ensure that your code is logically correct before actual deployment.

Example of a Unit Test in CDK: A simple example of how you might write a unit test for a CDK project using the Jest framework (commonly used with CDK projects in TypeScript or JavaScript).

Let's say you have a CDK stack that defines an S3 bucket:

```python
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

export class MyS3Stack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyBucket', {
      versioned: true,
    });
  }
}
```

To write a unit test, you want to ensure that this bucket is created and that it has versioning enabled

```python
import * as cdk from 'aws-cdk-lib';
import { MyS3Stack } from './lib/my-s3-stack';
import { Template } from 'aws-cdk-lib/assertions';

test('S3 Bucket Created with Versioning', () => {
  const app = new cdk.App();
  const stack = new MyS3Stack(app, 'MyTestStack');
  const template = Template.fromStack(stack);

  template.hasResourceProperties('AWS::S3::Bucket', {
    VersioningConfiguration: {
      Status: 'Enabled',
    },
  });
});
```

In this test, we create an instance of the CDK stack. Then, we use CDK Assertions (more on this later) to check that the stack contains an S3 bucket with versioning enabled.

### Integration Tests
While unit tests check the logical structure of your CDK code, integration tests verify that the cloud resources you defined are being deployed and configured correctly in your AWS environment. This means that your code will interact with real AWS services, and you will need to handle actual resources such as EC2 instances, S3 buckets, or Lambda functions.

**Why Integration Tests?**

They ensure that your cloud infrastructure behaves as expected in a real environment.

Integration tests catch issues that unit tests can't, such as permission problems or deployment failures.

**Example of an Integration Test:** Let's say you want to test that a Lambda function in your CDK project can read from an S3 bucket. Your test would deploy the resources to AWS and confirm that the Lambda function behaves as expected.

Example of how you might write such a test using AWS SDK to interact with real cloud resources:

```python
import * as AWS from 'aws-sdk';

test('Lambda function can read from S3 bucket', async () => {
  const s3 = new AWS.S3();
  const lambda = new AWS.Lambda();

  // Upload a test file to the S3 bucket
  await s3.putObject({
    Bucket: 'my-test-bucket',
    Key: 'test-file.txt',
    Body: 'Hello, world!',
  }).promise();

  // Invoke the Lambda function
  const result = await lambda.invoke({
    FunctionName: 'my-lambda-function',
    Payload: JSON.stringify({ bucket: 'my-test-bucket', key: 'test-file.txt' }),
  }).promise();

  // Check the Lambda function's output
  const payload = JSON.parse(result.Payload as string);
  expect(payload).toEqual('Hello, world!');
});
```

**Note:** Integration tests are slower and more resource-intensive than unit tests because they interact with real AWS services. It's also essential to clean up any resources (such as S3 buckets or EC2 instances) after the tests are complete.

### How to use CDK Assertions to verify your setups
AWS CDK comes with a testing utility called CDK Assertions, which helps you verify that your CDK code produces the correct CloudFormation template. CDK Assertions allow you to inspect the logical structure of the stack by comparing it against expected values, such as checking that certain resources are present or have specific properties.

Key Functions of CDK Assertions

**Checking for Resource Presence:** You can check whether a particular resource, like an S3 bucket, exists in the generated CloudFormation template.

**Verifying Resource Properties:** You can ensure that resources have the correct configurations, such as an EC2 instance with a specific instance type or a Lambda function with a particular runtime.

**Counting Resources:** You can verify that the correct number of resources is defined, like ensuring your stack has exactly two IAM roles.

Example of Using CDK Assertions: An example that uses CDK Assertions to verify that a CDK stack contains an S3 bucket with versioning enabled:

```python
import * as cdk from 'aws-cdk-lib';
import { Template } from 'aws-cdk-lib/assertions';
import { MyS3Stack } from './lib/my-s3-stack';

test('S3 Bucket Created', () => {
  const app = new cdk.App();
  const stack = new MyS3Stack(app, 'MyTestStack');
  const template = Template.fromStack(stack);

  template.resourceCountIs('AWS::S3::Bucket', 1);
  // Check if exactly one S3 bucket is created
});
```

We use resourceCountIs to assert that exactly one S3 bucket exists in the stack. You can write additional assertions to validate other resources or configurations.

#### Snapshop testing
**Ensuring Your CDK Code Generates the Expected Templates**

Snapshot testing is a method used to capture a snapshot of your CDK-generated CloudFormation template and compare it against future versions. This ensures that any changes you make to your CDK code are intentional, and you can easily spot if something in your infrastructure has been modified accidentally.

When you run a snapshot test, the test framework (like Jest) saves the generated CloudFormation template. On subsequent test runs, it compares the new template against the saved snapshot. If they differ, the test fails, and you can review the changes to decide if they were expected or need investigation.

**Why Snapshot Testing?**

It provides a historical record of changes to your infrastructure definitions.

It ensures that no unexpected or unintended changes are introduced into your infrastructure.

Example of Snapshot Testing with CDK: Let's use Jest to write a snapshot test for a CDK stack:

```python
import * as cdk from 'aws-cdk-lib';
import { MyS3Stack } from './lib/my-s3-stack';

test('Snapshot of S3 Stack', () => {
  const app = new cdk.App();
  const stack = new MyS3Stack(app, 'MyTestStack');
  const template = JSON.stringify(stack.template);
  expect(template).toMatchSnapshot();
});
```

In this test, we generate the CloudFormation template from the CDK stack. Then, we compare the generated template to the existing snapshot using toMatchSnapshot(). If the stack's template changes in the future, Jest will flag this as a failure, allowing you to review and update the snapshot if necessary.

Snapshot tests are beneficial for infrastructure projects because they help catch subtle changes that might go unnoticed during code reviews or deployments.

### Continuous testing with CI/CD
Testing doesn't stop at your local development environment. In production-grade projects, you'll want to integrate testing into your CI/CD pipeline to automate the entire process. This ensures that every code change triggers your unit, integration, and snapshot tests, allowing you to catch issues early in the deployment cycle.

A simple outline of how to set up continuous testing for CDK projects:

1\. Set Up a CI/CD Pipeline

Tools like AWS CodePipeline, GitHub Actions, Jenkins, or CircleCI can help automate your testing process.

The pipeline automatically runs the tests whenever a developer pushes new code or submits a pull request.

If any tests fail, the pipeline halts, and the developer is notified to fix the issue before deploying.

2\. Include Testing Stages

In your pipeline configuration, include steps to run your unit, integration, and snapshot tests:

``` 
# Example for GitHub Actions
name: CDK Project CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Install Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'
      
      - name: Install dependencies
        run: npm install
      
      - name: Run tests
        run: npm test
```

3\. Test Environments

You don't need to deploy actual infrastructure for unit tests so that they can run in any standard environment (such as a Docker container).

For integration tests, it's recommended to have a separate test AWS account or sandbox environment where your natural resources can be safely deployed and tested without affecting production.

### Best practices for testing cloud resources
Start with Unit Tests: Unit tests are fast and inexpensive, so always start by ensuring that your CDK code logically defines the right resources. Catching errors here will save you from deploying broken infrastructure to the cloud.

**Use Snapshot Testing Wisely:** Snapshot testing is robust but can produce noisy failures if misused. Ensure you update snapshots only when the changes are intentional, and regularly review the snapshot files.

**Set Up Automatic Cleanup for Integration Tests:** After running integration tests, clean up the created resources (such as S3 buckets or EC2 instances). You can automate this cleanup in your test scripts or use tools like AWS CloudFormation's automatic stack deletion feature.

**Test Permissions and Policies:** Cloud security is a significant concern. Write tests to validate that your IAM roles, policies, and permissions are configured correctly. Ensure that your resources have the least privileged access necessary to function properly.

**Use Feature Flags for Gradual Rollouts:** When deploying new features, use feature flags or separate environments (like staging and production) to control which parts of the infrastructure get updated. This minimizes risk and gives you time to test in production without affecting all users.

**Integrate with Monitoring and Alerts:** Testing shouldn't end once your infrastructure is deployed. Use monitoring services like Amazon CloudWatch and AWS Config to verify your cloud resources' health and configuration continuously. Set up alerts for any unexpected changes or failures in your infrastructure.
