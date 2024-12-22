---
title: "Introduction to AWS IAM: Users and Groups"
description: "Discover the prowess of tr, sed, and awk as they join forces with grep, elevating your text processing skills to a whole new level."
date: "2022-05-21"
draft: "true"
featured_image: "img/2022/05/grep_2.png"
---
Introduction to AWS IAM: Users and Groups

Identity and Access Management (IAM) is a fundamental service in AWS that allows you to manage access to your AWS resources securely. IAM is a global service, meaning it spans across all regions in your AWS account, ensuring a consistent access management experience wherever your resources are deployed.
Understanding IAM Users

When you create an AWS account, a root user is automatically created. The root user has full access to all resources in the account and can perform any action, including managing billing information and creating additional accounts. However, for security best practices, it's recommended that the root user is only used for initial setup and billing purposes, and not for day-to-day operations.

Instead of using the root user, AWS IAM allows you to create Users within your account. Each user represents a single identity, typically one person, who can perform specific actions on your AWS resources based on the permissions assigned to them. Users in IAM are distinct from the root user and can be assigned to one or more groups.
Organizing Users with IAM Groups

Groups in IAM are collections of users that share similar access permissions. For example, you might have a group for developers, another for system administrators, and another for support staff. By assigning users to groups, you can manage their permissions collectively rather than individually, simplifying the process of access management.

Groups in IAM are designed to hold users only; they cannot contain other groups. However, a user doesn't have to belong to any group and can belong to multiple groups if their role spans different functions.
Assigning Permissions with IAM Policies

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

    Create a User: Start by creating a new user in your AWS account. For example, you might create a user named developer1.

    Create a Group: Next, create a group named Developers and add developer1 to this group.

    Attach a Policy: Finally, attach a policy to the Developers group that grants read-only access to S3 buckets.

By following these steps, you ensure that all developers in your account have the necessary access to S3 buckets without giving them permissions they don’t need.
