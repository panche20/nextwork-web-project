# Implement CI CD pipeline on AWS

## Part 1 : Set Up a Web App in the Cloud

In this project, we will build from scratch a CI/CD pipeline that automates the build & deployment of a web app.

**What's CI/CD?** 
A CI/CD (Continuous Integration and Continuous Deployment/Delivery) pipeline is a tool that automates the steps from development (i.e. writing changes to code) to deployment (i.e. making the code changes available to users). 
This helps engineering teams build and release software much, much faster! 
That's why most modern tech companies use CI/CD pipelines to ship software to users, and why it's a crucial skill for DevOps Engineers.

<img width="1647" height="522" alt="image" src="https://github.com/user-attachments/assets/7d54ab9a-0cbe-4ffe-80e1-60e32d40aae6" />

**Please Note : All the resources we are going to create should be in 1 specific region.
So, select a region according to your choice.**

### Step 1 : Set up an IAM user

- Head to your AWS Account as the root user.
- Open the AWS IAM console.
- From the left hand navigation panel, choose Users.
- Choose Create user.
- For the User name, use Yourname-IAM-Admin‍
- Make sure to select the checkbox next to Provide user access to the AWS Management Console - optional.‍
- For the console password, choose Custom password.
- Type in a password that you will be able to remember/access in the future.
- Deselect the checkbox for Users must create a new password at next sign-in - Recommended.
- Choose Next.
- In the permissions set up page, choose Attach policies directly.
- From the list of Permissions policies, select AdministratorAccess.
- Choose Next.
- Choose Create user.
- Voilà - you've just created your new user! Stay on this page.
- Choose Download .csv file.
- Copy the Console sign-in URL.
- Log out of your root user's AWS Account.
- Paste and go to your copied console sign-in URL.
- Open your downloaded .csv file containing your user's access instructions.
- Log in using your IAM user's username and password in the .csv file.

### Step 2 : Launch an EC2 Instance

- 
