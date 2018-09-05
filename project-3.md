# Project 1

## Bakey Jenkins

- build an AMI with jenkins baked in
- create the ec2 from baked ami and security group using cloudformation

### Baking AMI using packer

Read about packer here: https://www.packer.io/

Basics:
- Used for creating images.
- has tools for creating amazon AMIs

How does it work:
- creates and instance in AWS
- runs a set of commands on that instances
- creates and AMI from the instance
- terminates the instance.

### How to create an AMI the slow way

- manually create ec2
- run YUM update
- take snapshot
- create image
- create instance from image
- try updating image again
- notice that the state has been saved.

- want to use this for jenkins so anytime we want to create a new jenkins can just be done by recreating the instance from the AMI

### Basic packer script to build ami with yum updates

```json
{
    "variables": {
        "some-var": "yo"
    },
    "builders": [{
      "type": "amazon-ebs",
      "region": "ap-southeast-2",
      "source_ami_filter": {
        "filters": {
        "virtualization-type": "hvm",
        "name": "amzn2-ami-hvm*",
        "root-device-type": "ebs"
        },
        "owners": ["137112412989"],
        "most_recent": true
      },
      "instance_type": "t2.micro",
      "ssh_username": "ec2-user",
      "ami_name": "packer-example {{timestamp}}"
    }]
}
```

output should give the AMI ID.

Look up the AMI details using the AWS cli

```
aws ec2 describe-images --image-id "ami-0f8beed61ccbdb89d"
```

Or if you want to see all the AMIs for your account:

```
aws ec2 describe-images --filters Name=owner-id,Values=348201094690
```

Look at the snapshots available to the account:

```
 aws ec2 describe-snapshots --filters Name=owner-id,Values=348201094690
```

To get your account number from the cli:
```
 aws sts get-caller-identity
```

What is all this snapshot and AMI business? What did Packer just do?

*overview of how packer works*

### Cleaning up after yourself

From the above calls we can see that packer creates and AMI and a snapshot. Storing snapshots isn't free. It's certainly not expensive but why pay for something that we don't need to.

Let' try deleting the snapshot, you'll need to look up your snapshot ID by using the `descibe-snapshots` command used earlier:

``` bash
aws ec2 delete-snapshot --snapshot-id "snap-03c43bc1e4504e1ec"
```

Hmm that didn't work. This message appeared:

```
An error occurred (InvalidSnapshot.InUse) when calling the DeleteSnapshot operation: The snapshot snap-03c43bc1e4504e1ec is currently in use by ami-0f8beed61ccbdb89d
```

The snapshot can't be deleted because there is a registered AMI that's using the snapshot. This snapshot is the blueprint being used for the root volume of an instance that gets created when using the AMI we created. If we delete the snapshot, the AMI won't have the required information to create a new instance for us.

There are two types of storage an instance can use for it's root volume:

- Instance backed.
- EBS backed.

The instance we created was EBS backed. If you take a look at the Packer json file we ran there is a hint that suggests this:

```json
"type": "amazon-ebs"
```

If you'd like to know more about what the difference between instance backed and EBS backed EC2 instances check out the AWS doco: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/RootDeviceStorage.html

Ok so we have a snapshot we can't delete because we have registered and EBS backed AMI that need it. Maybe deregistering the AMI first, then deleting the snapshot will work? Give it a shot. The AWS cli documentation will tell you how to deregister an AMI. If you are stuck check out the help section.

If everything ran to plan there should now be no snapshots returned when running `describe-snapshots`:

```
 aws ec2 describe-snapshots --filters Name=owner-id,Values=348201094690
```

We could have created a new EC2 instance from the AMI that was created, but it was no different to the standard Amazon Linux 2 AMI offered. Next up we will do something that's more useful.

### Packer provisioners

Similar to Ansible modules, Packer has provisioners. Two of those provisioners are used for running Ansible playbooks. We just wrote and Ansible playbook to install Jenkins are part of project 2. Maybe we can reuse that playbook here..

### Packer Ansible provisioner

Let's try and get Packer to run an Ansible playbook.

There are some slight modifications required to the Packer json:

```json
{
    "builders": [{
      "type": "amazon-ebs",
      "region": "ap-southeast-2",
      "source_ami_filter": {
        "filters": {
        "virtualization-type": "hvm",
        "name": "amzn2-ami-hvm*",
        "root-device-type": "ebs"
        },
        "owners": ["137112412989"],
        "most_recent": true
      },
      "instance_type": "t2.micro",
      "ssh_username": "ec2-user",
      "ami_name": "packer-ansible-test {{timestamp}}"
    }],
    "provisioners": [
      {
        "type": "ansible",
        "playbook_file": "./test-playbook.yml"
      }
    ]
  }
```

And lets create a test playbook file based on the early testing we did in the last project:

```yaml
---

- hosts: jenkins-boxes
  gather_facts: true
  become: yes
  become_method: sudo

  tasks:
    - name: Just some debug messaging
      debug:
         msg: "I am connecting to {{ ansible_nodename }} which is running {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

Check Packer documentation on what the host name is called when running Ansible.


With that working, use the whole Ansible file from the last project that installed and started Jenkins on the box.

### test the deployment

Should now have an AMI with Jenkins. Go to the console and start up a new EC2 instance using the AMI. Add the necessary security groups to connect to Jenkins console and then try navigating to it.

To find the AMI, have a look in the console under EC2 -> AMIs

Now can start up a box with Jenkins with no extra effort.

### introduce cloudformation

Now have a means to totally codify the creation of an instance with Jenkins (or any other software we need to deploy). However starting the instance, applying a security group.

Cloudformation gives us a mechanism to the codifiation of infrastructure.

Here's a quick template that simply starts an EC2 instance:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "My EC2 test"
Resources: 
  JenkinsEC2Instance: 
    Type: AWS::EC2::Instance
    Properties: 
      ImageId: "ami-00e17d1165b9dd3ec"
      InstanceType: "t2.micro"
```

The formatting of the file should look familiar, it's YAML, which is the same format we used for the Ansible playbooks.

The `Resources` section of the file lists all of the AWS resource to create. In this case, we only have one, an EC2 instance. There are a lot of properties that we can set for an EC2 instance through cloudformation. For this example, we have only specified which AMI to use and the type of instance. Important information such as security groups,  and access keys are missing which means the instance won't be very useful for us just yet.

## running cloudformation

Cloudformation can be run through the AWS console or from the command line.

Seeing as we've been getting used to the command line, let's do it that way.

First create a file named `my-cloudformation.yml` with the short sample shown above.

Then try running your template like this:

```bash
aws cloudformation create-stack --stack-name MyFirstStack --template-body file://./first-cloudformation.yml
```

### Checking resources

Go back into the AWS console and go to the CloudFormation service.

You should a new stack there called `MyFirstStack`. Awesome it worked!!

Take a look at your EC2 instances, there should be an instance there that was created through the CloudFormation template.

### Deleting resources

Deleting a CloudFormation stack is even easier than creating one.

```
aws cloudformation delete-stack --stack-name MyFirstStack
```

Have a look at your CloudFormation and EC2 in the AWS console again.


### cloudformation to deploy the jenkins AMI

Now build on this cloudformation template until you can deploy the Jenkins AMI you baked in the first part of this project.

Here are a few things to keep in mind:

- EC2
  - Specify a key pair so you can access the instance using SSH.
  - You need to change the image ID so it refers to the Jenkins image you created.
  - The instance will need a security group to allow access on the usual ports for SSH and the Jenkins console.
- Security group
  - You'll need another resource, a security group.
  - The EC2 instance needs to use this security group.


Build up the template a little bit at a time.

You'll notice that if you try to `create-stack` when the stack already exists you get an error.

Take a look at how `update-stack` works.

The final template is down in the help section if you need it.

### Checking it all out

When you have the template successfully working you should be able to access the Jenkins console on port 8080 as well as SSH onto the instance.

## Infrastructure as code

Well here we are, we now have our Jenkins instance fully codified. The only manual steps in our process now are:

- Running packer to build a new AMI.
- Running a cloudformation template to deploy the AMI and security group configuration.

Now, if we were running this Jenkins box in production and had an issue with it, we wouldn't have to bother trying to fix it, we could just delete the stack and recreate it. Winning!

To do something like terminate and restart an EC2 we wouldn't want to delete the whole stack though. We'd also be destroying the security group and another other resources that were part of the same CloudFormation stack.

Also what happens if the Jenkins instance gets sick and dies? AWS isn't magic, we are still running our EC2 in a data center. Data centers have hardware failures now and then. How do we protect our Jenkins from destruction? How can we terminate our Jenkins and get another healthy one to appear if needed?

### Autoscaling groups

Amazon provides autoscaling groups to help us with this problem.

With an autoscaling group, we can specify how many instances we want running at one time. If the number of instances drops below that, then the autoscaling group will create another one. It's also possible to have more instances created depending on certain instance metrics, such as CPU usage. If the CPU usage of the current instance gets too high, we might need another one to handle the load.

In this case, we only really want one instance of our Jenkins AMI running. An autoscaling group with the following properties will help us achieve this goal:

```yaml
DesiredCapacity: "1"
MinSize: "1"
MaxSize: "1"
```

With this configuration, the autoscaling group will always try to keep exactly one instance running. If the instance in the group is terminated, a new one will be created automatically.

You can probably guess what those properties do, even so it might be a good idea to check out the AWS documentation on autoscaling groups.

There are three types of resources we need in the cloudformation template for this exercise:

```yaml
AWS::EC2::SecurityGroup
AWS::EC2::LaunchTemplate
AWS::AutoScaling::AutoScalingGroup
```

You'll notice EC2 is nowhere to be found. The `LaunchTemplate` takes the place of the EC2 resource. When the autoscaling group starts a new instance, it needs to know what that instance should look like. The LaunchTemplate provides this information.

Venture forth into the AWS documentation, and see if you can get a cloudformation file build that creates:

- A security group with access on:
  - port 8080 and port 22
- An autoscaling group:
  - With a min, max and desired size of 1.
- A launch template:
  - That uses the Jenkins AMI baked earlier in this project.

There are a few tricky steps in here. The template likely won't run successfully the first time. Log into AWS console and check the CloudFormation error messages to help troubleshoot.

If you get really stuck, take a look in the help section for the complete template.

When you've got the template working, log into the AWS console and take a look at your running EC2 instances. Also check out your launch templates and autoscaling groups.

You should see there's now one autoscaling group that contains one instance. Try SSHing on to the instance.

Everything should be fine and dandy.

Let's shake things up a bit. Terminate the instance you have running, either via the command line or though the AWS console. Then go and take a look at the autoscaling group again. You should still see the instance there, but now it has a status of unhealthy.

If you wait a little while, you'll notice another instance pops up. The autoscaling group replaces the terminated instance with a new instance. It uses the LaunchTemplate we created as a blueprint for the new instance.

Once the new instance has started up, try SSHing onto it. You'll notice the new instance has a new IP address. This is not all that helpful to us. If we get a new IP address every time a new instance is created, things like DNS will stop working until they are updated. This might be fine for our little Jenkins project, but if we were running a production system, people would be running after us with torches.

Also remember the whole autoscaling thing? We are just using a group size of one at the moment, so there is only one EC2 instance and one IP address to send any internet traffic to. What if we had two instances in our autoscaling group? How would we split traffic evenly between them?

There must be a way to:

- Keep the IP address the same, even though the instances are terminating and restarting.
- Distribute traffic across more than one instance evenly.

Of course there is! Load balancers are the answer.

However they are a story for another day!

I hope you have enjoyed working through project 3.



## HELP

### Deregistering an AMI

```
aws ec2 deregister-image --image-id "ami-0f8beed61ccbdb89d"
```

### Deploying the EC2

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "My Jenkins EC2 test"
Resources: 
  JenkinsEC2Instance: 
    Type: AWS::EC2::Instance
    Properties: 
      ImageId: "ami-091848aec0ee8756a"
      InstanceType: "t2.micro"
      KeyName: "MyEC2Key"
      SecurityGroups:
        - !Ref JenkinsSecurityGroup
  JenkinsSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: '8080'
          ToPort: '8080'
          CidrIp: 0.0.0.0/0
```

### Autoscaling EC2 template

Make sure you change the `ImageId` to match the ID of your Jenkins AMI.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "My Jenkins EC2 test"
Resources: 
  JenkinsSecurityGroup:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupDescription: "Enable SSH access via port 22"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: "22"
          ToPort: "22"
          CidrIp: "0.0.0.0/0"
        - IpProtocol: "tcp"
          FromPort: "8080"
          ToPort: "8080"
          CidrIp: "0.0.0.0/0"
  JenkinsLaunchTemplate:
    Type: "AWS::EC2::LaunchTemplate"
    Properties:
      LaunchTemplateName: JenkinsTemplate
      LaunchTemplateData:
        ImageId: "ami-091848aec0ee8756a"
        InstanceType: "t2.micro"
        KeyName: "MyEC2KeyPair"
        SecurityGroups:
          - !Ref JenkinsSecurityGroup
  JenkinsAutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref JenkinsLaunchTemplate
        Version: 1
      DesiredCapacity: "1"
      MinSize: "1"
      MaxSize: "1"
      AvailabilityZones:
        - "ap-southeast-2a"
        - "ap-southeast-2b"

```