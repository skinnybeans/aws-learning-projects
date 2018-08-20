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

#### Instance type

We want to use the same instance type as when launching though the console.

#### Key name

Use the same key that you generated as part of project 1. If you've forgotten the key pair name, try running this from your command line:

```bash
aws ec2 describe-key-pairs --query KeyPairs[*].KeyName
```

#### Launch is go

By now you should have crated a security group to associate with the new instance, and gathered all the information you need to launch it.

The final command will look something like this:

```bash
aws ec2 run-instances --image-id 'ami-39f8215b' --security-group-ids 'sg-21bd3b59' --count 1 --instance-type 't2.micro' --key-name "MyKeyPair"
```

#### What the...

If you managed to successfully launch the instance, a whole lot of JSON will be output to the command line. This output has all the information you need to know about the instance you just created.

See if you can locate the InstanceId of the instance you just created.

Pretty painful right?

We can get this information for all the instances in the account like this:

```
aws ec2 describe-instances
```

You can imagine if you had many instances running, reading through this information would a nightmare. There must be a better way right?

Yes, yes there is.

#### Selecting fields

Fortunately there's an easier way to find information we are interested in, queries.

AWS provide some good information on how to do this over here: https://docs.aws.amazon.com/cli/latest/userguide/controlling-output.html#controlling-output-filter


##### 1. Selecting instance ID

See if you can filter the output of the `ec2 describe-instance` command to only show the instance id.

Once that's working try these slightly harder exercises. If you are having trouble check out the help section. The AWS link above explains what _dictionary notation_ and _array notation_ are.

##### 2. Selecting instance ID and status

Display the instance id and state name using _array notation_. The output should look like this:

```bash
[
    [
        [
            "i-0ce97e3575305bd47", 
            "running"
        ]
    ]
]
```

##### 3. Selecting instance ID, status and public IP

Display the instance id, status and public IP using _dictionary notation_. The output should look something like this.

```bash
[
    [
        {
            "publicIP": "54.206.58.107", 
            "state": "running", 
            "id": "i-0ce97e3575305bd47"
        }
    ]
]
```


##### 4. Filtering records

The results we get back now are much more usable. Because we have only have one instance running, it's pretty easy to find what we need. As the number of instances grow, we are going to need another technique to help keep the command line output manageable.

The AWS documentation describes a way to limit the results to records that match a specified criteria.

Launch a second instance in your account and see if you can run a `aws ec2 describe-instances` that only returns one of the two instances in the results. The `InstanceID` is a good field to use for this exercise.

Check out the help section for guidance if you get stuck.

## Now connect!

As we did in project 1, let's SSH onto the box and check everything is up and running:

```bash
ssh -i $keyname ec2-user@$ip_address
```

If you can't remember the public IP address of the instance, try using `aws ec2 describe-instances` and an appropriate query to find it.

## do the jenkins install and run things here..

Ansible things...


## now connect to the management console
http://13.211.62.213:8080



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

## Help

### I can't find the information to launch an instance!

#### AMI ID

If you go to the EC2 launch wizard the in the AWS console, the first screen shown asks you to select an AMI.

It should show the AMI ID there, and it should be something like: ami-39f8215b

#### Security group

To create the security group

```bash
aws ec2 create-security-group --description 'jenkins group' --group-name 'jenkins-group'
```

Note down the ID of the group once it's created.

To add the SSH connectivity on port 22:

```bash
aws ec2 authorize-security-group-ingress --group-id $security_group_id --protocol "tcp" --port "22" --cidr "0.0.0.0/0"
```

#### Instance type

Sticking to the free tier so use:

```
t2.micro
```

### I can't get the query working!

Help for each of the queries to get you going.

#### 1. Instance ID

```
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId]'
```


#### 2. Instance ID and status

Using array notation:

```
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId, State.Name]'
```

#### 3. Instance ID, status and public IP

Using dictionary notation:

```
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{id:InstanceId, state:State.Name, publicIP:PublicIpAddress}'
```

Get all running instance:
```
aws ec2 describe-instances --query 'Reservations[*].Instances[?State.Name==`running`]'.[InstanceId,State.Name,Tags]
```

#### 4. Filtering to one instance

##### The conditional expression

The AWS documentation tell us that `?` can be used to specify a conditional operation.

In this case, if we are filtering on InstanceID, the conditional would look like:
```
?InstanceId===`i-4345343453454`
```

Of course put the InstanceID that you are searching for in there.

##### Where to put the conditional

The conditional expression must be put in a position where more than one element exists. If there is only one element, then there is nothing to conditionally select.

An array holds a collection of elements, so that's a good place to put our conditional.

If we take a look back at one of the query strings previously used we can see there are two arrays (the parts with `[*]`:

```
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{id:InstanceId, state:State.Name, publicIP:PublicIpAddress}'
```

As we are selecting based on the instance id, it makes sense to place the conditional in the `Instances` array like this:

```
aws ec2 describe-instances --query 'Reservations[*].Instances[?InstanceId==`i-4345343453454`].{id:InstanceId, state:State.Name, publicIP:PublicIpAddress}'
```

https://jmespath.readthedocs.io/en/latest/specification.html#filter-expressions




```
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{ID:InstanceId,Status:State.Name}'

aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name]'

aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name, NetworkInterfaces[*].Association.PublicIp]'
```