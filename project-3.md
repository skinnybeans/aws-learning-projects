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
- take snapshop
- create image
- create instance from image
- try updating image again
- notice that the state has been saved.

- want to use this for jenkins so anytime we want to create a new jenkins can just be done by recreating the instance from the AMI

### Basic packer script to build ami with yum updates

### Add installing and configuring jenkins

### test the deployment

### introduce cloudformation

### cloudformation to deploy the jenkins AMI

### cloudformation to add security group creation to AMI