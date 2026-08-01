# AWS
## Setup IAM

### Create New aws account

Setup a AWS account with MFA. It will be root user

### Create a admin user

This will be used to do all work, we will leave the root user.

Create a group with administrator policy and assign to admin user.
create a password and possibly mfa also.

### Setup a power user

This will be used to do all work from CLI.
Create a user and assign Power user Policy, or assign to via new group.

This user will have all permissions, but no console access and no user management.
create access key and export to csv.

## Setup CLI
Use below command to install
 - irm https://awscli.amazonaws.com/v2/install.ps1 | iex
 - aws update

now configure the access by 
 - aws configure import --csv file://credentials.csv
or use below commands (Example):
 - aws configure
 - AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
 - AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
 - Default region name [None]: us-west-2
 - Default output format [None]: json

