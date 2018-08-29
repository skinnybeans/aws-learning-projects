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

Similar to Ansible modules, Packer has provisioners.





### test the deployment

### introduce cloudformation

### cloudformation to deploy the jenkins AMI

### cloudformation to add security group creation to AMI


## HELP

### Deregistering an AMI

```
aws ec2 deregister-image --image-id "ami-0f8beed61ccbdb89d"
```