# Project 1

## Clicky Jenkins

Project one to get you introduced to the basics of AWS!

For this project, you will be using the AWS console to create a virtual machine, then log onto that VM to install the jenkins automation server.

## Prerequisites

You'll need to have access to an AWS account.

Everything in this project uses AWS free tier resources so go sign up for a new account now!

## Setting up your account

Once you've created your account, if you navigate to the IAM section of the console you'll see a checklist that looks something like this:

__[image link here]__

Complete the checklist.

Create a user and add them to the _admin_ user group.

You'll also need to generate an access key and secret id.

## Getting down to business

This document will outline in broad terms what the goals of the mini project will. There will be information left out on exactly how to get some things working.

However! All the information required can be found quite easily doing google searches.

If you get really stuck, there is a walkthrough right down the bottom of the page. I encourage you to use this as a last resort. Troubleshooting problems and finding answers to them is one of the most important skills to have when dealing with tech problems and AWS is no different.

## Goal
Being able to log into a newly installed Jenkins instance and provide the unlock code.

These steps will be done though the AWS console and by SSHing onto the virtual machine that will run Jenkins.

## Steps

This is a high level outline of what needs to happen, if you are after an extra challenge see how far you can get with just these vague directions.

1. Create EC2 instance.

1. SSH onto the instance.

1. Update instance packages.

1. Install Jenkins.

1. Start the Jenkins service.

1. Unlock the Jenkins console.

1. Celebrate!

If you managed to get Jenkins up and running with just those instructions, you probably didn't need to go through this project anyway!

## Steps - with more

Here's a little more information about what's happening at each step.

### 1. Create an EC2 instance

EC2 what AWS call their cloud virtual machines. There are a massive range of different types avaliable for any sort of computing load you can think of. You pay by the hour for what you use, but if you have a new account you can use t2.micro instances free for a certain amount of time each month.

For this project we want to use the following settings when launching the EC2 instance:
1. AMI: Amazon Linux 2
1. Instance type: t2.micro
1. The rest can be left as defaults for now.

After you click launch a dialog box will show up asking you to select a key pair or create a new one.

Create a new one and call it something like MyEC2Key.

This key pair is used for authentication when logging onto the instance. Make sure you download it!

Now your instance should be up and running. Go back to the EC2 page and have a look under running instances.

### 2. SSH onto the instance

Now the instance is up and running it's time to try and connect to it!

Go to the directory where you downloaded the SSH key.

Then try this command:
```bash
ssh -i MyEC2Key ec2-user@<instance-ip-address>
```

You'll need to go and find what the instance ip address is.

You might get an error that says something about an unprotected key pair. See if you can find out how to fix it...


### 3. Update instance packages

If managed to successfully log onto the instance you should see something like this:

```
       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
8 package(s) needed for security, out of 79 available
Run "sudo yum update" to apply all updates.
```

You'll notice it's kindly asking us to update the packages on the instance. Keeping instances patched is a good habit to get into. When instances are publicly accessible through the internet we really want to make sure there are no security exploits that a hacker can take advantage of.

If you run the command suggested above: `sudo yum update` your instance will get patched up. However you'll notice that it prompts you if you want to install the updates.

In our case we always want to install the updates, and there are times that prompting for user input will break automation scripts that we write.

See if you can find a way of running the updates with getting prompted to enter `y`.

Note: after you've run the updates once, the instance won't need updating again. So you'll need to crate a new instance to try it out.

Follow the same steps as before. You don't need to generate a new SSH key, just select the one you created last time.

### 4. Install Jenkins

Ok so we've spun up an instance and SSHed onto it. Now for some Jenkins installing action!

#### 4.1 Where to find the package

Linux software typically gets distributed as packages.

There's a tool called YUM that is used to manage the downloading, installation and updating of those packages. There are other tools for other flavours of Linux, but we will just focus on YUM for now as that the manager that Amazon Linux 2 uses.

#### 4.2 Configuring the YUM repo

YUM needs to know a few things about the packages you want to install. The main ones being the name of the package and which YUM repository to get it from. This information get stored in `.repo` files in the following directory:
```
/etc/yum.repos.d/
```

Go to that directory now and take a look at the contents using:
```
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

Now list the contens of the `/etc/yum.repos.d` directory again. You'll see something new has popped up:

```
-rw-r--r--  1 root root   71 Nov 29  2016 jenkins.repo
```

Sweet! Now try installing Jenkins again using YUM install. You should get prompted to download and install the package! Success!!


Hahaha nah just kidding. Linux has a sick sense of humor, it was just lulling you into a false sense of security.

Another error has popped up:

```
Public key for jenkins-2.135-1.1.noarch.rpm is not installed
```

There are two ways to solve this error. One is obvious, install the public key. The other is not quite so obvious. See if you can find out what that is. The answer is just below, try to solve it before peeking!

Packages can be signed by the package author, using their private key. This is done to validate the package is actually legitamate, and hasn't been compormised by a third party. To check the package is legit, you need to get the package authors public key.

#### 4.3.1 Installing the public key
To install the public key for the Jenkins package:
```
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
```

The other option you have is:

#### 4.3.2 Disabling key checking
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

#### 4.3 Installing the package


#### 4.4 Oops!




Firstly to install jenkins, we need to get the Jenkins package from somewhere.

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat/jenkins.repo
```

After the package has been downloaded to the instance, lets install it:

```bash
sudo yum install jenkins
```



### 5. Start the Jenkins service

### 6. Unlock the Jenkins console


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