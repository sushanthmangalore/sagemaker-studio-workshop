# sagemaker-studio-workshop
Custom Workshop for SageMaker Studio

- ## Create Studio Domain as Studio Admin

1. Open AWS console and switch to AWS region you would like to use. For temporary AWS accounts, use us-east-2. Make a note of your AWS account ID in your notes.
2. As the default team role, in the search bar, type CloudFormation and click on CloudFormation.
3. Create the stack from the CloudFormation template in the repo. If you are using your own AWS account, use the most privileged IAM user/role that you have access to for this operation. Make a note of all the values in the Outputs section, as these will be used later.
4. The stack creates the VPC, Subnets, IAM roles and policies. It also creates 3 users, 1 SageMaker Studio admin and 2 SageMaker Studio users. 
5. Sign out of the AWS console and login as the SageMaker Studio admin user. The Studio admin user has no permissions directly assgined, so will need to assume the SMStudioAdminRole to get the necessary permissions that a SageMaker Studio admin requires.
6. In the search bar, type SageMaker and click on Amazon SageMaker.
7. Click on Amazon SageMaker Studio (first option on the left pane).
8. Select the "Standard setup" option and choose AWS IAM as the Authentication method.
9. For the Execution Role, select SMStudioExecutionRole
10. Leave the defaults under Notebook sharing configuration and SageMaker Projects and JumpStart. 
11. Under Network and Storage, select the SMStudioWorkshopVPC and add all subnets. Select Public Internet Only for Network Access and for Security Groups, select SMStudioSecurityGroup.  
12. Hit Submit to create your Studio domain. This step can take a 3-5 minutes to complete. The screen notifies you when the Studio domain is ready to use.

- ## Create user profiles in the Studio Domain

1. Attempt to Add a user using the "Add User" button in the Studio Control Panel. Select SMStudioExecutionRole as the Execution role. You will notice an AccessDeniedException. In this workshop, we are trying to ensure that one studio user cannot access another studio user's workspace, so we need to tag each individual user so they can be matched with the corresponding IAM user.
2. In the search bar, type CloudShell and click on Cloudshell. Wait a few seconds for your terminal to be ready.
3. Paste the below command in the terminal after replacing your Studio domain Id and your AWS account id in the right places- 

  ```
  aws sagemaker create-user-profile --domain-id <domain id> --user-profile-name smstudiouser1 --tags Key=studiouserid,Value=smstudiouser1 --user-settings ExecutionRole=arn:aws:iam::<account id>:role/SMStudioExecutionRole
  
 aws sagemaker create-user-profile --domain-id <domain id> --user-profile-name smstudiouser2 --tags Key=studiouserid,Value=smstudiouser2 --user-settings ExecutionRole=arn:aws:iam::<account id>:role/SMStudioExecutionRole
  ```
  
 The 2 users that you have created now show show up in the Studio Control Panel.
   
- ## Open the Studio Workspace as a user

1. Sign out of the AWS console and login as smstudiouser1. 
2. Attempt to open the Studio as smstudiouser2. You should see an access denied exception, based on the fact that the user profile name is being validated to match the IAM user who has signed in.
3. Atttempt to open the studio as smstudiouser1. The Studio should now launch correctly for first time use. Allow upto a minute for the Studio to fully launch. 
4. Clone this Git Repostory by navigating to Git -> Clone a repository option in the top menu.


- ## Read and write from an S3 bucket from the Studio

1. Navigate to S3 console and create a new S3 bucket. Name the bucket uniquely.
2. Upload the CSV file from the Git repo into the S3 bucket.
3. Open the s3-read-write.ipynb notebook. Edit the bucket name to match your bucket name. 
4. Execute the code in the cell. Observe that the file from the S3 bucket is read correctly and an output file is being written back to the same bucket.


- ## Create a custom kernel

1. Open the notebook bring-custom-container.ipynb
2. Run the code in the cells until cell 3. At this point your container is built and can be used as a custom image. The remaining cells are optional and can be run for the sake of completeness.
3. Return to the Studio Control Panel. Select the Attach Image option. Choose New Image and provide the the URL of the ECR image just built. Provide an image name and display name and select the execution role as SMStudioExecutionRole.
4. Do not change the EFS mount path. Provide a name for the Kernel and hit Submit. The custom image should now be available to use in the Studio.


- ## Share a notebook

1. While still in the context of bring-custom-container.ipynb, click on the Share option. Select the option to include the Git repo information and copy the share link to the clipboard.
2. Sign out of the AWS Console. Login as smstudiouser2
3. Navigate to the SageMaker console and open the Studio with the smstudiouser2 profile. 
4. Paste the link for the shared notebook. This will open a read-only copy of the shared notebook. smstudiouser2 can make a copy of the shared notebook and make changes to it.
5. Clone the shared repository to commit the changes to the main line.
6. In the search bar, type EFS and click on EFS. You should see a file system. This was created as part of the Studio domain and exists in the VPC selected at the time of domain creation. It has access points in each of the subnets selected during domain creation too. 
7. You can launch an EC2 instance and mount the file system to view the contents of this file system. Within the file system, each user profile in the Studio will have their own dedicated directory.
