# About validators<a name="appconfig-creating-configuration-and-profile-validators"></a>

When you create a configuration and configuration profile, you can specify up to two validators\. A validator ensures that your configuration data is syntactically and semantically correct\. You can create validators in either JSON Schema or as an AWS Lambda function\.

**Important**  
Configuration data stored in SSM documents must validate against an associated JSON Schema before you can add the configuration to the system\. SSM parameters do not require a validation method, but we recommend that you create a validation check for new or updated SSM parameter configurations by using AWS Lambda\.

**JSON Schema Validators**

If you create a configuration in an SSM document, then you must specify or create a JSON Schema for that configuration\. A JSON Schema defines the allowable properties for each application configuration setting\. The JSON Schema functions like a set of rules to ensure that new or updated configuration settings conform to the best practices required by your application\. Here is an example\. 

```
    {
      "$schema": "http://json-schema.org/draft-04/schema#",
      "title": "$id$",
      "description": "BasicFeatureToggle-1",
      "type": "object",
      "additionalProperties": false,
      "patternProperties": {
          "[^\\s]+$": {
              "type": "boolean"
          }
      },
      "minProperties": 1
    }
```

When you create the configuration by using the procedure in this topic, AWS AppConfig verifies that the configuration conforms to the schema requirements\. If it doesn't, Systems Manager returns a validation error\.

**Note**  
AWS AppConfig supports JSON Schema version 4\.X for inline schema\. If your application configuration requires a different version of JSON Schema, then you must create a Lambda validator\.

**AWS Lambda Validators**

Lambda function validators must be configured to expect the following event schema\. The content is a base64\-encoded string, and the URI is a string\. 

```
{
    "applicationId": "The application Id of the configuration profile being validated", 
    "configurationProfileId": "The configuration profile Id of the configuration profile being validated",
    "configurationVersion": "The configuration version of the configuration profile being validated",
    "content": "Base64EncodedByteString", 
    "uri": "The uri of the configuration"    
}
```

AWS AppConfig verifies that the Lambda `X-Amz-Function-Error` header is set in the response\. Lambda sets this header if the function throws an exception\. For more information about `X-Amz-Function-Error`, see [Error Handling and Automatic Retries in AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/retries-on-errors.html) in the *AWS Lambda Developer Guide*\.

Here is a simple example of a Lambda response code for a successful validation\.

```
import json

def handler(event, context):
     #Add your validation logic here
     print("We passed!")
```

Here is a simple example of a Lambda response code for an unsuccessful validation\.

```
def handler(event, context):
     #Add your validation logic here
     raise Exception("Failure!")
```

Here is another example that validates only if the configuration parameter is a prime number\.

```
function isPrime(value) {
    if (value < 2) {
        return false;
    }

    for (i = 2; i < value; i++) {
        if (value % i === 0) {
            return false;
        }
    }

    return true;
}

exports.handler = async function(event, context) {
    console.log('EVENT: ' + JSON.stringify(event, null, 2));
    const input = parseInt(Buffer.from(event.content, 'base64').toString('ascii'));
    const prime = isPrime(input);
    console.log('RESULT: ' + input + (prime ? ' is' : ' is not') + ' prime');
    if (!prime) {
        throw input + "is not prime";
    }
}
```

AWS AppConfig calls your validation Lambda when calling the `StartDeployment` and `ValidateConfigurationActivity` API actions\. You must provide `appconfig.amazonaws.com` permissions to invoke your Lambda\. For more information, see [Granting Function Access to AWS Services](https://docs.aws.amazon.com/lambda/latest/dg/access-control-resource-based.html#permissions-resource-serviceinvoke)\. AWS AppConfig limits the validation Lambda run time to 15 seconds, including start\-up latency\.
