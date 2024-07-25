Create an AWS SageMaker Domain in the Console



Introduction:

Amazon SageMaker is a fully managed service that provides every developer and data scientist with the ability to build, train, and deploy machine learning (ML) models quickly. With SageMaker, you can automate the tedious tasks of training and tuning models, allowing you to focus on developing high-quality models. This guide provides a comprehensive, step-by-step walkthrough on creating an AWS SageMaker Domain, enabling all necessary services, and setting up users for SageMaker Studio. By following this guide, you'll establish a robust environment for managing and deploying machine learning projects efficiently.

Table of Contents:

1.	Introduction

2.	Step-by-Step Instructions

•	Step 1: Create an Execution Role

•	Step 2: Create a SageMaker Domain

•	Step 3: Enable All SageMaker Services

•	Step 4: Create Users for SageMaker Studio

•	Step 5: Create a Project Template in SageMaker Studio

3.	Conclusion

4.	Troubleshooting and Tips

5.	Appendices

•	Appendix A: AWS CLI Commands

•	Appendix B: Useful Links and Resources



Step by Step procedure for creation of AWS Sagemaker Domain:

Step 1: Create an Execution Role

1. **Navigate to IAM Console:**

•	Go to the IAM service in the AWS Management Console.

•	Click on **Roles** in the left sidebar.

•	Click **Create role**.



2. **Select Trusted Entity:**

•	Choose **SageMaker** as the trusted entity.

•	Click **Next: Permissions**.

3. **Attach Policies:**

•	Attach the following managed policies:

•	`AmazonSageMakerFullAccess`

•	`AmazonS3FullAccess`

•	Click **Next: Tags**, then **Next: Review**.

4. **Name the Role:**

•	Enter a role name (e.g., `SageMakerExecutionRole`).

•	Click **Create role**.



Step 2: Create a SageMaker Domain

1. **Navigate to SageMaker Console:**

•	Go to the SageMaker service in the AWS Management Console.

2. **Create Domain:**

•	In the left sidebar, click **Domains** under the **SageMaker Studio** section.

•	Click **Create domain**.

3. **Specify Settings:**

•	**Domain name:** Enter a name for your domain.

•	**Authentication method:** Choose `IAM`.

•	**Execution role:** Select the role created in Step 1.

4. **Network Configuration:**

•	Select your VPC and subnets. Ensure the subnets have internet access if you plan to use internet-enabled resources.

5. **Encryption Settings:**

•	Choose default settings or configure custom encryption.

6. **Tags :**

•	Add tags.



7. **Create Domain:**

•	Review the settings and click **Create domain**.

Step 3: Enable All SageMaker Services

1. **Navigate to Studio:**

•	Once the domain is created, go to **SageMaker Studio** in the SageMaker console.

2. **Enable Services:**

•	SageMaker Studio automatically enables many services, but ensure the following are enabled:

•	**SageMaker Projects**

•	**SageMaker Experiments**

•	**SageMaker Pipelines**

•	**SageMaker Model Registry**

**SageMaker Feature Store**

•	**SageMaker Data Wrangler**

•	You may need to create and configure these services individually based on your project requirements.



Step 4: Create Users for SageMaker Studio

1. **Navigate to Domains:**

•	In the SageMaker console, click on **Domains**.

2. **Select Domain:**

•	Select the domain you created.

3. **Create User:**

•	Click **Add user**.

•	**User profile name:** Enter a user profile name.

•	**Execution role:** Select the previously created role.

•	Optionally, configure custom settings and tags.

4. **Repeat for Additional Users:**

•	Repeat the user creation process for each additional user.



Step 5: Create a Project Template in SageMaker Studio

1. **Navigate to SageMaker Studio:**

•	Open SageMaker Studio from the SageMaker console.

2. **Create Project:**

•	In SageMaker Studio, click on **SageMaker Projects** in the left sidebar.

•	Click **Create project**.

3. **Use Pre-built Templates:**

•	Choose from pre-built templates or create a custom template.

4. **Configure Project:**

•	Follow the prompts to configure your project. This typically involves specifying source control settings, dependencies, and other configurations.

5. Troubleshooting and Tips

Provide common issues and solutions:

•	Permissions Issues: How to resolve permission-related problems.

•	Network Issues: Ensuring proper VPC and subnet configurations.

•	Service Limits: How to request increases for AWS service limits.

6. Appendices

Appendix A: AWS CLI Commands

List useful AWS CLI commands relevant to the process:

•	Commands for IAM role creation.

•	Commands for SageMaker domain creation.

Appendix B: Useful Links and Resources

Provide links to relevant AWS documentation and resources:

•	AWS IAM documentation.

•	AWS SageMaker documentation.

•	AWS CLI documentation.



Conclusion:

By following this structured approach, you can create a comprehensive document that provides clear and detailed instructions for setting up an AWS SageMaker Domain, enabling necessary services, and creating users for SageMaker Studio. This will help users to efficiently manage and deploy their machine learning projects on AWS SageMaker.



By following these steps, DevOps should be able to set up a robust environment for developing and deploying machine learning models using AWS SageMaker.

