# Step 3: Creating a configuration and a configuration profile<a name="appconfig-creating-configuration-and-profile"></a>

A *configuration* is a collection of settings that influence the behavior of your application\. For example, you can create and deploy configurations that carefully introduce changes to your application or turn on new features that require a timely deployment, such as a product launch or announcement\. Here's a very simple example of an access list configuration\. 

```
{
    "AccessList": [
        {
            "user_name": "Mateo_Jackson"
        },
        {
            "user_name": "Jane_Doe"
        }
    ]
}
```

A *configuration profile* enables AWS AppConfig to access your configuration from a source location\. You can store configurations in the following formats and locations:
+ YAML, JSON, or text documents in the AWS AppConfig hosted configuration store
+ Objects in an Amazon Simple Storage Service \(Amazon S3\) bucket
+ Documents in the Systems Manager document store
+ Parameters in Parameter Store
+ Any [integration source action](https://docs.aws.amazon.com/codepipeline/latest/userguide/integrations-action-type.html#integrations-source) supported by AWS CodePipeline

A configuration profile includes the following information\.
+ The URI location where the configuration is stored\. 
+ The AWS Identity and Access Management \(IAM\) role that provides access to the configuration\.
+ A validator for the configuration data\. You can use either a JSON Schema or an AWS Lambda function to validate your configuration profile\. A configuration profile can have a maximum of two validators\.

For configurations stored in the AWS AppConfig hosted configuration store or SSM documents, you can create the configuration by using the Systems Manager console at the time you create a configuration profile\. The process is described later in this topic\. 

For configurations stored in SSM parameters or in S3, you must create the parameter or object first and then add it to Parameter Store or S3\. After you create the parameter or object, you can use the procedure in this topic to create the configuration profile\. For information about creating a parameter in Parameter Store, see [Creating Systems Manager parameters](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-su-create.html) in the *AWS Systems Manager User Guide*\. 

## About configuration store quotas and limitations<a name="appconfig-creating-configuration-and-profile-quotas"></a>

AWS AppConfig\-supported configuration store have the following quotas and limitations\.


****  

|  | AWS AppConfig hosted configuration store | S3 | Parameter Store | Document store | AWS CodePipeline | 
| --- | --- | --- | --- | --- | --- | 
|  **Configuration size limit**  | 1 MB |  1 MB Enforced by AWS AppConfig, not S3  |  4 KB \(free tier\) / 8 KB \(advanced parameters\)  |  64 KB  | 1 MBEnforced by AWS AppConfig, not CodePipeline | 
|  **Resource storage limit**  | 1 GB |  Unlimited  |  10,000 parameters \(free tier\) / 100,000 parameters \(advanced parameters\)  |  500 documents  | Limited by the number of configuration profiles per application \(100 profiles per application\) | 
|  **Server\-side encryption**  | Yes |  [SSE\-S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/serv-side-encryption.html)  |  No  |  No  | Yes | 
|  **AWS CloudFormation support**  | Yes |  Not for creating or updating data  |  Yes  |  No  | Yes | 
|  **Validate create or update API actions**  | Not supported |  Not supported  |  Regex supported  |  JSON Schema required for all put and update API actions  | Not supported | 
|  **Pricing**  | Free |  See [Amazon S3 pricing](https://aws.amazon.com//s3/pricing/)  |  See [AWS Systems Manager pricing](https://aws.amazon.com//systems-manager/pricing/)  |  Free  |  See [AWS CodePipeline pricing](https://aws.amazon.com//codepipeline/pricing/)  | 

## About the AWS AppConfig hosted configuration store<a name="appconfig-creating-configuration-and-profile-about-hosted-store"></a>

AWS AppConfig includes an internal or hosted configuration store\. Configurations must be 64 KB or smaller\. The AWS AppConfig hosted configuration store provides the following benefits over other configuration store options\. 
+ You don't need to set up and configure other services such as Amazon Simple Storage Service \(Amazon S3\) or Parameter Store\.
+ You don't need to configure AWS Identity and Access Management \(IAM\) permissions to use the configuration store\.
+ You can store configurations in YAML, JSON, or as text documents\.
+ There is no cost to use the store\.
+ You can create a configuration and add it to the store when you create a configuration profile\.

## Creating a configuration and a configuration profile<a name="appconfig-creating-configuration-and-profile-create"></a>

**Before you begin**  
Read the following related content before you complete the procedure in this section\.
+ The following procedure requires you to specify an IAM service role so that AWS AppConfig can access your configuration data in the configuration store you choose\. This role is not required if you use the AWS AppConfig hosted configuration store\. If you choose S3, Parameter Store, or the Systems Manager document store, then you must either choose an existing IAM role or choose the option to have the system automatically create the role for you\. For more information, about this role, see [About the configuration profile IAM role](appconfig-creating-configuration-and-profile-iam-role.md)\.
+ If you want to create a configuration profile for configurations stored in S3, you must configure permissions\. For more information about permissions and other requirements for using S3 as a configuration store, see [About configurations stored in Amazon S3](appconfig-creating-configuration-and-profile-S3-source.md)\.
+ If you want to use validators, review the details and requirements for using them\. For more information, see [About validators](appconfig-creating-configuration-and-profile-validators.md)\.

### Creating an AWS AppConfig configuration profile \(console\)<a name="appconfig-creating-configuration-and-profile-create-console"></a>

Use the following procedure to create an AWS AppConfig configuration profile and \(optionally\) a configuration by using the AWS Systems Manager console\.

**To create a configuration profile**

1. Open the AWS Systems Manager console at [https://console\.aws\.amazon\.com/systems\-manager/appconfig/](https://console.aws.amazon.com/systems-manager/appconfig/)\.

1. On the **Applications** tab, choose the application you created in [Create an AWS AppConfig configuration](appconfig-creating-application.md) and then choose the **Configuration profiles** tab\.

1. Choose **Create configuration profile**\.

1. For **Name**, enter a name for the configuration profile\.

1. For **Description**, enter information about the configuration profile\.

1. On the **Select configuration source** page, choose an option\.

1. 
   + If you selected **AWS AppConfig hosted configuration**, then choose either **YAML**, **JSON**, or **Text**, and enter your configuration in the field\. Choose **Next** and go to Step 10 in this procedure\.
   + If you selected **Amazon S3 object**, then enter the object URI\. Choose **Next**\.
   + If you selected **AWS Systems Manager parameter**, then choose the name of the parameter from the list\. Choose **Next**\.
   + If you selected **AWS CodePipeline**, then choose **Next** and go to Step 10 in this procedure\. 
   + If you selected **AWS Systems Manager document**, then complete the following steps\. 

   1. In the **Document source** section, choose either **Saved document** or **New document**\. 

   1. If you choose **Saved document**, then choose the SSM document from the list\. If you choose **New document**, the **Details** and **Content** sections appear\.

   1. In the **Details** section, enter a name for the new application configuration\.

   1. For the **Application configuration schema** section, either choose the JSON schema using the list or choose **Create schema**\. If you choose **Create schema**, Systems Manager opens the **Create schema** page\. Enter the schema details in the **Content** section, and then choose **Create schema**\.  
![\[Enter details of an AWS AppConfig configuration\]](http://docs.aws.amazon.com/appconfig/latest/userguide/images/appconfig-profile-2.png)

   1. For **Application configuration schema version** either choose the version from the list or choose **Update schema** to edit the schema and create a new version\.

   1. In the **Content** section, choose either **YAML** or **JSON** and then enter the configuration data in the field\.  
![\[Enter configuration data in an AWS AppConfig configuration profile\]](http://docs.aws.amazon.com/appconfig/latest/userguide/images/appconfig-profile-3.png)

   1. Choose **Next**\.

1. In the **Service role** section, choose **New service role** to have AWS AppConfig create the IAM role that provides access to the configuration data\. AWS AppConfig automatically populates the **Role name** field based on the name you entered earlier\. Or, to choose a role that already exists in IAM, choose **Existing service role**\. Choose the role by using the **Role ARN** list\.

1. On the **Add validators** page, choose either **JSON Schema** or **AWS Lambda**\. If you choose **JSON Schema**, enter the JSON Schema in the field\. If you choose **AWS Lambda**, choose the function Amazon Resource Name \(ARN\) and the version from the list\. 
**Important**  
Configuration data stored in SSM documents must validate against an associated JSON Schema before you can add the configuration to the system\. SSM parameters do not require a validation method, but we recommend that you create a validation check for new or updated SSM parameter configurations by using AWS Lambda\.

1. In the **Tags** section, enter a key and an optional value\. You can specify a maximum of 50 tags for a resource\. 

1. Choose **Create configuration profile**\.

**Important**  
If you created a configuration profile for AWS CodePipeline, then after you create a deployment strategy, as described in the next section, you must create a pipeline in CodePipeline that specifies AWS AppConfig as the *deploy provider*\. For information about creating a pipeline that specifies AWS AppConfig as the deploy provider, see [Tutorial: Create a Pipeline That Uses AWS AppConfig as a Deployment Provider](https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-AppConfig.html) in the *AWS CodePipeline User Guide*\. 

Proceed to [Step 4: Creating a deployment strategy](appconfig-creating-deployment-strategy.md)\.

### Creating an AWS AppConfig configuration profile \(commandline\)<a name="appconfig-creating-configuration-and-profile-create-commandline"></a>

The following procedure describes how to use the AWS CLI \(on Linux or Windows\) or AWS Tools for PowerShell to create a AWS AppConfig configuration profile\.

**To create a configuration profile step by step**

1. Install and configure the AWS CLI or the AWS Tools for PowerShell, if you have not already\.

   For information, see [Install or upgrade AWS command line tools](getting-started-cli.md)\.

1. Run the following command to create a configuration profile\. 

------
#### [ Linux ]

   ```
   aws appconfig create-configuration-profile \
     --application-id The_application_ID \
     --name A_name_for_the_configuration_profile \
     --description A_description_of_the_configuration_profile \
     --location-uri A_URI_to_locate_the_configuration \
     --retrieval-role-arn The_ARN_of_the_IAM_role_with_permission_to_access_the_configuration_at_the_specified_LocationUri \
     --tags User_defined_key_value_pair_metadata_of_the_configuration_profile \
     --validators "Content=JSON_Schema_content_or_the_ARN_of_an_AWS Lambda_function,Type=validators_of_type_JSON_SCHEMA_and_LAMBDA"
   ```

------
#### [ Windows ]

   ```
   aws appconfig create-configuration-profile ^
     --application-id The_application_ID ^
     --name A_name_for_the_configuration_profile ^
     --description A_description_of_the_configuration_profile ^
     --location-uri A_URI_to_locate_the_configuration ^
     --retrieval-role-arn The_ARN_of_the_IAM_role_with_permission_to_access_the_configuration_at_the_specified_LocationUri ^
     --tags User_defined_key_value_pair_metadata_of_the_configuration_profile ^
     --validators "Content=JSON_Schema_content_or_the_ARN_of_an_AWS Lambda_function,Type=validators_of_type_JSON_SCHEMA_and_LAMBDA"
   ```

------
#### [ PowerShell ]

   ```
   New-APPCConfigurationProfile `
     -Name A_name_for_the_configuration_profile `
     -ApplicationId The_application_ID `
     -Description Description_of_the_configuration_profile `
     -LocationUri A_URI_to_locate_the_configuration `
     -RetrievalRoleArn The_ARN_of_the_IAM_role_with_permission_to_access_the_configuration_at_the_specified_LocationUri `
     -Tag Hashtable_type_user_defined_key_value_pair_metadata_of_the_configuration_profile `
     -Validators "Content=JSON_Schema_content_or_the_ARN_of_an_AWS Lambda_function,Type=validators_of_type_JSON_SCHEMA_and_LAMBDA"
   ```

------

   The system returns information like the following\.

------
#### [ Linux ]

   ```
   {
      "ApplicationId": "The application ID",
      "Id": "The configuration profile ID",
      "Name": "The name of the configuration profile",
      "Description": "The configuration profile description",
      "LocationUri": "The URI location of the configuration",
      "RetrievalRoleArn": "The ARN of an IAM role with permission to access the configuration at the specified LocationUri",
      "Validators": [ 
         { 
            "Content": "The JSON Schema content or the ARN of a Lambda function",
            "Type": "Validators of type JSON_SCHEMA and LAMBDA"
         }
      ]
   }
   ```

------
#### [ Windows ]

   ```
   {
      "ApplicationId": "The application ID",
      "Id": "The configuration profile ID",
      "Name": "The name of the configuration profile",
      "Description": "The configuration profile description",
      "Id": "The configuration profile ID",
      "LocationUri": "The URI location of the configuration",
      "RetrievalRoleArn": "The ARN of an IAM role with permission to access the configuration at the specified LocationUri",
      "Validators": [ 
         { 
            "Content": "The JSON Schema content or the ARN of a Lambda function",
            "Type": "Validators of type JSON_SCHEMA and LAMBDA"
         }
      ]
   }
   ```

------
#### [ PowerShell ]

   ```
   ApplicationId    : The application ID
   ContentLength    : Runtime of the command
   Description      : The configuration profile description
   HttpStatusCode   : HTTP Status of the runtime
   Id               : The configuration profile ID
   LocationUri      : The URI location of the configuration
   Name             : The name of the configuration profile
   ResponseMetadata : Runtime Metadata
   RetrievalRoleArn : The ARN of an IAM role with permission to access the configuration at the specified LocationUri
   Validators       : {Content: The JSON Schema content or the ARN of a Lambda function, Type : Validators of type JSON_SCHEMA and LAMBDA}
   ```

------