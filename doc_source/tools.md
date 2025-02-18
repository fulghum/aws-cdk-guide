--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS CDK Tools<a name="tools"></a>

This topic describes the tools you can use to work with the AWS CDK\. These tools include the AWS CDK Command Line Interface \(cdk\) and AWS SAM CLI\.

## AWS CDK Command Line Interface \(cdk\)<a name="cli"></a>

The AWS CDK CLI, known as the cdk, is your main tool for interacting with your AWS CDK app\. The cdk executes the AWS CDK app you wrote and compiled, interrogates the application model you defined, and produces and deploys the AWS CloudFormation templates that the AWS CDK generates\.

There are two ways to tell the cdk what command to use to run your AWS CDK app\. 

The first way is to include an explicit \-\-app option whenever you use a cdk command\.

```
cdk --app "npx ts-node bin/hello-cdk.ts" ls
```

The second way is to add the following entry to the `cdk.json` file \(if you use the cdk init command, the command does this for you\)\.

```
{
  "app": "npx ts-node bin/hello-cdk.ts"
}
```

You can also use the npx cdk instead of just the cdk\.

Here are the actions you can take on your AWS CDK app \(this is the output of the cdk \-\-help command\)\.

```
Usage: cdk -a <cdk-app> COMMAND

Commands:
  cdk list                        List all stacks in the app      [aliases: ls]
  cdk synthesize [STACKS..]       Synthesize and print the AWS CloudFormation
                                  template for this stack       [aliases: synth]
  cdk bootstrap [ENVIRONMENTS..]  Deploy the CDK toolkit stack into an AWS
                                  environment
  cdk deploy [STACKS..]           Deploy the stack(s) named STACKS into your
                                  AWS account
  cdk destroy [STACKS..]          Destroy the stack(s) named STACKS
  cdk diff [STACKS..]             Compare the specified stack with the deployed
                                  stack or a local template file, and returns
                                  with status 1 if any difference is found
  cdk metadata [STACK]            Return all metadata associated with this
                                  stack
  cdk init [TEMPLATE]             Create a new, empty CDK project from a
                                  template. Invoked without TEMPLATE, the app
                                  template will be used.
  cdk context                     Manage cached context values
  cdk docs                        Open the reference documentation in a browser
                                                                  [aliases: doc]
  cdk doctor                      Check your setup for potential problems

Options:
  --app, -a             REQUIRED: command line for executing your app or a cloud
                        assembly directory (e.g., "node bin/my-app.js")  [string]
  --context, -c         Add contextual string parameter (KEY=VALUE)      [array]
  --plugin, -p          Name or path of a node package that extends the CDK
                        features. Can be specified multiple times        [array]
  --rename              Rename stack name if different from the one defined in
                        the cloud executable ([ORIGINAL:]RENAMED)       [string]
  --trace               Print trace for stack warnings                 [boolean]
  --strict              Do not construct stacks with warnings          [boolean]
  --ignore-errors       Ignore synthesis errors, which will likely produce an
                        invalid output                [boolean] [default: false]
  --json, -j            Use JSON output instead of YAML
                                                      [boolean] [default: false]
  --verbose, -v         Show debug logs               [boolean] [default: false]
  --profile             Use the indicated AWS profile as the default environment
                                                                        [string]
  --proxy               Use the indicated proxy. Will read from HTTPS_PROXY
                        environment variable if not specified.          [string]
  --ec2creds, -i        Force trying to fetch EC2 instance credentials. Default:
                        guess EC2 instance status.                     [boolean]
  --version-reporting   Include the "AWS::CDK::Metadata" resource in synthesized
                        templates (enabled by default)                 [boolean]
  --path-metadata       Include "aws:cdk:path" CloudFormation metadata for each
                        resource (enabled by default)  [boolean] [default: true]
  --asset-metadata      Include "aws:asset:*" CloudFormation metadata for
                        resources that user assets (enabled by default)
                                                       [boolean] [default: true]
  --role-arn, -r        ARN of role to use when invoking AWS CloudFormation [string]
  --toolkit-stack-name  The name of the CDK toolkit stack               [string]
  --staging             Copy assets to the output directory (use --no-staging to
                        disable, needed for local debugging of the source files
                        with SAM CLI)                  [boolean] [default: true]
  --output, -o          Emits the synthesized cloud assembly into a directory
                        (default: cdk.out)                              [string]
  --ci                  Force CI detection. Use --no-ci to disable CI
                        autodetection.                [boolean] [default: false]
  --tags, -t            Tags to add to the stack (KEY=VALUE)             [array]
  --version             Show version number                            [boolean]
  -h, --help            Show help                                      [boolean]

If your app has a single stack, there is no need to specify the stack name

If one of cdk.json or ~/.cdk.json exists, options specified there will be used
as defaults. Settings in cdk.json take precedence.
```

If your app has a single stack, you don't have to specify the stack name\.

### Bootstrapping Your AWS Environment<a name="tools_bootstrap"></a>

Before you can use the AWS CDK, you must create the infrastructure that the AWS CDK CLI \(cdk\) needs to deploy your AWS CDK app\. Currently the bootstrap command creates only an Amazon S3 bucket\.

You incur any charges for objects that the AWS CDK stores in the bucket\. Because the AWS CDK doesn't remove any objects from the bucket, the bucket accumulates them as you use the AWS CDK\. You can get rid of the bucket by deleting the **CDKToolkit** stack from your account\. 

### Security\-Related Changes<a name="tools_security"></a>

To protect you against unintended changes that affect your security posture, the AWS CDK command\-line tool prompts you to approve security\-related changes before deploying them\.

You modify the level of changes that require approval by specifying the following\.

```
cdk deploy --require-approval LEVEL
```

*LEVEL* can be one of the following values\.

never  
Approval is never required\.

any\-change  
Requires approval on any AWS Identity and Access Management \(IAM\) or security\-group related change\.

broadening  
\(default\) Requires approval when IAM statements or traffic rules are added\. Removals don't require approval\.

You can also configure the setting in the `cdk.json` file\.

```
{
  "app": "...",
  "requireApproval": "never"
}
```

### Version Reporting<a name="version_reporting"></a>

To gain insight into how the AWS CDK is used, the versions of libraries used by AWS CDK applications are collected and reported by using a resource identified as `AWS::CDK::Metadata`\. This resource is added to AWS CloudFormation templates, and can easily be reviewed\. You can also use this information to identify stacks using a package with known serious security or reliability issues, and contact the package users with that important information\.

The AWS CDK reports the name and version of `npm` modules that are loaded into the application at synthesis time, unless their `package.json` file contains the `"private": true` attribute\.

The `AWS::CDK::Metadata` resource looks like the following\.

```
CDKMetadata:
  Type: "AWS::CDK::Metadata"
  Properties:
    Modules: "@aws-cdk/core=0.7.2-beta,@aws-cdk/s3=0.7.2-beta,lodash=4.17.10"
```

### Opting Out from Version Reporting<a name="version_reporting_opt_out"></a>

To opt out of version reporting, use one of the following methods:
+ Use the cdk command with the \-\-no\-version\-reporting argument\.

  ```
  cdk --no-version-reporting synth
  ```
+ Set versionReporting to **false** in `./cdk.json` or `~/cdk.json`\.

  ```
  {
    "app": "...",
    "versionReporting": false
  }
  ```

## AWS SAM CLI<a name="sam"></a>

This topic describes how to use the AWS SAM CLI with the AWS CDK to test an AWS Lambda function locally\.For more information about testing Lambda functions locally, see [Invoking Functions Locally](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-invoke.html)\. 

To install the AWS SAM CLI, see [Installing the AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)\.

### AWS SAM Example<a name="sam_example"></a>

The following example walks you through creating an AWS CDK application with a Lambda function and testing that function locally using AWS SAM\.

1. Create an AWS CDK app and add the Lambda package\.

   ```
   mkdir cdk-sam-example
   cd cdk-sam-example
   cdk init app --language typescript
   npm install @aws-cdk/aws-lambda
   ```

1. Add a Lambda reference to `lib/cdk-sam-example-stack.ts`\.

   ```
   import lambda = require('@aws-cdk/aws-lambda');
   ```

1. Replace the comment in `lib/cdk-sam-example-stack.ts` with the following Lambda function\.

   ```
   new lambda.Function(this, 'MyFunction', {
     runtime: lambda.Runtime.Python37,
     handler: 'app.lambda_handler',
     code:    lambda.Code.asset('./my_function'),
   });
   ```

1. Create the directory `my_function`\.

   ```
   mkdir my_function
   ```

1. Create the file `app.py` in `my_function` with the following content\.

   ```
   def lambda_handler(event, context):
     return "This is a Lambda Function defined through CDK"
   ```

1. Compile your AWS CDK app and create an AWS CloudFormation template\.

   ```
   npm run build
   cdk synth > template.yaml
   ```

1. Find the logical ID for your Lambda function in `template.yaml`\. It will look like `MyFunction`*12345678*, where *12345678* represents an eight\-character unique ID that the AWS CDK generates for all resources\. The line right after it should look like the following\.

   ```
   Type: AWS::Lambda::Function
   ```

1. Run the function by executing the following code\.

   ```
   sam local invoke MyFunction12345678 --no-event
   ```

   The output should look something like the following\.

   ```
   2019-04-01 12:22:41 Found credentials in shared credentials file: ~/.aws/credentials
   2019-04-01 12:22:41 Invoking app.lambda_handler (python3.7)
   
   Fetching lambci/lambda:python3.7 Docker container image......
   2019-04-01 12:22:43 Mounting D:\cdk-sam-example\.cdk.staging\a57f59883918e662ab3c46b964d2faa5 as /var/task:ro,delegated inside runtime container
   START RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72 Version: $LATEST
   END RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72
   REPORT RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72     Duration: 3.70 ms       Billed Duration: 100 ms Memory Size: 128 MB     Max Memory Used: 22 MB
   
   "This is a Lambda Function defined through CDK"
   ```