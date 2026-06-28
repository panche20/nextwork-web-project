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

- Open the AWS IAM console.
- Choose Create user, select Name : Yourname-IAM-Admin‍
- Provide user access to the AWS Management Console - optional, For the console password, choose Custom password.
- Choose Next
- In the permissions set up page, choose Attach policies directly, From the list of Permissions policies, select AdministratorAccess
- Choose Next & Create user, Download the .csv & login using IAM user.

### Step 2 : Launch an EC2 Instance

- Amazon Console - EC2 - Instances - Launch instances.
- Name : nextwork-devops-yourname, AMI : Amazon Linux 2023 AMI
- Instance type : t2.micro, head to the Network settings section
- Create a new key pair
- Back to our EC2 instance setup, head to the Network settings section, For Allow SSH traffic from, select the dropdown and choose My IP
- For Allow SSH traffic from, select the dropdown and choose My IP. This makes sure only you can access your EC2 instance.
- Leave the default values for the remaining sections, Choose Launch instance

### Step 3 : Connect to your EC2 Instance

- Connect your EC2 instance, using SSH key.

### Step 4 : Install Apache Maven and Amazon Corretto 8

**💡 What is Apache Maven?**

Apache Maven is a tool that helps developers build and organize Java software projects. It's also a package manager, which means it automatically download any external pieces of code your project depends on to work. We're also using Maven today because it's really useful for kick-starting web projects! It uses something called archetypes, which are like templates, to lay out the foundations for different types of projects e.g. web apps.

**Install Apache Maven using below commands:**

```
wget https://archive.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
sudo tar -xzf apache-maven-3.5.2-bin.tar.gz -C /opt
echo "export PATH=/opt/apache-maven-3.5.2/bin:$PATH" >> ~/.bashrc
source ~/.bashrc
```

**Now, we are going to install Java 8 or more specifically, Amazon Correto 8**

```
sudo dnf install -y java-1.8.0-amazon-corretto-devel
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64
export PATH=/usr/lib/jvm/java-1.8.0-amazon-corretto.x86_64/jre/bin/:$PATH
```

**💡 What is Java? What is Amazon Correto 8?**
Java is a popular programming language used to build different types of applications, from mobile apps to large enterprise systems.
Maven, which we just downloaded, is a tool that NEEDS Java to operate. So if we don't install Java, we won't be able to use Maven to generate/build our web app today. Amazon Corretto 8 is a version of Java that we're using for this project. It's free, reliable and provided by Amazon.

**Verify using :**

```
mvn -v
java -version
```

### Step 5 : Create the Application

```bash
mvn archetype:generate \
   -DgroupId=com.nextwork.app \
   -DartifactId=nextwork-web-project \
   -DarchetypeArtifactId=maven-archetype-webapp \
   -DinteractiveMode=false
```

```
-DartifactId=nextwork-web-project : names your project
-DarchetypeArtifactId=maven-archetype-webapp : specifies that you're creating a web application.
-DinteractiveMode=false : runs the command without pausing for user input, so Maven will go ahead and install everything without waiting for your confirmation.
```

## Part 2 : Connect a GitHub Repo with AWS

### Step 1 : Install Git

```bash
sudo dnf update -y
sudo dnf install git -y
```

### Step 2 : Create a GitHUB repository

- Select New repository
- Name : nextwork-web-project
- Description : Java web app set up on an EC2 instance, Select Create repository.

**Now, push code to Github**

- git init
- git remote add origin [YOUR GITHUB REPO LINK]
- git add .
- git commit -m "Updated index.jsp with new content"
- git push -u origin master
- git config --global user.name "Your Name"
- git config --global user.email you@nextwork.org
- git config --global credential.helper store

## Part 3 : Secure Packages with CodeArtifact
