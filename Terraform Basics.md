
There's a handful of Terraform basic commands that you will use anytime that you use Terraform including the Terraform ivPAD

- terraform init (does not touch the state file at all)
	- initialize and download what's needed locally to get to work
- terraform validate
	- validates if everything is consistent within the code and valid
- terraform plan
	- plan what needs doing to go from current state to proposed code given
- terraform apply
	- executes the configuration file
- terraform destroy
	- takes everything down that Terraform built (only what Terraform is managing not the manually deployed resources)


1. Find out what version of Terraform you have by running
````sh
terraform -version
````

You can also find out about other Terraform commands and what they do by running
````sh
terraform -help
````

You should see an output that looks like this

![[Terraform Commands.png]]

2. Initialize your workspace, then head to whatever folder you would like and create a new file named main.tf. Enter the following code into that file

````sh
resource "random_string" "random" {
  length = 16
}
````

Once saved you can go ahead and run Terraform init. You will notice that the terraform.tfstate and .terraform.lock.hcl files are automatically created in that same folder

3. Validate the configuration by typincg 
   ````sh 
   terraform validate
   ````

This will go ahead and confirm that the code that you have entered is a valid Terraform code. 

4. Generate a Terraform plan by running 
````sh
terraform plan -out myplan
````

This will show you the changes that need to be made in order for the code to be executed. For this specific code, expect to see something like:

Terraform will perform the following actions:

  # random_string.random will be created
  + resource "random_string" "random" {
      + id          = (known after apply)
      + length      = 16
      + lower       = true
      + min_lower   = 0
      + min_numeric = 0
      + min_special = 0
      + min_upper   = 0
      + number      = true
      + result      = (known after apply)
      + special     = true
      + upper       = true
    }

Plan: 1 to add, 0 to change, 0 to destroy.

You must also know that by typing the -out flag and inserting myplan, the specific plan was put into a file called myplan. This means that you can apply this SPECIFIC plan later as needed.

5. Apply the plan as mentioned above by typing 
````sh
terraform apply myplan
````
   
If you were to change the file and then type in terraform apply, you would notice that it plants to add one thing (your new action) and will destroy 1 (your old action)

6. Once done with everything, you can go ahead and destroy the resources created by running 
````sh
terraform destroy
````
