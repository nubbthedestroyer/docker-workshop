# Exercise #1: Launching a Cloud9 Environment

This is an exercise in which we will launch a Cloud9 IDE in your AWS account.

## Launch your Environment

1. Log into the Console
1. At the main screen, under "AWS services", click in the search bar and type "Cloud9".  Click on the result below
to go to the Cloud9 service portal.
1. Click on "Create Environment"
1. Give your environment a name and, optionally, a description.  Click Next
1. Accept all the default values on this screen and click "Next".
1. Review your settings and click "Create"
1. Wait for your environment to start.  In this step, AWS is provisioning an EC2 instance that 
your IDE environment will run on.  This gives us the distinct advantage of having a controlled 
environment for development regardless of client hardware and OS.  In the case of this workshop,
it also allows us to connect to our instances and AWS's API without worrying about port 
availability in a corporate office.  :-)
1. Once your IDE loads, you should see a Welcome document.  Your instructor will give you a 
walkthrough of the visible panel.  Feel free to take a moment to read through the welcome document.


## Configure your environment

1. Below the Welcome Document, you should see a terminal panel.
1. Expand the terminal panel up by clicking on the square icon at the upper right margin of the terminal panel (not the X).
1. This is a fully functioning bash terminal running inside an EC2 instance, but it is bare and doesn't have the things
we need to execute this workshop, so lets install a few packages.

```bash
sudo yum -y install jq git
sudo pip install --upgrade pip
sudo ln -s /usr/local/bin/pip /bin
sudo pip install --upgrade awscli
```

## Test your AWS Credentials

Your IDE comes with managed IAM credentials that match the permissions of the IAM user running the environment, 
but because we are going to be interacting with IAM, we need to use standard credentials due to restrictions that
AWS places on temporary credentials.  To setup the correct credentials, we need to input them into the credentials
chain on the Cloud9 IDE.  

#### Disable Managed Temporary Credentials

1. In Cloud9 IDE, click on "AWS Cloud9" in the top left.  
1. Click "Preferences"
1. Expand the "AWS Settings" section.
1. Click the slider button to disable "AWS managed temporary credentials"

#### Setup AWS CLI credentials chain

You should have received credentials from your instructor in the form of:
1. Account ID
1. User Name
1. Password
1. aws_access_key_id
1. aws_secret_access_key

We need #4 and #5 to setup the creds chain.  Run the following command in your IDE

```bash
aws configure
```

You'll be asked for the credentials.  Supply them as prompted.  For the region, enter "us-east-1"
Once that completes, run the following commands to test to ensure that you have AWS API 
access and your credentials chain is working.

```bash
aws ec2 describe-regions
```

If you get a string of JSON, then you should be good.  Your IAM user should have already been configured with the 
permissions you will need for future modules.

## Pull your resources repository

The next thing we need to do is pull this repository down so we can use it in future modules.  Run the following to 
do this:

```bash
mkdir -p workshop
cd workshop
git clone https://github.com/nubbthedestroyer/docker-workshop .

```

Having done that, you should be ready for the next module.