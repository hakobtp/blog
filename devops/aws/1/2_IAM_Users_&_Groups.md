# IAM: Users & Groups

So, **IAM** stands for Identity and Access Management. It is a global service because in IAM we create users and put them into groups.

- IAM means Identity and Access Management.
- IAM is a global service. This means users and groups work in all regions, not just one.
- The root account is made when you create your AWS account. Do not use it for daily work and do not share it.
- Users are people or apps in your organization. You can put users into groups.
- Groups can only have users. They cannot have other groups.
- A user does not need to be in a group, and a user can be in many groups.

<p align="center">
    <img src="./assets/img2.png" alt="img2" width="500"/>
</p>

## IAM: Permissions

- Users or groups can have policies. Policies are JSON documents.
- Policies define what a user can do in AWS.
- AWS uses the least privilege principle: give users only the permissions they need, not more.

**Example JSON policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:Describe*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "elasticloadbalancing:Describe*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:ListMetrics",
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```

- `"Version": "2012-10-17"` **-** This is the policy language version. It tells AWS which version of JSON rules we are using.
- `"Statement"` **-** This is a list of rules. Each rule says what actions are allowed or denied.
- `"Effect": "Allow"` **-** This means the rule allows the action. It could also be `"Deny"` to block something.


---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [Understanding AWS Regions and Availability Zones](./1_Understanding_AWS_Regions_and_Availability_Zones.md)