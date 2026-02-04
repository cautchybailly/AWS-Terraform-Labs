
Steps for setting up and configuring Terraform on your device

#### PREREQUISITES
1. Download and install a code editor. For our purposes, Visual Studio Code works well.
	1. Head to https://code.visualstudio.com/download 
2. Select the download file needed based on your operating system
3. Once downloaded, follow the steps in the installer

#### TERRAFORM INSTALL
1. Head over to https://developer.hashicorp.com/terraform/install and select the install file appropriate for your operating system. Download it
2. Run the downloaded installer file and follow the steps making sure to install on path
3. Once installed, open up your CLI of choice and confirm that it has been installed correctly. For us that looks like opening up Git Bash and entering:

````sh
terraform version
````

#### MAKE SURE AWS IS CONFIGURED IN CLI
1. Head over to Git Bash, type in:

````sh
aws configure
````

2. If all is configured, you should see an AWS Access Key ID, hit enter. You will then see the AWS Secret Access Key; hit enter. You will then see Default region name; hit enter. And finally you will see your default output format
3. If this information has not been configured already:
	1. Head over to AWS console,  then head to IAM > Users > select your user > generate an access key and secret key from a user with administrative privileges. 

In Git Bash type in the following:
````sh
aws configure --profile YOUR_PREFERRED_PROFILE_NAME
````

When prompted, enter your AWS Access Key ID, Secret Access Key, default region, and default output format. 

NOTE: if using PowerShell, you need to do the following
````sh
PS C:\> $Env:AWS_ACCESS_KEY_ID="<YOUR ACCESS KEY>"
PS C:\> $Env:AWS_SECRET_ACCESS_KEY="<YOUR SECRET KEY>"
````


Terraform show will show you the details of everything that has been deployed
Terraform state list will show you the basic form of resources that are being managed by Terraform