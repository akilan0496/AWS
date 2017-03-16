# Code Deploy



Steps: 

 * Create User in IAM
 * Create IAM roles 
 * Create EC2 instance
 * Create Codedeploy-agent
 * Create Application to deploy in Code Deploy

##### Create User in IAM Steps:
1. createuser
2. assign following roles

	a. AmazonEC2FullAccess - AWS Managed policy
    
    b. AmazonEC2RoleforAWSCodeDeploy - AWS Managed policy
 	
    c. AWSCodeDeployFullAccess - AWS Managed policy
 
 	d. mycustompolicy - Inline policy
    permission:
    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "s3:Get*",
                    "s3:Put*",
                    "s3:List*"
                ],
                "Effect": "Allow",
                "Resource": "*"
            }
        ]
	}
    ```
3. download credentials

##### Create IAM Roles:

Create two custom roles to access EC2 instance and for Code Deploy

1. EC2InstanceRole

permission:
```
{ 
    "Version": "2012-10-17", 
    "Statement": [   
      {     
          "Action": [       
              "s3:Get*",       
              "s3:List*"     
          ],     
          "Effect": "Allow",     
          "Resource": "*"   
      } 
    ]
}
```

2. CodeDeployRole 

permission:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "autoscaling:PutLifecycleHook",
                "autoscaling:DeleteLifecycleHook",
                "autoscaling:RecordLifecycleActionHeartbeat",
                "autoscaling:CompleteLifecycleAction",
                "autoscaling:DescribeAutoscalingGroups",
                "autoscaling:PutInstanceInStandby",
                "autoscaling:PutInstanceInService",
                "ec2:Describe*"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```

Trusted Relation:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "codedeploy.ap-southeast-1.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Note: Here ap-southeast-1 is my location (Singapore)

##### Create EC2 instance:

Create EC2 Instance with Amazon linux.

Note: EC2InstanceRole should be assigned while creating.

##### Create Codedeploy-agent:

Login to Instance which created and execute the following commands.

```
#!/bin/bash

REGION=$(curl 169.254.169.254/latest/meta-data/placement/availability-zone/ | sed 's/[a-z]$//')

sudo yum update
sudo yum install python-pip
sudo yum install ruby
sudo yum install wget
cd /home/ec2-user
wget https://aws-codedeploy-$REGION.s3.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
```

##### Create Application to deploy in Code Deploy:

Create application - assign CodeDeployRole while creating.

Deploy (either via s3 or github)