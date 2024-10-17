---
layout: post
title: Setting up CloudGoat
categories: Cloud
---

# Explanation
[CloudGoat](https://github.com/RhinoSecurityLabs/cloudgoat) is a Opensource "Vulnerable by design" AWS deployment tool.<br>
This explains how to setup the environment using CloudGoat on Kali Linux.

## Kali Version
```shell
kali@kali:~$ uname -a
Linux kali 6.8.11-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.8.11-1kali2 (2024-05-30) x86_64 GNU/Linux
```

## Terraform Installation
```shell
kali@kali:~$ mkdir terraform 

kali@kali:~/terraform$ wget https://releases.hashicorp.com/terraform/1.9.7/terraform_1.9.7_linux_386.zip
--2024-10-16 04:53:51--  https://releases.hashicorp.com/terraform/1.9.7/terraform_1.9.7_linux_386.zip
Resolving releases.hashicorp.com (releases.hashicorp.com)... 3.165.39.46, 3.165.39.6, 3.165.39.26, ...
Connecting to releases.hashicorp.com (releases.hashicorp.com)|3.165.39.46|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 24543390 (23M) [application/zip]
Saving to: ‘terraform_1.9.7_linux_386.zip’

terraform_1.9.7_linux_386.zip       100%[===================================================================>]  23.41M  68.0MB/s    in 0.3s    

2024-10-16 04:53:51 (68.0 MB/s) - ‘terraform_1.9.7_linux_386.zip’ saved [24543390/24543390]

                                                                                                        
kali@kali:~/terraform$ ls
terraform_1.9.7_linux_386.zip
                                                                                                        
kali@kali:~/terraform$ unzip terraform_1.9.7_linux_386.zip 
Archive:  terraform_1.9.7_linux_386.zip
  inflating: LICENSE.txt             
  inflating: terraform               
                                                  
kali@kali:~/terraform$ sudo cp terraform /usr/local/bin 
[sudo] password for kali: 
                                                                                                        
kali@kali:~/terraform$ terraform version            
Terraform v1.9.7
on linux_386
```

## JQ Installation
```shell
kali@kali:~$ sudo apt install jq 

<snip>

kali@kali:~$ jq --version
jq-1.7
```

## AWS-CLI Installation
```shell
kali@kali:~$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

<snip>

kali@kali:~$ aws --version              
aws-cli/2.18.7 Python/3.12.6 Linux/6.8.11-amd64 exe/x86_64.kali.2024
```

## CloudGoat Installation
```shell
 kali@kali:~$ sudo apt-get update

 <snip>

 kali@kali:~$ sudo apt install python3-venv

<snip>

kali@kali:~$ python3 -m venv .venv 

kali@kali:~$ git clone https://github.com/RhinoSecurityLabs/cloudgoat.git

kali@kali:~$ cd cloudgoat 

kali@kali:~/cloudgoat$ source ../.venv/bin/activate

(.venv)kali@kali:~/cloudgoat$ pip3 install -r ./requirements.txt 
Collecting argcomplete~=3.2.3 (from -r ./requirements.txt (line 5))
  Downloading argcomplete-3.2.3-py3-none-any.whl.metadata (16 kB)

<snip>
```
## AWS CLI setup
```shell
(.venv)kali@kali:~/cloudgoat$ aws configure              
AWS Access Key ID [None]: AKIARMY7LNBF5PZ5EDF2
AWS Secret Access Key [None]: <MASKED>
Default region name [None]: ap-northeast-1
Default output format [None]: 

(.venv)kali@kali:~/cloudgoat$ cat /home/kali/.aws/config 
[default]
region = ap-northeast-1
                                                                                                        
(.venv)kali@kali:~/cloudgoat$ cat /home/kali/.aws/credentials 
[default]
aws_access_key_id = AKIARMY7LNBF5PZ5EDF2
aws_secret_access_key = <MASKED>

(.venv)kali@kali:~/cloudgoat$ aws iam list-groups
{
    "Groups": [
        {
            "Path": "/",
            "GroupName": "administrators",
            "GroupId": "AGPARMY7LNBF7TF724LYP",
            "Arn": "arn:aws:iam::096165652555:group/administrators",
            "CreateDate": "2024-04-11T05:04:18+00:00"
        }
    ]
}
                                                                                               
(.venv)kali@kali:~/cloudgoat$                                       
```

## CloudGoat Setup
```shell
(.venv)kali@kali:~/cloudgoat$ ./cloudgoat.py config whitelist --auto
No whitelist.txt file was found at /home/kali/cloudgoat/whitelist.txt

CloudGoat can automatically make a network request, using https://ifconfig.co to find your IP address, and then overwrite the contents of the whitelist file with the result.
Would you like to continue? [y/n]: y

whitelist.txt created with IP address 106.185.157.100/32

(.venv)kali@kali:~/cloudgoat$ ./cloudgoat.py config profile         
No configuration file was found at /home/kali/cloudgoat/config.yml
Would you like to create this file with a default profile name now? [y/n]: y
Enter the name of your default AWS profile: default
A default profile name of "default" has been saved.
                                                                                                        
(.venv)kali@kali:~/cloudgoat$ 
```

## Create a LAB (example of vulnerable_lambda)
```shell
.venv)kali@kali:~/cloudgoat$ ./cloudgoat.py create vulnerable_lambda
Using default profile "default" from config.yml...
Loading whitelist.txt...
A whitelist.txt file was found that contains at least one valid IP address or range.
Initializing the backend...
Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Finding latest version of hashicorp/archive...
- Installing hashicorp/archive v2.6.0...
- Installed hashicorp/archive v2.6.0 (signed by HashiCorp)
- Installing hashicorp/aws v5.72.1...
- Installed hashicorp/aws v5.72.1 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

<snip>

[cloudgoat] terraform output completed with no error code.
cloudgoat_output_aws_account_id = 096165652555
cloudgoat_output_bilbo_access_key_id = AKIARMY7LNBF3HNSCLM4      # Credential we need for this LAB
cloudgoat_output_bilbo_secret_key = <MASKED>
profile = default
scenario_cg_id = vulnerable_lambda_cgid6wknt7nttr

[cloudgoat] Output file written to:

    /home/kali/cloudgoat/vulnerable_lambda_cgid6wknt7nttr/start.txt
```

## Listing all scenarios with deployed/undeployed information
```shell
(.venv)kali@kali:~/cloudgoat$ ./cloudgoat.py list all

  Deployed scenario instances: 1

    vulnerable_lambda
        CGID: cgid6wknt7nttr
        Path: /home/kali/cloudgoat/vulnerable_lambda_cgid6wknt7nttr

  Undeployed scenarios: 18

    sqs_flag_shop
    lambda_privesc
    sns_secrets
    scenario_template
    ec2_ssrf
    cloud_breach_s3
    cicd
    rds_snapshot
    ecs_efs_attack
    detection_evasion
    vulnerable_cognito
    codebuild_secrets
    ecs_takeover
    glue_privesc
    iam_privesc_by_attachment
    iam_privesc_by_key_rotation
    iam_privesc_by_rollback
    rce_web_app

                                                                                                        
(.venv)kali@kali:~/cloudgoat$
```

## Destroying the lab created
```shell
(.venv)kali@kali:~/cloudgoat$ ./cloudgoat.py destroy vulnerable_lambda
Using default profile "default" from config.yml...
Destroy "vulnerable_lambda_cgid6wknt7nttr"? [y/n]: y
data.archive_file.policy_applier_lambda1_zip: Reading...
data.archive_file.policy_applier_lambda1_zip: Read complete after 0s [id=af246c11811b83f16e4d3b0286731080306081d2]
data.aws_caller_identity.current: Reading...
aws_iam_user.bilbo: Refreshing state... [id=cg-bilbo-vulnerable_lambda_cgid6wknt7nttr]
aws_cloudwatch_log_group.policy_applier_lambda1: Refreshing state... [id=/aws/lambda/vulnerable_lambda_cgid6wknt7nttr-policy_applier_lambda1]
aws_secretsmanager_secret.final_flag: Refreshing state... [id=arn:aws:secretsmanager:us-east-1:096165652555:secret:vulnerable_lambda_cgid6wknt7nttr-final_flag-xkgVjU]
data.aws_caller_identity.current: Read complete after 1s [id=096165652555]
aws_secretsmanager_secret_version.final_flag_value: Refreshing state... [id=arn:aws:secretsmanager:us-east-1:096165652555:secret:vulnerable_lambda_cgid6wknt7nttr-final_flag-xkgVjU|terraform-20241017011455343700000002]
aws_iam_user_policy.standard_user: Refreshing state... [id=cg-bilbo-vulnerable_lambda_cgid6wknt7nttr:cg-bilbo-vulnerable_lambda_cgid6wknt7nttr-standard-user-assumer]
aws_iam_access_key.bilbo: Refreshing state... [id=AKIARMY7LNBF3HNSCLM4]
aws_iam_role.policy_applier_lambda1: Refreshing state... [id=vulnerable_lambda_cgid6wknt7nttr-policy_applier_lambda1]
aws_lambda_function.policy_applier_lambda1: Refreshing state... [id=vulnerable_lambda_cgid6wknt7nttr-policy_applier_lambda1]
aws_iam_role.cg-lambda-invoker: Refreshing state... [id=cg-lambda-invoker-vulnerable_lambda_cgid6wknt7nttr]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_cloudwatch_log_group.policy_applier_lambda1 will be destroyed
  - resource "aws_cloudwatch_log_group" "policy_applier_lambda1" {
      - arn               = "arn:aws:logs:us-east-1:096165652555:log-group:/aws/lambda/vulnerable_lambda_cgid6wknt7nttr-policy_applier_lambda1" -> null
      - id                = "/aws/lambda/vulnerable_lambda_cgid6wknt7nttr-policy_applier_lambda1" -> null
      - log_group_class   = "STANDARD" -> null
      - name              = "/aws/lambda/vulnerable_lambda_cgid6wknt7nttr-policy_applier_lambda1" -> null
      - retention_in_days = 14 -> null
      - skip_destroy      = false -> null
      - tags              = {
          - "Name" = "cg-vulnerable_lambda_cgid6wknt7nttr"
        } -> null
      - tags_all          = {
          - "Name"     = "cg-vulnerable_lambda_cgid6wknt7nttr"
          - "Scenario" = "vulnerable-lambda"
          - "Stack"    = "CloudGoat"
        } -> null
        # (2 unchanged attributes hidden)
    }

<snip>

```