# AWS Config Rules for Deep Security

A set of AWS Config Rules to help ensure that your AWS deployments are leveraging the protection of Deep Security. These rules help centralize your compliance information in one place, AWS Config.


## Table of Contents

* [Architecture](#architecture)
* [Setup](#setup)
* [Support](#support)
* [Contribute](#contribute)

## Setup

## Create a User in Deep Security

During execution, the AWS Lambda functions will query the Deep Security API. To do this, they require a Deep Security login with permissions.

You should set up a dedicated use account for API access. To configure the account with the minimum privileges (which reduces the risk if the credentials are exposed) required by this integration, follow the steps below.

1. In Deep Security, go to *Administration > User Manager > Roles*. 
1. Click **New**. Create a new role with a unique, meaningful name.
1. Under **Access Type**, select **Allow Access to web services API**.
1. Under **Access Type**, deselect **Allow Access to Deep Security Manager User Interface**.
1. On the **Computer Rights** tab, select either **All Computers** or **Selected Computers**, ensuring that only the greyed-out **View** right (under **Allow Users to**) is selected.
1. On the **Policy Rights** tab, select **Selected Policies**. Verify that no policies are selected. (The role does not grant rights for any policies.)
1. On the **User Rights** tab, select **Change own password and contact information only**.
1. On the **Other Rights* tab, verify that the default options remain, with only **View-Only** and **Hide** permissions.
1. Go to **Administration > User Manager > Users**.
1. Click **New**. Create a new user with a unique, meaningful name.
1. Select the role that you created in the previous section.


### Configure AWS Lambda

Configure these for each rule:

- Handler: filename_for_the_rule.aws_config_rule_handler
- Role: a role with at least the rights as shown in [dsConfigRulePolicy.json](/dsConfigRulePolicy.json). **Remember** to change line 18 to reflect your S3 bucket information (BUCKET/PATH/TO/OBJECTS/*)
- (Advanced Settings) Memory: 128 MB
- (Advanced Settings) Timeout: 3m 0s

### Rules

#### ds-IsInstanceProtectedByAntiMalware

Checks to see if the current instance is protected by Deep Security Anti-Malware controls. Anti-malware must be "on" and in "real-time" mode for the rule to be considered compliant.

Lambda handler: **dsIsInstanceProtectedByAntiMalware.aws_config_rule_handler**

##### Rule Parameters:

<table>
<tr>
  <th>Rule Parameter</th>
  <th>Expected Value Type</th>
  <th>Description</th>
</tr>
<tr>
  <td>dsUsername</td>
  <td>string</td>
  <td>The username of the Deep Security account to use for querying anti-malware status</td>
</tr>
<tr>
  <td>dsPassword</td>
  <td>string</td>
  <td>The password for the Deep Security account to use for querying anti-malware status. This password is readable by any identity that can access the AWS Lambda function. Use only the bare minimum permissions within Deep Security (see note below)</td>
</tr>
<tr>
  <td>dsPasswordKey</td>
  <td>string or URI</td>
  <td>The encrypted data encryption key used to encrypt the <code>dsPassword</code>. If this is specified, the rule will first decrypt the <code>dsPasswordKey</code> and then decrypt the <code>dsPassword</code> using the value. See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for more details.
</tr>
<tr>
  <td>dsPasswordEncryptionContext</td>
  <td>string or URI</td>
  <td>The encryption context used to encrypt the <code>dsPassword</code>. If this parameter is given, the rule will include the encryption context information when decrypting the <code>dsPassword</code> value. Requires <code>dsPasswordKey</code> to be useful. See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for more details.
</tr>
<tr>
  <td>dsTenant</td>
  <td>string</td>
  <td><i>Optional as long as dsHostname is specified</i>. Indicates which tenant to sign in to within Deep Security</td>
</tr>
<tr>
  <td>dsHostname</td>
  <td>string</td>
  <td><i>Optional as long as dsTenant is specified</i>. Defaults to Deep Security as a Service. Indicates which Deep Security manager the rule should sign in to</td>
</tr>
<tr>
  <td>dsPort</td>
  <td>int</td>
  <td><i>Optional</i>. Defaults to 443. Indicates the port to connect to the Deep Security manager on</td>
</tr>
<tr>
  <td>dsIgnoreSslValidation</td>
  <td>int (0 or 1)</td>
  <td><i>Optional</i>. 0 for false, 1 for true. Use only when connecting to a Deep Security manager that is using a self-signed SSL certificate</td>
</tr>
</table>

During execution, this rule sign in to the Deep Security API. You should setup a dedicated API access account to do this. Deep Security contains a robust role-based access control (RBAC) framework which you can use to ensure that this set of credentials has the least amount of privileges to success.

This rule requires view access to one or more computers within Deep Security.

#### ds-IsInstanceProtectedBy

Checks to see if the current instance is protected by any of Deep Security's controls. Controls must be "on" and set to their strongest setting (a/k/a "real-time" or "prevention") in order for the rule to be considered compliant.

This is the generic version of *ds-IsInstanceProtectedByAntiMalware*.

Lambda handler: **dsIsInstanceProtectedBy.aws_config_rule_handler**

##### Rule Parameters:

<table>
<tr>
  <th>Rule Parameter</th>
  <th>Expected Value Type</th>
  <th>Description</th>
</tr>
<tr>
  <td>dsUsername</td>
  <td>string</td>
  <td>The username of the Deep Security account to use for querying anti-malware status</td>
</tr>
<tr>
  <td>dsPassword</td>
  <td>string or URI</td>
  <td>The password for the Deep Security account to use for querying anti-malware status. This password is readable by any identity that can access the AWS Lambda function. Use only the bare minimum permissions within Deep Security (see note below)</td>
</tr>
<tr>
  <td>dsPasswordKey</td>
  <td>string or URI</td>
  <td>The encrypted data encryption key used to encrypt the <code>dsPassword</code>. If this is specified, the rule will first decrypt the <code>dsPasswordKey</code> and then decrypt the <code>dsPassword</code> using the value. See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for more details.
</tr>
<tr>
  <td>dsPasswordEncryptionContext</td>
  <td>string or URI</td>
  <td>The encryption context used to encrypt the <code>dsPassword</code>. If this parameter is given, the rule will include the encryption context information when decrypting the <code>dsPassword</code> value. Requires <code>dsPasswordKey</code> to be useful. See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for more details.
</tr>
<tr>
  <td>dsTenant</td>
  <td>string</td>
  <td><i>Optional as long as dsHostname is specified</i>. Indicates which tenant to sign in to within Deep Security</td>
</tr>
<tr>
  <td>dsHostname</td>
  <td>string</td>
  <td><i>Optional as long as dsTenant is specified</i>. Defaults to Deep Security as a Service. Indicates which Deep Security manager the rule should sign in to</td>
</tr>
<tr>
  <td>dsPort</td>
  <td>int</td>
  <td><i>Optional</i>. Defaults to 443. Indicates the port to connect to the Deep Security manager on</td>
</tr>
<tr>
  <td>dsIgnoreSslValidation</td>
  <td>int (0 or 1)</td>
  <td><i>Optional</i>. 0 for false, 1 for true. Use only when connecting to a Deep Security manager that is using a self-signed SSL certificate</td>
</tr>
<tr>
  <td>dsControl</td>
  <td>string</td>
  <td>The name of the control to verify. Must be one of [ anti_malware, web_reputation, firewall, intrusion_prevention, integrity_monitoring, log_inspection ]</td>
</tr>
</table>

During execution, this rule signs in to the Deep Security API. You should setup a dedicated API access account to do this. Deep Security contains a robust role-based access control (RBAC) framework which you can use to ensure that this set of credentials has the least amount of privileges to success.

This rule requires view access to one or more computers within Deep Security.

#### ds-DoesInstanceHavePolicy

Checks to see if the current instance is protected by a specific Deep Security policy.

Lambda handler: **dsDoesInstanceHavePolicy.aws_config_rule_handler**

##### Rule Parameters:

<table>
<tr>
  <th>Rule Parameter</th>
  <th>Expected Value Type</th>
  <th>Description</th>
</tr>
<tr>
  <td>dsUsername</td>
  <td>string</td>
  <td>The username of the Deep Security account to use for querying anti-malware status</td>
</tr>
<tr>
  <td>dsPassword</td>
  <td>string</td>
  <td>The password for the Deep Security account to use for querying anti-malware status. This password is readable by any identity that can access the AWS Lambda function. Use only the bare minimum permissions within Deep Security (see note below)</td>
</tr>
<tr>
  <td>dsPasswordKey</td>
  <td>string or URI</td>
  <td>The encrypted data encryption key used to encrypt the <code>dsPassword</code>. If this is specified, the rule will first decrypt the <code>dsPasswordKey</code> and then decrypt the <code>dsPassword</code> using the value. See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for more details.
</tr>
<tr>
  <td>dsPasswordEncryptionContext</td>
  <td>string or URI</td>
  <td>The encryption context used to encrypt the <code>dsPassword</code>. If this parameter is given, the rule will include the encryption context information when decrypting the <code>dsPassword</code> value. Requires <code>dsPasswordKey</code> to be useful. See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for more details.
</tr>
<tr>
  <td>dsTenant</td>
  <td>string</td>
  <td><i>Optional as long as dsHostname is specified</i>. Indicates which tenant to sign in to within Deep Security</td>
</tr>
<tr>
  <td>dsHostname</td>
  <td>string</td>
  <td><i>Optional as long as dsTenant is specified</i>. Defaults to Deep Security as a Service. Indicates which Deep Security manager the rule should sign in to</td>
</tr>
<tr>
  <td>dsPort</td>
  <td>int</td>
  <td><i>Optional</i>. Defaults to 443. Indicates the port to connect to the Deep Security manager on</td>
</tr>
<tr>
  <td>dsIgnoreSslValidation</td>
  <td>int (0 or 1)</td>
  <td><i>Optional</i>. 0 for false, 1 for true. Use only when connecting to a Deep Security manager that is using a self-signed SSL certificate</td>
</tr>
<tr>
  <td>dsPolicy</td>
  <td>string</td>
  <td>The name of the policy to verify</td>
</tr>
</table>

During execution, this rule signs in to the Deep Security API. You should setup a dedicated API access account to do this. Deep Security contains a robust role-based access control (RBAC) framework which you can use to ensure that this set of credentials has the least amount of privileges to success.

This rule requires view access to one or more computers within Deep Security.

#### ds-IsInstanceClear

Checks to see if the current instance is has any warnings, alerts, or errors in Deep Security. An instance is compliant if it does **not** have any warnings, alerts, or errors (a/k/a compliant, which means everything is working as expected with no active security alerts).

Lambda handler: **dsIsInstanceClear.aws_config_rule_handler**

##### Rule Parameters:

<table>
<tr>
  <th>Rule Parameter</th>
  <th>Expected Value Type</th>
  <th>Description</th>
</tr>
<tr>
  <td>dsUsername</td>
  <td>string</td>
  <td>The username of the Deep Security account to use for querying anti-malware status</td>
</tr>
<tr>
  <td>dsPassword</td>
  <td>string</td>
  <td>The password for the Deep Security account to use for querying anti-malware status. This password is readable by any identity that can access the AWS Lambda function. Use only the bare minimum permissions within Deep Security (see note below)</td>
</tr>
<tr>
  <td>dsPasswordKey</td>
  <td>string or URI</td>
  <td>The encrypted data encryption key used to encrypt the <code>dsPassword</code>. If this is specified, the rule will first decrypt the <code>dsPasswordKey</code> and then decrypt the <code>dsPassword</code> using the value. See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for more details.
</tr>
<tr>
  <td>dsPasswordEncryptionContext</td>
  <td>string or URI</td>
  <td>The encryption context used to encrypt the <code>dsPassword</code>. If this parameter is given, the rule will include the encryption context information when decrypting the <code>dsPassword</code> value. Requires <code>dsPasswordKey</code> to be useful. See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for more details.
</tr>
<tr>
  <td>dsTenant</td>
  <td>string</td>
  <td><i>Optional as long as dsHostname is specified</i>. Indicates which tenant to sign in to within Deep Security</td>
</tr>
<tr>
  <td>dsHostname</td>
  <td>string</td>
  <td><i>Optional as long as dsTenant is specified</i>. Defaults to Deep Security as a Service. Indicates which Deep Security manager the rule should sign in to</td>
</tr>
<tr>
  <td>dsPort</td>
  <td>int</td>
  <td><i>Optional</i>. Defaults to 443. Indicates the port to connect to the Deep Security manager on</td>
</tr>
<tr>
  <td>dsIgnoreSslValidation</td>
  <td>int (0 or 1)</td>
  <td><i>Optional</i>. 0 for false, 1 for true. Use only when connecting to a Deep Security manager that is using a self-signed SSL certificate</td>
</tr>
</table>

During execution, this rule signs in to the Deep Security API. You should setup a dedicated API access account to do this. Deep Security contains a robust role-based access control (RBAC) framework which you can use to ensure that this set of credentials has the least amount of privileges to success.

This rule requires view access to one or more computers within Deep Security.

### Risk of Credentials in AWS Config

If you're curious about the wisdom of storing access credentials in a 3rd party service... good. You've got your security hat on. Let's take a look at the risks.

Right now, Deep Security uses its role-based access control to provide access to its APIs. This means we need to provide our AWS Lambda functions with some way of getting a set of credentials.

Because Deep Security sits outside of the AWS IAM structure (in other words, it's not an AWS service), we can either:

1. hard-code the credentials inside the AWS Lambda function
1. pass the credentials to the function (current method)
1. put the credentials somewhere else and provide access to that location to the function

Option #1 removes the credentials from AWS Config and moves them to AWS Lambda. This allows for the segregation of duties via IAM access to these services. In turn it creates a significant maintenance challenge in the event you change credentials.

Option #2 (current method) stores the credentials in AWS Config. Any [user or role with sufficient privileges](http://docs.aws.amazon.com/config/latest/developerguide/recommended-iam-permissions-using-aws-config-console-cli.html) can access AWS Config and the stored credentials. This is the same situation as option #1 but doesn't present the same maintenance issue.

Option #3 would put the credentials in an alternative location (like S3 encrypted by KMS) and significantly reduces the possibility of the credentials leaking out. Using IAM, you can ensure that only the Lambda execution role used for the function has access to the encryption key and the S3 key (object in the bucket).

In a discussion of option #1 vs #2, option #2 is the better choice as the maintenance issues presented by hard coding credentials are significant. If segregation is required, option #3 is far superior as it directly addresses the security issues of broader access to the credentials.

The risk posed by exposing the credentials to AWS Config can be partially mitigated by reducing the permissions that the credentials hold in Deep Security (see above, "Permissions In Deep Security"). However, option #3 (storing the credentials in S3 and encrypting with KMS) is a much better option.

See [Protecting Your Deep Security Manager API Password](#protecting-your-deep-security-manager-api-password) below for instructions on how to implement option #3.

<a name="protecting-your-deep-security-manager-api-password" />

### Protect Your Deep Security Manager API Password

You can set up with an encrypted password.

You may have noticed that the rules give you the option to specify your password as a string **or URI**, and that there are optional parameters for specifying an encryption key and an encryption context, each of which can also be provided as string or URI.

For details, see the [AWS Key Management Service documentation](https://aws.amazon.com/documentation/kms/).

In the AWS Key Management Service (AWS KMS), you create a [Customer Master Key](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys). This master key remains securely stored by AWS KMS, and can be used to generate [data keys](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys) that are then used to encrypt individual data items, in this case your password. The encrypted data key is normally stored alongside the encrypted data. You can also optionally provide an [encryption context](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context) to allow more fine-grained access to the encrypted data.

If you provide encryption context, you should provide the key-value pairs as a JSON object, like this:
```
{ "key1": "value1", "key2": "value2" }
```

If you provide an password encryption key, the rule will first call AWS KMS to decrypt the password encryption key, and will then decrypt the password. If you have provided an encryption context, that information will be provided when decrypting your password encryption key.

AWS S3 is a great place to keep your encrypted password and encrypted password key. To do this, simply provide the S3 URI to the objects when you're specifying the rule parameter values. For example:
<table>
<tr>
<td><code>dsPassword</code></td>
<td><code>s3://bucket/path/to/password.enc</code></td>
</tr>
<tr>
<td><code>dsPasswordKey</code></td>
<td><code>s3://bucket/path/to/password.key</code></td>
</tr>
<tr>
<td><code>dsPasswordEncryptionContext</code></td>
<td><code>s3://bucket/path/to/password.ctx</code></td>
</tr>
</table>

If you want to keep the encryption context separate, you can put the value directly into the rule parameters as a JSON string:
<table>
<tr>
<td><code>dsPassword</code></td>
<td><code>s3://bucket/path/to/password.enc</code></td>
</tr>
<tr>
<td><code>dsPasswordKey</code></td>
<td><code>s3://bucket/path/to/password.key</code></td>
</tr>
<tr>
<td><code>dsPasswordEncryptionContext</code></td>
<td><code>{ "key1": "value1", "key2": "value2" }</code></td>
</tr>
</table>

There are some tools in the `tools` directory that will help you create and verify an encrypted password and an encrypted password key.

**IMPORTANT**: Make sure that the IAM role assigned to your rule has permissions to read the S3 bucket and objects, and also to access the KMS `Decrypt` API and the key.

**ALSO IMPORTANT**: The entire set of configuration rule parameters must be less than 256 characters when encoded as JSON.

## Support

This is an Open Source community project. Project contributors may be able to help, 
depending on their time and availability. Please be specific about what you're 
trying to do, your system, and steps to reproduce the problem.

For bug reports or feature requests, please 
[open an issue](../issues). 
You are welcome to [contribute](#contribute).

Official support from Trend Micro is not available. Individual contributors may be 
Trend Micro employees, but are not official support.

## Contribute

We accept contributions from the community. To submit changes:

1. Fork this repository.
1. Create a new feature branch.
1. Make your changes.
1. Submit a pull request with an explanation of your changes or additions.

We will review and work with you to release the code.
