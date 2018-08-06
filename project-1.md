# Project 1

## Clicky Jenkins

Project one to get you introduced to the basics of AWS!

For this project, you will be using the AWS console to create a virtual machine, then log onto that VM to install the Jenkins automation server.

## Prerequisites

The commands in this document assume you are using a Linux based terminal.

You'll need to have access to an AWS account.

Everything in this project uses AWS free tier resources so go sign up for a new account now!

I'd recommend using your own account rather than a shared account, just so you get the behavior expected though out this project.

## Setting up your account

Once you've created your account, if you navigate to the IAM section of the console you'll see a checklist of tasks to complete.

Complete the checklist.

Create a user and add them to the _admin_ user group.

You'll also need to generate an access key and secret id.

## Getting down to business

This document will outline the steps you need to take to complete the project. At some points, things will not work. This has been done deliberately so you can see what kind of errors are produced and what actions can solve them.

## Goal

The ultimate goal of this project:

```
Log onto the management console of a newly set up Jenkins automation server.
```

Let's get cracking.

## View from above

Here is a high level outline of what needs to happen, if you are after an extra challenge see how far you can get with just these vague directions.

1. Create EC2 instance.

1. SSH onto the instance.

1. Update instance packages.

1. Install Jenkins.

1. Start the Jenkins service.

1. Unlock the Jenkins console.

1. Celebrate!

If you managed to get Jenkins up and running with just those instructions, you probably didn't need to go through this project anyway!

## Steps - with more

Here's the detailed rundown of the steps. Remember, things will not work the first time! Fixing broken things is why we all got into IT in the first place right?!

### 1. Create an EC2 instance

#### What's an EC2?

EC2 what AWS call their cloud virtual machines. There are a massive range of different types available for any sort of computing load you can think of. You pay by the hour for what you use, but if you have a new account you can use t2.micro instances free for a certain amount of time each month.

#### Choosing your flavor

For this project we want to use the following settings when launching the EC2 instance:

| attribute | value |
| --- | --- |
| AMI | Amazon Linux 2 |
| Type | t2.micro |
| The rest | defaults |

#### Creating a key pair

After you click launch a dialog box will show up asking you to select a key pair or create a new one.

Create a new one and call it something like MyEC2Key.

This key pair is used for authentication when logging onto the instance. Make sure you download it!

#### Success!

The instance should be up and running. Go back to the EC2 page and have a look under running instances.

### 2. SSH onto the instance

Now the instance is up and running it's time to try and connect to it!

#### Getting connected

Go to the directory where you downloaded the SSH key.

Then try this command:
```bash
ssh -i MyEC2Key ec2-user@<instance-ip-address>
```

You'll need to go and find what the instance ip address is. Check the help section at the end of this document if you are having trouble.

You might get an error that says something about an unprotected key pair. See if you can find out how to fix it. Again refer to the help section if you are stumped.

#### Success!

If managed to successfully log onto the instance you should see something like this:

```
       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
8 package(s) needed for security, out of 79 available
Run "sudo yum update" to apply all updates.
```

### 3. Update instance packages

#### Patch that sucker

You'll notice as part of the login message, AWS politely asking us to update the packages on the instance. Keeping instances patched is a good habit to get into.

If you run the command suggested above: `sudo yum update` your instance will get patched up. However you'll notice that it prompts you if you want to install the updates.

#### Do, don't ask

There are times that prompting for user input will break automation scripts that we write. If we have a script setup to run at 3:00am we don't want it to stop and ask for input.

See if you can find a way of running the updates with getting prompted to enter `y`.

Note: after you've run the updates once, the instance won't need updating again. So you'll need to crate a new instance to try it out.

Follow the same steps as before. You don't need to generate a new SSH key, select the one you created last time.

#### Success!

You have now patched up the instance and are ready to start installing Jenkins on it.

### 4. Install Jenkins

We've spun up an instance and SSHed onto it. Now for some Jenkins installing action!

#### Where to find the package

Linux software typically gets distributed as packages.

There's a tool called YUM that is used to manage the downloading, installation and updating of those packages. There are other tools for other flavors of Linux, but we will just focus on YUM for now as that the manager that Amazon Linux 2 uses.

#### Configuring the YUM repo

YUM needs to know a few things about the packages you want to install. The main ones being the name of the package and which YUM repository to get it from. This information get stored in `.repo` files in the following directory:
```
/etc/yum.repos.d/
```

Go to that directory now and take a look at the contents like this:
```
cd /etc/yum.repos.d
ls -la
```

You should see something like this:
```
drwxr-xr-x  2 root root   54 Jun 22 22:46 .
drwxr-xr-x 79 root root 8192 Aug  2 21:51 ..
-rw-r--r--  1 root root  982 Jun 22 21:50 amzn2-core.repo
-rw-r--r--  1 root root  763 Jun 22 22:46 amzn2-extras.repo
```

Try installing Jenkins using:
```
yum install jenkins
```

Oops that didn't work. How about:
```
sudo yum install jenkins
```

Hmm different error but still no good...

YUM doesn't know anything about a package called Jenkins. Perhaps we can help it out:

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat/jenkins.repo
```

Now list the contents of the `/etc/yum.repos.d` directory again. You'll see something new has popped up:

```
-rw-r--r--  1 root root   71 Nov 29  2016 jenkins.repo
```

Sweet! Now try installing Jenkins again using YUM install. You should get prompted to download and install the package!

#### Not so fast

Linux has a sick sense of humor, it was just lulling you into a false sense of security.

Another error has popped up:

```
Public key for jenkins-2.135-1.1.noarch.rpm is not installed
```

There are two ways to solve this error. One is obvious, install the public key (the error message tells us it's missing). 

The other is not quite so obvious. See if you can find out what that is. The answer is just below, try to solve it before peeking!

#### Package signing

Packages can be signed by the package author, using their private key. This is done to validate the package is actually legitimate, and hasn't been compromised by a third party. To check the package is legit, you need to get the package authors public key.

The alternative is to disable key checking.

##### Installing the public key

To install the public key for the Jenkins package:
```
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
```

#### Disabling key checking
The other option is disabling key checking. This option is riskier, especially when you are downloading packages from public repositories. A field in the `.repo` file indicates whether key checking should occur or not.

To check the contents of the Jenkins .repo file try:
```
more jenkins.repo
```

or

```
cat jenkins.repo
```

or

```
less jenkins.repo
```
press   `q` to exit less!

The three commands do more or less the same thing (see what I did there??).

#### Installing again

Try again...

```
sudo yum install jenkins
```

#### Success!

The Jenkins package should now finally be installed!

We are almost there... There's now an EC2 instance running with Jenkins installed on it.

### 5. Starting Jenkins

Now that Jankins has been installed, it't time to boot it up.

#### Starting services

Jenkins runs as a background service. Services in Amazon Linux 2 can be managed using a program called `systemctl`.

Starting the Jenkins service using using `systemctl` can be done in the following way:

```
sudo systemctl start jenkins.service
```

#### Success?

Not quite hey.

The error message gives you directions on how to find out what went wrong.

Have a read and see if you can fix the problem. If you have trouble, check out the help section at the end of the document.

Now those errors are fixed up try starting the Jenkins service again.


#### Success!

You can check that the service is running like this:

```
systemctl status jenkins.service
```

Jenkins is now running. Let's get onto that management console.

### 6. Accessing the Jenkins console

#### Getting in

The Jenkins documentation tells us the console can be accessed on port 8080.

So we should be able to get to the console by doing the following in a web browser:

```
http://<instance-public-ip>:8080
```

The public IP is the same IP you have been using for SSH. Try accessing the console though a browser now.

#### What's the hold up?

You should notice the request times out. So what's the deal?

#### Debugging our connection

Let's do some investigation.. We've been connecting to the instance via SSH. SSH works on port 22. Try poking the instance on port 22:

```
curl <instance-public-ip>:22
```

We get error code 56 back pretty quickly after executing the command.

Now try on port 8080:

```
curl <instance-public-ip>:8080
```

It hangs for a bit, then gives us error 7.

So what's the deal with error 56 and error 7? How does this in anyway help us? We might need to know a little more about what these error messages mean. A quick google shows:

```
CURLE_RECV_ERROR (56)
Failure with receiving network data. 

CURLE_COULDNT_CONNECT (7)
Failed to connect() to host or proxy. 
```

In the first case on port 22 we connected but didn't get any data back from the server.

In the second case, on port 8080 we couldn't even connect.

Something must be blocking the connection to EC2 instance on port 8080.

#### Unblock me!

There are two mechanisms that AWS uses for controlling network access to EC2 instances: security groups and NACLs. In this case the culprit is a security group.

You need to add a new security group rule to your instance that allows traffic on port 8080.

Head on into the AWS management console and give it a shot!

#### We are in

After adding the security group rule try using the curl command on port 8080 again.

If you added the rule correctly, there should be a HTML response this time!

Now try using a browser to navigate to the instance again on port 8080.

```
http://<instance-public-ip>:8080
```

The Jenkins management console should finally be greeting you!

#### The final step

Jenkins wants some proof that you are actually the administrator of this Jenkins instance. To make Jenkins happy, you better do what he says.

SSH back onto the instance and grab the key from the file mentioned and enter it.

#### Success!

Well that's it!

## 7. Celebrate!!

Phew, that was an epic adventure! Congrats for getting though it all. You've covered a lot of ground and should have a much better understanding on the basics of running software service on an EC2 instance.

One last thing, you might want to go into the AWS management console and terminate any instances you created as part of this little project.

For the next project, we will do a similar thing, but not using the AWS console, it will be command line all the way!


## Help!!

### I couldn't find my instance IP address!

1. In the AWS console, go to the EC2 section.
1. Click on running instances.
    - There should be 1 running instance. If not try creating one again.
1. Click on the instance.
    - at the bottom of the screen a you should see a big table of information about the instance.
1. Locate the `IPv4 Public IP` field and copy the address.
   - the address will look something like this: `54.153.155.115`

### I couldn't fix my unprotected key!

When you are in the directory containing the key, type the following:
`chmod 600 MyEC2Key.pem`

### I don't know why Jenkins failed to start!

The key line in the output is this:
```
systemctl status jenkins.service
```

If you run that command and do a little more treasure hunting you'll see that something was missing:

```
Aug 04 23:17:29 ip-172-31-9-177.ap-southeast-2.compute.internal jenkins[19184]: Starting Jenkins bash: /usr/bin/java: No such file or directory
```

Taking a look at the Jenkins website, we find that Jenkins requires Java 8 to run:

https://jenkins.io/doc/administration/requirements/java/

So go and install Java 8!

### I can't figure out this Java thing!

Java can be confusing with all the take of JDK, JRE, OpenJDK, OpenJRE and Oracle stuff thrown into the mix.

Here's a quick summary:
JDK = Java Development Kit = Need for building java apps.
JRE = Java Runtime Environment = Need for running java apps.

Java is an interpreted language, not a compiled one. So to run a java application you need the Java runtime environment.

Oracle is the gatekeeper of Java now, but you can also get an open sourced version called OpenJDK. To install the OpenJDK JRE do the following:

```
sudo yum install java-1.8.0-openjdk
```