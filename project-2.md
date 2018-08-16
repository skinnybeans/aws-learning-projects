# Project 2

## Typey Jenkins

Welcome to project 2!

For project 2 the goals will be largely the same: Get an EC2 instance up and running then get Jenkins running on it.

However! This time we will try doing it though the aws command line!

For some extra points, lets install Jenkins without logging onto the instance.

## Prerequisites

There are a few more things to get working this time.

### AWS command line

The AWS command line utility allows us to interact with AWS using the..... command line!


There are steps to set it up over on the AWS site:

https://docs.aws.amazon.com/cli/latest/userguide/installing.html

It can be a bit tricky to get working.

When it's all good to go you should be able to run the following command and see some nice output:

```bash
aws ec2 describe-regions --output table
```

### Ansible

To assist with the automation we are going to use Ansible.

Ansible runs on Python, so you can install it in a similar way to the AWS CLI.

```
pip install ansible
```

To check everything has gone well try running:
```
ansible --version
```

## View from above

The top level view looks similar as for project 1, except this time Ansible will be doing the instance configuration.

1. Create EC2 instance.

1. Use Ansible to:

    - Update instance packages.

    - Install Jenkins.

    - Start the Jenkins service.

1. Unlock the Jenkins console.

1. Celebrate!

## Steps - with more

### 1. Create EC2 instance

If you remember way back to project 1, there were a few things we needed to consider when creating an EC2 instance:

  - Choosing the AMI and instance type.
  - Choosing a key pair to use.
  - Configuring the security group to allow traffic on the ports required.

When going through the launch instance wizard, actions such as creating the security group and key pair were taken care of automatically. When doing things through the command line, there isn't as much hand holding done.

The call to launch an instance looks like this:
```bash
aws ec2 run-instances --image-id $image_id --security-group-ids $security_group_id --count 1 --instance-type $instance_type --key-name $key_name
```
There are four pieces of information we must provide:

1. image-id  
    - The ID of the AMI to use.
1. security-group-id
    - The ID of the security group to use.  
1. instance-type
    - Type of instance to use.
1. key-name
    - The ssh key pair to use.

Here are some tips on how to find this information. There are answers in the **Help** section at the bottom of the document.

#### Image ID

There are ways to find image IDs from the AWS command line, but actually pinning the correct one can be difficult.

Perhaps there's a way to find the ID of the image we used when launching and instance through the console? Log into the console and see what you can find.

#### Security Group ID

We could use the same security group that we used for project 1. That would be no fun... Instead lets have some fun and create a new one.

There are two things we need to have a functioning security group:

1. Create the security group.
1. Add the connectivity rules we need.

AWS provides really good documentation for the command line, you can find it here: https://docs.aws.amazon.com/cli/latest/reference/

Security groups can be found under the EC2 section.

Make sure you record the security group ID for use later.

If you are having trouble figuring out how to create the security group, check out the help section at the end of the document.



## create a security group
```bash
aws ec2 create-security-group --description 'jenkins group' --group-name 'test-group'
```

Remember security group id for later
"GroupId": "sg-21bd3b59"

## add an ingress rule for SSH to the security group
```bash
aws ec2 authorize-security-group-ingress --group-id "sg-21bd3b59" --protocol "tcp" --port "22" --cidr "0.0.0.0/0"
```

## Launch an instance!

### Find the AMI ID to use for amazon linux 2
`ami-39f8215b`

### create the instance
```bash
aws ec2 run-instances --image-id 'ami-39f8215b' --security-group-ids 'sg-21bd3b59' --count 1 --instance-type t2.micro --key-name "MyEC2KeyPair" --query 'Instances[0].InstanceId'
```
_remember your instance-id_
i-0b4e5483cd5c2a905

### to connect to it we now need the instance ip address

#### find the IP address
aws ec2 describe-instances --query 'Reservations[*].Instances[?InstanceId==`i-0b4e5483cd5c2a905`].NetworkInterfaces[*].Association.PublicIp'

_remember your ip-address_
13.211.62.213


## Now connect!

```bash
ssh -i _keyname_ ec2-user@_ip-address_
```

ssh -i MyEC2KeyPair ec2-user@13.211.62.213

## do the jenkins install and run things here..


## now connect to the management console
http://13.211.62.213:8080

## need to add port 8080 to the security group

```bash
aws ec2 authorize-security-group-ingress --group-id "sg-21bd3b59" --protocol "tcp" --port "8080" --cidr "0.0.0.0/0"
```


## Stop the instance
```bash
aws ec2 stop-instances --instance-id "i-0b4e5483cd5c2a905"
```

## Check the instance is stopped

```bash
aws ec2 describe-instances --instance-id "i-0b4e5483cd5c2a905" --query Reservations[0].Instances[0].State.Name
```

## Terminate the instance

```bash
aws ec2 terminate-instances --instance-id "i-0b4e5483cd5c2a905"
```

## Check the status again
```bash
aws ec2 describe-instances --instance-id "i-0b4e5483cd5c2a905" --query Reservations[0].Instances[0].State.Name
```