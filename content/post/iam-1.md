---
title: "Introduction to AWS IAM: Users and Groups"
date: "2022-05-22"
description: "Discover the prowess of tr, sed, and awk as they join forces with grep, elevating your text processing skills to a whole new level."
featureImage: "img/2022/05/iam.png"
featureImageAlt: 'iam users and groups'
thumbnail: "img/2022/05/iam.png"
shareImage: "img/2022/05/iam.png"
categories:
  - Technology
tags:
  - aws
---

# Introduction to AWS IAM: Users and Groups

Identity and Access Management (IAM) is a fundamental service in AWS that allows you to manage access to your AWS resources securely. IAM is a global service, meaning it spans across all regions in your AWS account, ensuring a consistent access management experience wherever your resources are deployed.
## Understanding IAM Users

When you create an AWS account, a root user is automatically created. The root user has full access to all resources in the account and can perform any action, including managing billing information and creating additional accounts. However, for security best practices, it's recommended that the root user is only used for initial setup and billing purposes, and not for day-to-day operations.

Instead of using the root user, AWS IAM allows you to create Users within your account. Each user represents a single identity, typically one person, who can perform specific actions on your AWS resources based on the permissions assigned to them. Users in IAM are distinct from the root user and can be assigned to one or more groups.

## Organizing Users with IAM Groups

Groups in IAM are collections of users that share similar access permissions. For example, you might have a group for developers, another for system administrators, and another for support staff. By assigning users to groups, you can manage their permissions collectively rather than individually, simplifying the process of access management.

Groups in IAM are designed to hold users only; they cannot contain other groups. However, a user doesn't have to belong to any group and can belong to multiple groups if their role spans different functions.
## Assigning Permissions with IAM Policies

Both users and groups in IAM can be assigned permissions using Policies. Policies are JSON documents that define what actions users or groups are allowed to perform on which AWS resources. For example, you might create a policy that allows users to read data from an S3 bucket but not to delete it.

Here's a simple example of what an IAM policy might look like:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

In this example, the policy allows the user or group to perform the s3:GetObject action on any object in the example-bucket S3 bucket.

When assigning permissions, it’s crucial to follow the Principle of Least Privilege—only grant the permissions necessary for the user or group to perform their tasks and nothing more. This reduces the risk of accidental or malicious actions that could impact your AWS environment.
Practical Example: Setting Up Users, Groups, and Policies

Let’s walk through a simple demonstration where we create an IAM user, assign them to a group, and attach a policy to the group:

### Create a User: 
#### Start by creating a new user in your AWS account. For example, you might create a user named developer1.

Let's walk through the process of creating a new IAM user named "developer1" in your AWS account. We'll cover both the AWS Management Console method and the AWS CLI method.
Method 1: Using AWS Management Console

1. Sign in to the AWS Management Console and navigate to the IAM dashboard.
2. In the navigation pane, choose "Users" and then click "Add user".
3. For the user name, enter "developer1".
4. Select the type of access this user will need. For this example, let's choose both "Programmatic access" and "AWS Management Console access".
5. Set a custom password or let AWS generate one. Decide if you want the user to create a new password at next sign-in.
6. Click "Next: Permissions".
7. For this example, we won't add the user to a group yet. Click "Next: Tags".
8. (Optional) Add tags to the user for better organization.
9. Review the user details and click "Create user".
10. Make sure to save the access key ID and secret access key (if you chose programmatic access) and the console login link.

Method 2: Using AWS CLI

If you prefer using the command line, you can create a user with the AWS CLI. First, ensure you have the AWS CLI installed and configured with appropriate permissions.

1. To create a user with programmatic access:

```shell
aws iam create-user --user-name developer1
```

2. To create access keys for the user:
```shell
aws iam create-access-key --user-name developer1
``` 

This command will return an Access key ID and Secret access key. Make sure to save these securely.

3. To set a password for console access:
```shell
aws iam create-login-profile --user-name developer1 --password "InitialPassword123!" --password-reset-required
```

Replace "InitialPassword123!" with a strong initial password. The --password-reset-required flag ensures the user must change their password on first login.

4. To verify the user has been created:
```shell
aws iam get-user --user-name developer1
```

This command will return details about the user if it exists.

Remember, after creating the user, you'll need to grant appropriate permissions either by adding the user to a group with the necessary policies or by attaching policies directly to the user.

Safety note: Always follow the principle of least privilege when assigning permissions to new users. Start with minimal access and gradually increase permissions as needed.

By following these steps, you've successfully created a new IAM user named "developer1" in your AWS account. This user can now be assigned to groups and granted specific permissions as required for their role.


### Create a Group: 
#### Next, create a group named Developers and add developer1 to this group.

Let's walk through the process of creating a new IAM group named "Developers" and adding the "developer1" user to this group.

Method 1: Using AWS Management Console

1. Sign in to the AWS Management Console and navigate to the IAM dashboard.
2. In the navigation pane, choose "User groups" and then click "Create group".
3. For the group name, enter "Developers".
4. Optional) In the "Attach permissions policies" section, you can choose policies to attach to this group. For now, let's skip this step as we'll cover policies in the next part.
5. Click "Create group".
6. After the group is created, select the "Developers" group from the list.
7. In the "Users" tab, click "Add users".
8. Find and select the "developer1" user from the list.
9. Click "Add users" to confirm.

Method 2: Using AWS CLI

If you prefer using the command line, you can create a group and add a user with the AWS CLI. Ensure you have the AWS CLI installed and configured with appropriate permissions.

1. To create the Developers group:
```shell
aws iam create-group --group-name Developers
```
2. To verify the group has been created:
```shell
aws iam get-group --group-name Developers
```
3. To add the developer1 user to the Developers group:
```shell
aws iam add-user-to-group --user-name developer1 --group-name Developers
```
4. To verify that the user has been added to the group:
```shell
aws iam get-group --group-name Developers
```

This command will return details about the group, including a list of users in the group.

Additional CLI commands that might be useful:

* To list all groups in your account:
```shell
aws iam list-groups
```
* To remove a user from a group:
```shell
aws iam remove-user-from-group --user-name developer1 --group-name Developers
```
* To delete a group (note: you must remove all users and detach all policies first):
```shell
aws iam delete-group --group-name Developers
```
Remember, creating groups and adding users to them is just the first step. The real power of groups comes from attaching policies to them, which we'll cover in the next part about assigning permissions.

Best Practice: It's generally better to manage permissions at the group level rather than the individual user level. This makes it easier to maintain consistent permissions as your organization grows and changes.

By following these steps, you've successfully created a new IAM group named "Developers" and added the "developer1" user to this group. This structure allows you to manage permissions for all developers collectively, simplifying your IAM administration.
### Attach a Policy: 
#### Finally, attach a policy to the Developers group that grants read-only access to S3 buckets.

Let's attach the AWS managed policy "AmazonS3ReadOnlyAccess" to the "Developers" group. This policy grants read-only access to all S3 buckets in the account.

Method 1: Using AWS Management Console

1. Sign in to the AWS Management Console and navigate to the IAM dashboard.
2. In the navigation pane, choose "User groups".
3. Find and select the "Developers" group from the list.
4. In the "Permissions" tab, click "Add permissions" and then "Attach policies".
5. In the search box, type "AmazonS3ReadOnlyAccess".
6. Select the checkbox next to "AmazonS3ReadOnlyAccess" policy.
7. Click "Attach policy" at the bottom of the page.
8. You should now see the policy listed under the group's permissions.

Method 2: Using AWS CLI

If you prefer using the command line, you can attach a policy to a group with the AWS CLI. Ensure you have the AWS CLI installed and configured with appropriate permissions.

1. To attach the AmazonS3ReadOnlyAccess policy to the Developers group:
```shell
aws iam attach-group-policy --group-name Developers --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```
2. To verify that the policy has been attached to the group:
```shell
aws iam list-attached-group-policies --group-name Developers
```

This command will return a list of policies attached to the group.

Additional CLI commands that might be useful:

* To list all AWS managed policies:
```shell
aws iam list-policies --scope AWS --only-attached
```
* To get details about a specific policy:
```shell
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```
* To detach a policy from a group:
```shell
aws iam detach-group-policy --group-name Developers --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```


#### Creating a Custom Policy:

If you need more fine-grained control, you can create a custom policy. Here's an example of a custom policy that grants read-only access to all S3 buckets:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*"
            ],
            "Resource": "*"
        }
    ]
}
```

To create and attach this custom policy using CLI:

1. Save the policy JSON to a file named s3readonlypolicy.json
2. Create the policy:
```shell
aws iam create-policy --policy-name S3CustomReadOnlyAccess --policy-document file://s3readonlypolicy.json
```
3. Attach the custom policy to the group:
```shell
aws iam attach-group-policy --group-name Developers --policy-arn <ARN of the policy created in step 2>
```

Remember, it's crucial to regularly review and audit your IAM policies to ensure they align with the principle of least privilege.

By following these steps, you've successfully attached a policy to the "Developers" group that grants read-only access to S3 buckets. All users in this group, including "developer1", will now have this level of access to S3.

## Conclusion:

AWS Identity and Access Management (IAM) is a powerful and essential service for securing your cloud infrastructure. By leveraging IAM Users, Groups, and Policies, you can implement a robust and scalable access management strategy that adheres to the principle of least privilege.

In this post, we've explored the fundamental concepts of IAM:

* We learned about IAM Users, which represent individual identities within your AWS account.
* We discussed IAM Groups, which allow you to organize users and manage permissions collectively.
* We examined IAM Policies, the JSON documents that define the permissions for users and groups.

Through our practical example, we walked through the process of creating a user, assigning them to a group, and attaching a policy to grant specific permissions. This demonstrates how you can use IAM to create a structured and secure environment, even as your organization grows and evolves.

Remember, effective use of IAM is crucial for maintaining the security of your AWS resources. Always follow best practices such as:

* Regularly reviewing and auditing your IAM setup
* Implementing the principle of least privilege
* Using multi-factor authentication (MFA) for added security
* Rotating access keys periodically

By mastering IAM, you're not just managing access—you're building a foundation for secure and efficient cloud operations. As you continue your AWS journey, keep exploring more advanced IAM features like Roles and Identity Federation to further enhance your security posture.

Properly implemented, IAM becomes your first line of defense in the cloud, ensuring that only the right people have access to the right resources at the right time. With this knowledge, you're well-equipped to start implementing robust access management in your AWS environment.