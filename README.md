# How I use testing to build better apps with AWS CDK

Published: 2024-10-18
Medium: [https://medium.com/@kyle-t-jones/how-i-use-testing-to-build-better-apps-with-aws-cdk-eccf0cf13476](https://medium.com/@kyle-t-jones/how-i-use-testing-to-build-better-apps-with-aws-cdk-eccf0cf13476)

## Business context

Testing is essential to any software development process, even with infrastructure code. The need for testing has grown significantly as infrastructure becomes more complex and dynamic with the use of Infrastructure as Code (IaC). In AWS CDK (Cloud Development Kit), writing tests for your infrastructure is not just about making sure it works today, it's about ensuring it continues to work as expected in the future, even as things change.

Like with application code, you can write two main types of tests for your infrastructure code: unit tests and integration tests.

Unit tests focus on testing individual components or pieces of your CDK code. In this case, you're not deploying any cloud resources; instead, you're testing the logical structure of your CDK constructs. This could involve verifying that a particular resource, such as an S3 bucket or a Lambda function, is defined correctly.



## Disclaimer

Educational/demo code only. Not financial, safety, or engineering advice. Use at your own risk. Verify results independently before any production or operational use.

## License

MIT — see [LICENSE](LICENSE).