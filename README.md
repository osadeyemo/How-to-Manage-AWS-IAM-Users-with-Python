# How-to-Manage-AWS-IAM-Users-with-Python
(https://www.ipswitch.com/blog/how-to-manage-aws-iam-users-with-python)


Amazon Web Services (AWS) offers a service known as Identity and Access Management (IAM) that lets AWS Administrators provision and manage users and permissions in AWS cloud.  AWS IAM can prove very useful for System Administrators looking to centrally manage users, permissions and credentials; in order to authorize user access on AWS services like EC2, S3, CloudWatch etc.

AWS SDK for Python, also known as the Boto3 library, makes user management very simple by letting developers and sysadmins write Python scripts to create and manage IAM users in AWS infrastructure. Before you can start writing Python programs to automate IAM, it is a prerequisite to configure AWS credentials in a Bash Shell environment. These credentials must have full admin rights to manage the AWS IAM service, and you can refer my previous article here to setup the environment on a local computer.

 Let’s start by fetching details of IAM users that already exist in AWS Cloud…
 
Getting User Details

To list all IAM users in your console using Python, simply import the boto3 library in Python and then use the 'list_users' method in the IAM client to get a list of all users in your Python console. This list will also provide user properties like creation date, path, unique ID, and ARN properties, as seen in the following image. 

import boto3
iam = boto3.client('iam')

for user in iam.list_users()['Users']:
 print("User: {0}\nUserID: {1}\nARN: {2}\nCreatedOn: {3}\n".format(
 user['UserName'],
 user['UserId'],
 user['Arn'],
 user['CreateDate']
 )
 )

prateek-aws-iam-python1If you're part of a big organization with more users in your IAM systems than can be queried with the default user limit, then you should use Paginators, which has the capabilities to handle response objects to large to be returned in a single request. A Paginator object splits the query response into small pages, and manages automatic calls to get the next page until the complete response is obtained. 

To add pagination interface, use the 'get_paginator()' method with the IAM client, and then iterate through each page and list of users one at a time. 

import boto3
iam = boto3.client('iam')

paginator = iam.get_paginator('list_users')
for page in paginator.paginate():
 for user in page['Users']:
 print("User: {0}\nUserID: {1}\nARN: {2}\nCreatedOn: {3}\n".format(
 user['UserName'],
 user['UserId'],
 user['Arn'],
 user['CreateDate']
 )
 )

You can also retrieve the information of a specific IAM user using the 'get_user()' method of the IAM client object, by passing the name of the user to the named property: 'UserName'. 

import boto3
iam = boto3.client('iam')
iam.get_user(UserName='prateek')
iam.get_user(UserName='<Name of User>') 

prateek-aws-iam-python2If you don't specify a user name, IAM determines the user name automatically based on the AWS credentials used to query the AWS APIs. In this case, I used my Access Key and Secret ID, which I configured in Bash Shell to Authenticate to AWS. 

import boto3
iam = boto3.client('iam')
iam.get_user()

prateek-aws-iam-python3

Now that we know how to query AWS IAM users, lets's try to create one!
Creating a User and Assigning Policies

To create a new IAM user, you must first create an IAM client, then use the 'create_user()' method of the client object by passing a user name to the name property 'UserName', as demonstrated in the following code sample. 

import boto3
iam = boto3.client('iam')

# create a user
iam.create_user( UserName='John')

# attach a policy
iam.attach_user_policy(
 UserName = 'John', 
 PolicyArn='arn:aws:iam::aws:policy/AmazonEC2FullAccess'
)

prateek-aws-iam-python4We will also assign a policy to set permissions on the new user object. In this case, I'll grant the user full access on the EC2 service, but before we do that, we need to get the Policy ARN (Amazon Resource Number), which can be obtained from the AWS IAM console by searching for the required policy, as demonstrated in the screenshot below. 

Once you have the Policy ARN and user created, simply use the 'attach_user_policy()' method of client object to pass the username and Policy ARN, which will attach the policy and permissions to the user object. 

prateek-aws-iam-python5Log on to the AWS IAM Console to verify that the permissions are now set uo for the new user by navigating to the Users>Name of User>Permissions tab. 
prateek-aws-iam-python6Listing All User Permissions

The 'get_account_authorization_detail()' method of the IAM client can be utilized to filter out all users with applicable policies. Then these filtered user objects can be iterated to populate Policy Names and ARNs agains each user's details, as in the following code sample. 

import boto3
iam = boto3.client('iam')

for user_detail in iam.get_account_authorization_details(Filter=['User'])['UserDetailList']:
 policyname = []
 policyarn = []
 # find each policy attached to the user
 for policy in user_detail['AttachedManagedPolicies']:
 policyname.append(policy['PolicyName'])
 policyarn.append(policy['PolicyArn'])
 # print user details 
 print("User: {0}\nUserID: {1}\nPolicyName: {2}\nPolicyARN: {3}\n".format(
 user_detail['UserName'],
 user_detail['UserId'],
 policyname,
 policyarn
 )
 )

prateek-aws-iam-python7Deleting a User

Boto3 library lets developers easily delete any specific user using the ‘delete_user()’ method of IAM client object, which lets you pass  the name of the user to a named parameter: UserName to delete it.

But make sure that user is not a member of any groups, and that they don't have any access keys, signing certificates, or any attached policies. If there are any such dependencies,  be sure to remove them before performing the delete operation. Otherwise, you’ll end up with errors.

import boto3
iam = boto3.client('iam')

# OPTIONAL: detach a policy
iam.detach_user_policy(
 UserName = 'John', 
 PolicyArn='arn:aws:iam::aws:policy/AmazonEC2FullAccess'
)

# attach a policy
iam.attach_user_policy(
 UserName = 'John', 
 PolicyArn='arn:aws:iam::aws:policy/AmazonEC2FullAccess'
)

prateek-iam-aws-python8To conclude everything we've learned so far, in this article, we covered how to use Python's Boto3 library to get a specific IAM user, or a complete list of IAM users through pagination interface. We also learned how to create a new user and grant user permissions through policies. We also learned how to populate user details with effective permissions and delete a user from IAM, with dependencies, when they are no long required. 
