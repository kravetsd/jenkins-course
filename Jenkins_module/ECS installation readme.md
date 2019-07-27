###1. Create  ECS Cluster named Jenkins:

#[mentor@vagrant ~]$ aws ecs create-cluster --cluster-name Jenkins
{
    "cluster": {
        "clusterArn": "arn:aws:ecs:us-east-1:264112291867:cluster/Jenkins",
        "clusterName": "Jenkins",
        "status": "ACTIVE",
        "registeredContainerInstancesCount": 0,
        "runningTasksCount": 0,
        "pendingTasksCount": 0,
        "activeServicesCount": 0,
        "statistics": [],
        "tags": []
    }
}


###2. Setting up ECR repository:
#[mentor@vagrant ~]$ aws ecr describe-repositories
{
    "repositories": []
}


#[mentor@vagrant ~]$ aws ecr create-repository --repository-name  mp2019-jenkins
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-1:264112291867:repository/mp2019-jenkins",
        "registryId": "264112291867",
        "repositoryName": "mp2019-jenkins",
        "repositoryUri": "264112291867.dkr.ecr.us-east-1.amazonaws.com/mp2019-jenkins",
        "createdAt": 1563604475.0
    }
}


#[mentor@vagrant ~]$ aws ecr describe-repositories
{
    "repositories": [
        {
            "repositoryArn": "arn:aws:ecr:us-east-1:264112291867:repository/mp2019-jenkins",
            "registryId": "264112291867",
            "repositoryName": "mp2019-jenkins",
            "repositoryUri": "264112291867.dkr.ecr.us-east-1.amazonaws.com/mp2019-jenkins",
            "createdAt": 1563604475.0
        }
    ]
}





###3. Create an appropriate role to assignt to your EC2 container isntace:
#Trust relationship policy:
#[mentor@vagrant ~]$  aws iam create-role --role-name  ecsInstanceRole  --assume-role-policy-document file://ecs-trust-ec2-role-policy.json
{
    "Role": {
        "Path": "/",
        "RoleName": "ecsInstanceRole",
        "RoleId": "AROAT27SRYAN3WIVR4F2R",
        "Arn": "arn:aws:iam::264112291867:role/ecsInstanceRole",
        "CreateDate": "2019-07-20T10:08:41Z",
        "AssumeRolePolicyDocument": {
            "Version": "2008-10-17",
            "Statement": [
                {
                    "Sid": "",
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}


#cat > ecs-trust-ec2-role-policy.json
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}



#[mentor@vagrant ~]$ aws iam put-role-policy --role-name ecsInstanceRole \
                           --policy-name ecsInstancePolicy \
                           --policy-document file://AmazonEC2ContainerServiceforEC2Role.json


#Create IAM role AmazonEC2ContainerServiceforEC2Role  for your container isntace to assignt:
#cat > AmazonEC2ContainerServiceforEC2Role.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecs:CreateCluster",
        "ecs:DeregisterContainerInstance",
        "ecs:DiscoverPollEndpoint",
        "ecs:Poll",
        "ecs:RegisterContainerInstance",
        "ecs:StartTelemetrySession",
        "ecs:Submit*",
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}




###Creating instance profile:
#[mentor@vagrant ~]$ aws iam create-instance-profile --instance-profile-name ecsInstanceProfile
{
    "InstanceProfile": {
        "Path": "/",
        "InstanceProfileName": "ecsInstanceProfile",
        "InstanceProfileId": "AIPAT27SRYAN72SJILKMN",
        "Arn": "arn:aws:iam::264112291867:instance-profile/ecsInstanceProfile",
        "CreateDate": "2019-07-20T10:47:55Z",
        "Roles": []
    }
}

#Attachong role to instance profile:
#[mentor@vagrant ~]$ aws iam add-role-to-instance-profile --instance-profile-name ecsInstanceProfile \
                                     --role-name ecsInstanceRole







To stay in scope of free tier you need to launch containers only in free tier eligible instances. This is the only aws ecs optimized instance.
Starting EC2 instance and conect it to your cluster. Thios is needed to stay in scope of free tier. 

##4. Create EFS filesystem 
```
[mentor@vagrant ~]$ aws efs create-file-system \
 --creation-token FileSystemForWalkthrough1 \
 --tags Key=Name,Value="data"


{
    "OwnerId": "264112291867",
    "CreationToken": "FileSystemForWalkthrough1",
    "FileSystemId": "fs-c6a4b425",
    "CreationTime": 1563627845.0,
    "LifeCycleState": "creating",
    "Name": "data",
    "NumberOfMountTargets": 0,
    "SizeInBytes": {
        "Value": 0,
        "ValueInIA": 0,
        "ValueInStandard": 0
    },
    "PerformanceMode": "generalPurpose",
    "Encrypted": false,
    "ThroughputMode": "bursting",
    "Tags": [
        {
            "Key": "Name",
            "Value": "data"
        }
    ]
}

```

#Enable transitioning to IA after 30 days as a best pracetice 
```
[mentor@vagrant ~]$ aws efs put-lifecycle-configuration \
> --file-system-id fs-c6a4b425 \
> --lifecycle-policies TransitionToIA=AFTER_30_DAYS
{
    "LifecyclePolicies": [
        {
            "TransitionToIA": "AFTER_30_DAYS"
        }
    ]
}
```


##5. Run EC2 instance with user-data:
#
```
[mentor@vagrant ~]$ aws ec2 run-instances --instance-type t2.micro \
                          --image-id ami-0c6b1d09930fac512 \
                          --key-name mp2019-keypair-y \
                          --security-group-ids sg-0fef7b240b2937f8a \
                          --subnet-id subnet-04bb2555dda3a9a5d \
                          --associate-public-ip-address \
                          --user-data file://prepare-for-ecs.sh \
                          --iam-instance-profile Name=ecsInstanceProfile
```
#Output:
```
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0c6b1d09930fac512",
            "InstanceId": "i-03643821a711e3cd9",
            "InstanceType": "t2.micro",
            "KeyName": "mp2019-keypair-y",
            "LaunchTime": "2019-07-20T10:52:14.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1a",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-123-1-219.ec2.internal",
            "PrivateIpAddress": "10.123.1.219",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-04bb2555dda3a9a5d",
            "VpcId": "vpc-0043424991adf8b6c",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "",
            "EbsOptimized": false,
            "Hypervisor": "xen",
            "IamInstanceProfile": {
                "Arn": "arn:aws:iam::264112291867:instance-profile/ecsInstanceProfile",
                "Id": "AIPAT27SRYAN72SJILKMN"
            },
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2019-07-20T10:52:14.000Z",
                        "AttachmentId": "eni-attach-071d3431fc0bee6c6",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching"
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "Admin",
                            "GroupId": "sg-0fef7b240b2937f8a"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "12:96:20:f1:4e:ae",
                    "NetworkInterfaceId": "eni-03de89b1fd693426f",
                    "OwnerId": "264112291867",
                    "PrivateDnsName": "ip-10-123-1-219.ec2.internal",
                    "PrivateIpAddress": "10.123.1.219",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-10-123-1-219.ec2.internal",
                            "PrivateIpAddress": "10.123.1.219"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-04bb2555dda3a9a5d",
                    "VpcId": "vpc-0043424991adf8b6c",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "Admin",
                    "GroupId": "sg-0fef7b240b2937f8a"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            }
        }
    ],
    "OwnerId": "264112291867",
    "ReservationId": "r-0fa338f68441ec0bd"
}
```




##Creating a task definitionf or ECS jenkins task
```
[mentor@vagrant ~]$ cat > task-def.json
{
    "family": "jenkins",
    "networkMode": "bridge",
    "containerDefinitions": [
        {
          "name": "jenkins-master",
          "image": "jenkins",
          "mountPoints": [
            {
              "sourceVolume": "data-volume",
              "containerPath": "/var/jenkins_home"
            }
          ],
          "essential": true,
          "cpu": 1024,
          "memoryReservation": 768,
          "portMappings": [
            {
              "hostPort": 8080,
              "containerPort": 8080,
              "protocol": "tcp"
            },
            {
              "hostPort": 50000,
              "containerPort": 50000,
              "protocol": "tcp"
            }
          ]
        }
      ],
    "volumes":[
        {
          "host": {
            "sourcePath": "/data/"
          },
          "name": "data-volume"
        }
      ],
    "requiresCompatibilities": [
        "EC2"
    ]
}
```


```
[mentor@vagrant ~]$ aws ecs register-task-definition --cli-input-json file://task-def.json
{
    "taskDefinition": {
        "taskDefinitionArn": "arn:aws:ecs:us-east-1:264112291867:task-definition/jenkins:2",
        "containerDefinitions": [
            {
                "name": "jenkins-master",
                "image": "jenkins",
                "cpu": 1024,
                "memoryReservation": 768,
                "portMappings": [
                    {
                        "containerPort": 8080,
                        "hostPort": 8080,
                        "protocol": "tcp"
                    },
                    {
                        "containerPort": 50000,
                        "hostPort": 50000,
                        "protocol": "tcp"
                    }
                ],
                "essential": true,
                "environment": [],
                "mountPoints": [
                    {
                        "sourceVolume": "data-volume",
                        "containerPath": "/var/jenkins_home"
                    }
                ],
                "volumesFrom": []
            }
        ],
        "family": "jenkins",
        "networkMode": "bridge",
        "revision": 2,
        "volumes": [
            {
                "name": "data-volume",
                "host": {
                    "sourcePath": "/data/"
                }
            }
        ],
        "status": "ACTIVE",
        "requiresAttributes": [
            {
                "name": "com.amazonaws.ecs.capability.docker-remote-api.1.21"
            }
        ],
        "placementConstraints": [],
        "compatibilities": [
            "EC2"
        ],
        "requiresCompatibilities": [
            "EC2"
        ]
    }
}

```

#output
```
[mentor@vagrant ~]$ aws ecs run-task --cluster Jenkins --task-definition arn:aws:ecs:us-east-1:264112291867:task-definition/jenkins:2 --launch-type EC2
{
    "tasks": [
        {
            "taskArn": "arn:aws:ecs:us-east-1:264112291867:task/4e2365b9-6c41-49ca-9e4d-d6ee5fd35a7a",
            "clusterArn": "arn:aws:ecs:us-east-1:264112291867:cluster/Jenkins",
            "taskDefinitionArn": "arn:aws:ecs:us-east-1:264112291867:task-definition/jenkins:2",
            "containerInstanceArn": "arn:aws:ecs:us-east-1:264112291867:container-instance/cd5df1dc-285b-4e1f-aa55-779ffd9448d3",
            "overrides": {
                "containerOverrides": [
                    {
                        "name": "jenkins-master"
                    }
                ]
            },
            "lastStatus": "PENDING",
            "desiredStatus": "RUNNING",
            "cpu": "1024",
            "memory": "768",
            "containers": [
                {
                    "containerArn": "arn:aws:ecs:us-east-1:264112291867:container/197a7d23-1944-4ea7-b4ff-06e07101f732",
                    "taskArn": "arn:aws:ecs:us-east-1:264112291867:task/4e2365b9-6c41-49ca-9e4d-d6ee5fd35a7a",
                    "name": "jenkins-master",
                    "lastStatus": "PENDING",
                    "networkInterfaces": [],
                    "cpu": "1024",
                    "memoryReservation": "768"
                }
            ],
            "version": 1,
            "createdAt": 1563627184.357,
            "group": "family:jenkins",
            "launchType": "EC2",
            "attachments": [],
            "tags": []
        }
    ],
    "failures": []
}
```



How to create your custom ami using console and aws-cli:
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html


Usefull aws cli commands:
create-image  - https://docs.aws.amazon.com/cli/latest/reference/ec2/create-image.html

Installing Jenkins contenerized deployment:
https://docs.aws.amazon.com/whitepapers/latest/jenkins-on-aws/containerized-deployment.html



ecample user-data to install ecs-agent:
https://docs.amazonaws.cn/en_us/AmazonECS/latest/developerguide/example_user_data_scripts.html



AWS examples of tsk definitions in json:
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/example_task_definitions.html



###failure got:
[mentor@vagrant ~]$ aws ecs run-task --cluster Jenkins --task-definition arn:aws:ecs:us-east-1:264112291867:task-definition/jenkins:1 --launch-type EC2
{
    "tasks": [],
    "failures": [
        {
            "arn": "arn:aws:ecs:us-east-1:264112291867:container-instance/cd5df1dc-285b-4e1f-aa55-779ffd9448d3",
            "reason": "RESOURCE:MEMORY"
        }
    ]
}



Creation EFS and mounting it to EC2
https://docs.aws.amazon.com/efs/latest/ug/wt1-create-efs-resources.html