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

## Install Jenkins

Originally I was going to keep this section the same as project 1. The would be a bit easy though, and the whole idea of these projects is to learn new things, so lets dive into Ansible!

### What's Ansible?

Ansible provides a way of automating configuration management of servers. It can be used for managing datacenter based servers and VMs as well as cloud based services.

There are a couple of other popular tools for doing the same thing such as Puppet and Salt.

### Ansible terminology

#### Control node

Ansible doesn't have to run on the host that it's configuring. It can run on a different host, and this is the way its been traditionally used. The control node connects to hosts via SSH then runs a set of commands against it.

In this project, your local machine will act as the control node.

#### Hosts

Also called managed nodes. These are the machines that we want to configure. Ansible provides ways of classifying groups of hosts. So you could for example run a set of Ansible commands against only your database servers or web severs. This listing of servers is called an _inventory_ in Ansible speak.

In this project, the EC2 instance we want to install Jenkins on will be the only host.

#### Playbooks

Playbooks contain the commands that you want to run on the hosts. The playbook will be run on the _control node_ by use the `ansible-playbook` command. Ansible will then connect to the hosts that have been specified and execute the commands in the playbook.

### Our first Ansible

Just like we can use the `ping` command to see if a server is up and running, Ansible has a ping module that can be used to check if the host we are connecting to is available.

The syntax of this command:
```bash
ansible $hosts -m ping
```

`hosts` specifies which hosts we want to run the command on. We will build a host file to reference in here.

`-m` specifies which Ansible module to use, in this case we just want to execute the ping module.

#### The hosts file

The hosts file, or inventory file lists all the hosts we want to manage using Ansible. We will start off with just one host.

Save the following text in a file called `my-hosts.yml`. Replace the `ansible_host` IP with the public IP of your EC2 instance.

```yml
jenkins-boxes:
  hosts:
    jenkins-box-1:
      ansible_host: 54.206.58.107
      ansible_user: ec2-user
```

First up this file declares a group of hosts called `jenkins-boxes`. In this example we are only going to have one host `jenkins-box-1`.

You can probably figure out that `ansible_host` specifies the IP address of the host and `ansible_user` is the user we are going to use for the connection.

#### Ping

So lets try doing an Ansible ping to the server:

```bash
ansible -i my-hosts.yml -m ping
```

#### Oops

If you followed the instructions correctly, it should have failed:

```
ERROR! Missing target hosts
```

Hang on, didn't we just pass in an inventory file to the command that contained hosts in it? After all that's what the `-i` stands for (inventory).

Our inventory file only has one host. Imagine it had more, something like this:

```yml
jenkins-boxes:
  hosts:
    jenkins-box-1:
      ansible_host: 54.206.58.107
      ansible_user: ec2-user
    jenkins-box-2:
      ansible_host: 54.206.58.108
      ansible_user: ec2-user
database-boxes:
  hosts:
    db-box-1:
      ansible_host: 54.206.59.107
      ansible_user: ec2-user
```

Ansible needs to know which hosts we want to run the command on, it won't automatically run it against every host in the file. Think of it as a safety check.

Let's try running the command again:

```
ansible -i my-hosts.yml jenkins-box-1 -m ping
```

#### ARGH!!

One of two likely outcomes has just occurred.

##### 1. Copy paste has a bug in it

If you saw a lot of purple warning text output to the command line that looked like this:

```
 [WARNING]:  * Failed to parse /Users/skinnybeans/Documents/AWS-projects/ansible-test/my-hosts.yml with yaml plugin: Syntax Error while loading
YAML.   mapping values are not allowed in this context  The error appears to have been in '/Users/skinnybeans/Documents/AWS-projects/ansible-test
/my-hosts.yml': line 4, column 19, but may be elsewhere in the file depending on the exact syntax problem.  The offending line appears to be:
jenkins-box-1       ansible_host: 54.206.58.107                   ^ here

 [WARNING]:  * Failed to parse /Users/skinnybeans/Documents/AWS-projects/ansible-test/my-hosts.yml with ini plugin: /Users/skinnybeans/Documents
/AWS-projects/ansible-test/my-hosts.yml:4: Expected key=value host variable assignment, got: 54.206.58.107

 [WARNING]:  * Failed to parse /Users/skinnybeans/Documents/AWS-projects/ansible-test/my-hosts.yml with auto plugin: Syntax Error while loading
YAML.   mapping values are not allowed in this context  The error appears to have been in '/Users/skinnybeans/Documents/AWS-projects/ansible-test
/my-hosts.yml': line 4, column 19, but may be elsewhere in the file depending on the exact syntax problem.  The offending line appears to be:
jenkins-box-1       ansible_host: 54.206.58.107                   ^ here

 [WARNING]: Unable to parse /Users/skinnybeans/Documents/AWS-projects/ansible-test/my-hosts.yml as an inventory source

 [WARNING]: No inventory was parsed, only implicit localhost is available
```

Your hosts file isn't correctly formatted. YAML is very picky with spacing, and doesn't like tabs at all.

Try fixing up the hosts file formatting. There's some guidance offered over at https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html

##### 2. You followed instructions correctly

In this case, you'll see another error:

```
jenkins-box-1 | UNREACHABLE! => {
    "changed": false, 
    "msg": "Failed to connect to the host via ssh: ec2-user@54.206.58.107: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).\r\n", 
    "unreachable": true
}
```

From the error message we can see some familiar looking snippets: `ssh`, `ec2-user@54.206.58.107`, `Permission denied`.

Ansible is trying to connect to the host via ssh as the user `ec2-user`. However there is something missing here... When you manually log into the instance using ssh there was another piece of information that you provided, otherwise the `permission denied` exception occurred.

See if you can work out what that was, and get the Ansible command working correctly. If you are stumped, check out the help section.

#### Ping pong

Now it should all work! The command line output:

```
jenkins-box-1 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

We now have an EC2 instance that's ready for configuration by Ansible.

#### Installing the Jenkins things

I briefly touched on `playbooks` when introducing Ansible terminology. A playbook provides a mechanism of grouping commands together. The kind of commands that could install Jenkins on an EC2 instance.

Playbooks are executed slightly differently to the `ping` command we used before:

```bash
ansible-playbook -i $inventory_file $playbook
```

Let's try running a simple playbook, save the following into a file called `my-playbook.yml`

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

Now let's check the syntax of the playbook is correct:

```bash
ansible-playbook -i my-hosts.yml my-playbook.yml --syntax-check
```

You should see:

```
playbook: my-playbook.yml
```

Which means we are good to go. Just one more thing before we run the playbook for real. Let's see which hosts the playbook will run on:

```
ansible-playbook -i my-hosts.yml my-playbook.yml --list-hosts
```

If everything has gone well so far your output should look something like:

```
playbook: my-playbook.yml

  play #1 (jenkins-boxes): jenkins-boxes	TAGS: []
    pattern: [u'jenkins-boxes']
    hosts (1):
      jenkins-box-1
```

It looks like it will run on a set of hosts matching `jenkins-boxes`, of which there is one host `jenkins-box-1`.

Finally, run the sucker:

```
ansible-playbook -i my-hosts.yml my-playbook.yml
```

Nope, not going to happen. We've got permission errors again. Must have forgot something, but I'm sure you can remember right?


#### Adding the private key to the inventory file


Specifying the private key file all the time becomes a pain. Also, what if we have lots of hosts in the inventory file and they all use different private keys? There must be a better way!

There is a better way. The private key file for a host can be specified in the inventory file. The Ansible documentation will tell you how: https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#list-of-behavioral-inventory-parameters

When you have this working correctly, you should be able to run the playbook without having to specify the private key on the command line.

### Configuring the instance

The instance is up and running and we can run an Ansible playbook against. Now's the time to get Jenkins installed using the playbook we started.

The playbook calls Ansible modules. There are a range of modules available for different tasks. For example, there's a `yum` module that executes `yum` commands. It's also possible to run commands as if they were entered on the command line of the host using the `command` module.

To get you started, update your playboook so it looks like this:

```yaml
---

- hosts: boxes
  gather_facts: true
  become: yes
  become_method: sudo

  tasks:
    - name: Just some debug messaging
      debug:
         msg: "I am connecting to {{ ansible_nodename }} which is running {{ ansible_distribution }} {{ ansible_distribution_version }}"
    - name: Updating packages
      yum:
        name: "*"
        state: "latest"
```

Note the spacing of the new command aligns with the previous one.

A new task has been added. This new task will call the `yum` module and update all packages on the host to the latest version. You'll notice there are two name properties under the newly added task. This shows why correct indenting must be used. The `name: Updating packages` relates to the name of the task. The second occurrence of name, `name: "*"` relates to the `yum` module. It tells the module to execute on all currently installed packages.

Running the newly modified playbook should result in the packages on the instance being updated. You won't all of the update output that you saw when logged into the instance. You'll be presented with this unassuming line:

```
TASK [Updating packages] *************
changed: [jenkins-box-1]
```

Try running the playbook again and see what happens.

### Installing Jenkins

The steps we need to take are the same as for project 1. Just as a reminder, here are the `name` of each task I used in my playbook for getting Jenkins up and running:

```yaml
    - name: Jenkins YUM config

    - name: Jenkins public key

    - name: install jenkins

    - name: install openjdk jre

    - name: start Jenkins if it isn't running
```

Using this as a guide, fill in the gaps! You can use the `command` module for all of them. Look up the Ansible help on how to use `command`.


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


### I can't get ansible ping working

The missing piece of the puzzle here is a private key file.

When manually using ssh to connect, we did something like this:
```
ssh -i MyKeyFile.pem ec2-user@123.34.111.204
```

Ansible also uses ssh to connect, so the EC2 instance will also require it has the correct credentials for ssh access.

The Ansible documentation shows the syntax needed for specifying a private key file:
https://ansible-tips-and-tricks.readthedocs.io/en/latest/ansible/commands/#running-ansible-as-a-different-user

So the final command we want to run looks like this:

```bash
ansible -i my-hosts.yml jenkins-box-1 -m ping --private-key ~/MyKeyFile.pem
```

```
aws ec2 describe-instances --query 'Reservations[*].Instances[*].{ID:InstanceId,Status:State.Name}'

aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name]'

aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name, NetworkInterfaces[*].Association.PublicIp]'
```

### I can't get my playbook to run!

Fairly straight forward, the ssh private key file needs to be included in the command. The full command should look like:

```
ansible-playbook -i my-hosts.yml my-playbook.yml --private-key ~/MyKeyFile.pem
```

### Adding the private key to the inventory file

The property we are looking for here is `ansible_ssh_private_key_file`

So the inventory file should look like this:

```
jenkins-boxes:
  hosts:
    jenkins-box-1:
      ansible_host: 54.206.58.107
      ansible_user: ec2-user
      ansible_ssh_private_key_file: ~/Documents/ssh/MyEC2KeyPair.pem
```