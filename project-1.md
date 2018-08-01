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
`ssh -i MyEC2Key ec2-user@<instance-ip-address>`

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

If you run the command suggested above: `sudo yum update` your instance will get patch up. However you'll notice that it prompts you if you want to install the updates.

In our case we always want to install the updates, and there are times that prompting for user input will break automation scripts that we write.

See if you can find a way of running the updates with getting prompted to enter `y`.

Note: after you've run the updates once, the instance won't need updating again. So you'll need to crate a new instance to try it out.

Follow the same steps as before. You don't need to generate a new SSH key, just select the one you created last time.


### 4. Install Jenkins

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