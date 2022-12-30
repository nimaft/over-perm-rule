## IAM Empty role config rule
AWS Config rule to detect overly permissive IAM managed policies or IAM role inline policies. Triggered if sts:AssumeRole action is used with last character of resource element being '*', created using rule [AWS Config Rules Development Kit](https://github.com/awslabs/aws-config-rdk).

## Deployment
1. Make sure you have [rdk](https://rdk.readthedocs.io/en/latest/getting_started.html#installation) installed
2. Clone this repository
3. Navigate to cloned repository directory
2. run rdk deploy IAM_OVERLY_PERMISSIVE_PERMISSIONS

This will deploy your rule to AWS Config, and you can track deployment progress in your terminal. For more information, view Deploy in RDK Command Reference documentation.

## Rule properties
### Resources in scope
AWS IAM Role, AWS IAM Policy
### Rule triggers
* Configuration item change: runs every time an in-scope resource is changed. For example, when a role is created or modified. Ignores deleted resources. When the rule is triggered by configuration item change, it only assesses the changed resource except for the first run when the rule assesses all the configuration items for resources in scope, stored in AWS config.

### Parameters
This rule does not take any parameters.
### Logic
For configuration item change assessments, configuration item is used to assess compliance.

#### Configuration item change assessments:
[`evaluate_compliance`](IAM_OVERLY_PERMISSIVE_PERMISSIONS/IAM_OVERLY_PERMISSIVE_PERMISSIONS.py#L30) function receives the configuration item. It first checks to verify the resource type triggering the rule. Following section reviews assessment method for each  resource type.

#### IAM Role assessment
For IAM roles:
1. The function lists the names of the inline policies that are embedded in the specified IAM role using boto3 iam client [`list_role_policies`](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.list_role_policies) method.
2. It then uses [`get_role_policy`](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.get_role_policy) method for each role's policy to retrieve the specified inline policy document that is embedded with the specified IAM role. 
3. It loops over policy statements to find `sts:AssumeRole` action used with `Resource` element's last character being euql to `*`. This way both `Resource: “*”` and `Resource: "arn:aws:iam::123456789012:role/*"` would be picked and marked as non-compliant.

Note that `list_role_policies` method only returns role's inline policy, and managed policies are ignored because they are assessed when the rule is triggered by AWS::IAM::Policy resource type.

#### IAM Policy assessment
For IAM policies:
1. The function uses [`get_policy_version`](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.get_policy_version) method of boto3 iam client to eetrieve information about the specified version of the specified managed policy, including the policy document. 
2. It loops over policy statements to find `sts:AssumeRole` action used with `Resource` element's last character being euql to `*`. This way both `Resource: “*”` and `Resource: "arn:aws:iam::123456789012:role/*"` would be picked and marked as non-compliant.