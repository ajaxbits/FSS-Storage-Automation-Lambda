# Automation using Lambda to Deploy FSS- Storage Stack 

Automated process to deploy FSS Storage stack on each new S3 bucket using Lambda. The storage stack will be linked to the scanner stack previously defined.

![](architecture.png)
# Deploy via CloudFormation
   * If not already present, [deploy a Scanner Stack](https://cloudone.trendmicro.com/docs/file-storage-security/stack-add/#AddScanner) in your Cloud One Account
   * Obtain the stack's Cloud One ID and SQSURL
      - The Stack ID and SQSURL can be obtained with list stacks api call
      - See [FSS API Documentation](https://cloudone.trendmicro.com/docs/file-storage-security/api-create-stack/) for details.
   * In AWS Console > Services > CloudFormation
    - **Create New Stack**
      - Prerequisites: *template is ready*
      - Specify Template: *upload from file*
      - Select: **[storage_stack.yaml](https://github.com/JustinDPerkins/FSS-Storage-Automation-Lambda/blob/main/cloudformation/storage_stack.yaml)**
      - **Next**
      - StackName: *enter name for stack*
      - C1API: [`Cloud One API Key`](https://cloudone.trendmicro.com/docs/container-security/api-key-create/)
      - SQSURL: `http://scanner-stack-sqs-queue-url.com`
      - StackID: `Scanner Stack ID`
      - **Create Stack**
      
# Deploy via AWS Console
<details>
<summary>Create IAM Role/Policy for lambda execution</summary>

   * In AWS Console > Services > IAM
      - In left panel under **Access Management** click **Policy**
      - Click **Create Policy**
         - Select **JSON**
         - paste **[fss_policy](https://github.com/JustinDPerkins/FSS-Storage-Automation-Lambda/blob/main/policy/fss_policy.json)**
         - Save
   * In AWS Console > Services > IAM
      - In left panel under **Access Management** click **Roles**
      - Create Role
         - Entity: AWS Service
         - Service: Lambda
         - select policy made previously
         - Save
</details>

<details>
<summary>Create Lambda Function</summary>

   * In AWS Console > Services > Lambda >  Create Function

      - Select **Author from scratch**
      - Function Name: *example-name*
      - Runtime: **Python 3.8**
      - Select Service role: **Select role created in previous steps**
      - Create Function
   * Under Code
      - Copy and Paste: **[storage_stack.yaml](https://github.com/JustinDPerkins/FSS-Storage-Automation-Lambda/blob/main/cloudformation/storage_stack.yaml)**
      - Deploy
   * Under Configuration
      - Environment Variables
      - **C1-API** : *your fss api key*
      - **SQS_Name** : *scanner sqs url*
      - **STACK_ID** : *scanner stack id*
      - See [FSS API Documentation](https://cloudone.trendmicro.com/docs/file-storage-security/api-create-stack/) for details.
      - Configuration
      - General configuration > Edit
      - increase timeout to 8m
</details>

<details>
<summary>Create CloudWatch Rule</summary>

   * In AWS Console > Services > CloudWatch
      - In left panel under **Events** click **Rules**
   * Create New CloudWatch Rule
      - Event Source: Event Pattern
      - Service Name: S3
      - Event Type: Bucket Level Operations
      - Specific Operation(s): CreateBucket
   * Add Target
      - Select: **Lambda Function**
      - Choose the function made in  first step
      - leave the rest to defaults
   * Configure Rule details
      - Name: *example-rule-name*
      - Description: 
      - State: **Enabled**
      - Create rule 
</details>

# A Note on Tags

The Lambda will choose whether or not to deploy a storage stack depending on a bucket's tags. See below for details:

| Tag            | Value  | Behavior                       |
| -------------- | ------ | ------------------------------ |
| [no tag]       | [none] | Storage Stack deployed         |
| `FSSMonitored` | `yes`  | Storage Stack deployed         |
| `FSSMonitored` | `no`   | Storage Stack **not** deployed |

The script will add the proper tags automatically to untagged buckets, but you can *exclude* buckets by adding a `FSSMonitored` == `no` tag. 
