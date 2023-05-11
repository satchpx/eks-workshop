---
title: "Access your Workspace"
chapter: false
weight: 5
---

<!---
{{% notice info %}}
This workshop was designed to run in the **Oregon (us-west-2)** region. **Please don't
run in any other region.** Future versions of this workshop will expand region availability,
and this message will be removed.
{{% /notice %}}
-->

{{% notice tip %}}
Ad blockers, javascript disablers, and tracking blockers should be disabled for
the cloud9 domain, or connecting to the workspace might be impacted.
Cloud9 requires third-party-cookies. You can whitelist the [specific domains]( https://docs.aws.amazon.com/cloud9/latest/user-guide/troubleshooting.html#troubleshooting-env-loading).
{{% /notice %}}

### Access your Cloud9 instance:

In the AWS Management Console, access your Cloud9 instance and customize the environment by:

- Closing the **Welcome tab**
![c9before](/images/prerequisites/cloud9-1.png)
- Opening a new **terminal** tab in the main work area
![c9newtab](/images/prerequisites/cloud9-2.png)
- Closing the lower work area
![c9newtab](/images/prerequisites/cloud9-3.png)
- Your workspace should now look like this
![c9after](/images/prerequisites/cloud9-4.png)

### Update IAM settings for your Workspace

{{% notice info %}}
Cloud9 normally manages IAM credentials dynamically. This isn't currently compatible with
the EKS IAM authentication, so we will disable it and rely on the IAM role instead.
{{% /notice %}}

- Return to your Cloud9 workspace and click the gear icon (in top right corner)
- Select **AWS SETTINGS**
- Turn off **AWS managed temporary credentials**
- Close the Preferences tab
![c9disableiam](/images/prerequisites/c9disableiam.png)

To ensure temporary credentials aren't already in place we will also remove
any existing credentials file:

```sh
rm -vf ${HOME}/.aws/credentials
```

We should configure our aws cli with our current region as default.

{{% notice info %}}
If you are [at an AWS event](https://eksworkshop.com/020_prerequisites/aws_event/), ask your instructor which **AWS region** to use.
{{% /notice %}}

```sh
yum install -y jq
export ACCOUNT_ID=$(aws sts get-caller-identity --output text --query Account)
export AWS_REGION=$(curl -s 169.254.169.254/latest/dynamic/instance-identity/document | jq -r '.region')
export AZS=($(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text --region $AWS_REGION))

```

Check if AWS_REGION is set to desired region

```sh
test -n "$AWS_REGION" && echo AWS_REGION is "$AWS_REGION" || echo AWS_REGION is not set
```

 Let's save these into bash_profile

```sh
echo "export ACCOUNT_ID=${ACCOUNT_ID}" | tee -a ~/.bash_profile
echo "export AWS_REGION=${AWS_REGION}" | tee -a ~/.bash_profile
echo "export AZS=(${AZS[@]})" | tee -a ~/.bash_profile
aws configure set default.region ${AWS_REGION}
aws configure get default.region
```

### Validate the IAM role

Use the [GetCallerIdentity](https://docs.aws.amazon.com/cli/latest/reference/sts/get-caller-identity.html) CLI command to validate that the Cloud9 IDE is using the correct IAM role.

```bash
aws sts get-caller-identity --query Arn | grep eksworkshop-admin -q && echo "IAM role valid" || echo "IAM role NOT valid"
```

If the IAM role is not valid, <span style="color: red;">**DO NOT PROCEED**</span>. Go back and confirm the steps on this page.
Create an IAM role called `eksworkshop-admin` for EC2 service with `Administrator Access` -> Attach this role to the Cloud9 EC2 instance.

{{% notice info %}}
If you intend to run all the sections in this workshop, it will be useful to have more storage available for all the repositories and tests.
{{% /notice %}}

### Increase the disk size on the Cloud9 instance

{{% notice note %}}
The following command adds more disk space to the root volume of the EC2 instance that Cloud9 runs on. Once the command completes, we reboot the instance and it could take a minute or two for the IDE to come back online.
{{% /notice %}}

```bash
pip3 install --user --upgrade boto3
export instance_id=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
python -c "import boto3
import os
from botocore.exceptions import ClientError 
ec2 = boto3.client('ec2')
volume_info = ec2.describe_volumes(
    Filters=[
        {
            'Name': 'attachment.instance-id',
            'Values': [
                os.getenv('instance_id')
            ]
        }
    ]
)
volume_id = volume_info['Volumes'][0]['VolumeId']
try:
    resize = ec2.modify_volume(    
            VolumeId=volume_id,    
            Size=30
    )
    print(resize)
except ClientError as e:
    if e.response['Error']['Code'] == 'InvalidParameterValue':
        print('ERROR MESSAGE: {}'.format(e))"
if [ $? -eq 0 ]; then
    sudo reboot
fi

```
