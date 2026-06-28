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

Today, we're working with AWS CodeArtifact to secure your web app's packages.

<img width="1700" height="522" alt="image" src="https://github.com/user-attachments/assets/15f2eed1-23c5-4da0-94f8-04d8cad36c0f" />

**💡 Why am I learning about AWS CodeArtifact?**
When building apps, you don't create everything from scratch. Instead, you often use pre-made "packages" (chunks of code) that other developers have already created. It's just like making pasta with pre-made pasta sauce, instead of making everything from scratch.

CodeArtifact is an **artifact repository**, which means you use it to store all of your app's packages in one place. It's an important part of a CI/CD pipeline because it makes sure an engineering team is always using the same, verified versions of packages when building and deploying the app, which reduces errors and security risks!

### Step 1 : Set Up AWS CodeArtifact Repository

Now, let's set up AWS CodeArtifact, a fully managed artifact repository service. We'll use it to store and manage our project's dependencies, ensuring secure and reliable access to Java packages.
This is important because CodeArtifact provides a centralized, secure, and scalable way to manage dependencies for our Java projects, improving build consistency and security.

**💡 What is AWS CodeArtifact?**
CodeArtifact is a secure, central place to store all your software packages. When you're building an application, you typically use dozens of external packages or libraries - things other developers have created that you don't want to build from scratch. 
An artifact repository gives you a consistent, reliable place to store and retrieve these components. This gives you three big benefits: 
1️⃣ Security: Everyone in a team retrieves packages from a secure repository (CodeArtifact), instead of downloading from unsafe sources on the internet (hello, security risks)!
2️⃣ Reliability: If public package websites go down, you have backups in your CodeArtifact repository.
3️⃣ Control: Your team can easily share and use the same versions of packages, instead of everyone working with a different version of the same package.

<img width="1075" height="360" alt="image" src="https://github.com/user-attachments/assets/5a48a050-b8af-454a-addf-2c5c42a84c26" />

**Create and configure your CodeArtifact repository**

- Head back to AWS Management Console - Type CodeArtifact
- Repositories - Create repository, Repository name : nextwork-devops-cicd
- Description : This repository stores packages related to a Java web app created as a part of NextWork's CI/CD Pipeline series.
- Under Public upstream repositories - optional, select the checkbox next to maven-central-store.
- This will configure Maven Central as an upstream repository for your CodeArtifact repository.
- Click Next
- Under Domain selection, choose This AWS account.
- Domain Name : nextwork, Click Next
- Select the Create repository button to create your CodeArtifact repository.

**💡 What are upstream repositories?**
Upstream repositories are like **backup libraries** that your primary repository can access when it doesn't have what you need. If you didn't set up CodeArtifact or have an upstream repository, your build would fail because a package is missing!

When your application looks for a package that isn't in your CodeArtifact repository, CodeArtifact will check its upstream repositories (like Maven Central in our case) to find it.

Once found, Maven will then store a copy in your CodeArtifact repository for future use. This gives you three major benefits that you'll appreciate as your projects grow:
**Speed** - After the first download from Maven Central, retrieving packages directly from CodeArtifact will speed up how quickly your app starts up and runs.
**Reliability** - If Maven Central goes down (which happens more often than you'd think!), your builds keep working because you've got local copies
**Control** - You can audit which external packages are being used in your organization and even block problematic ones if needed.

**💡 What is Maven Central?**
Maven Central is essentially the App Store of the Java world - it's the most popular public repository where developers publish and share Java libraries. When you're building Java applications, chances are you'll need packages from Maven Central. It contains virtually every popular open-source Java library out there, from database connectors to testing frameworks and UI components.

By connecting our CodeArtifact repository to Maven Central, we're setting up a system where we get the best of both worlds: access to all these public libraries, but with the added benefits of caching, control, and consistency that come with our private CodeArtifact repository.

**💡 What is a CodeArtifact domain?**
A CodeArtifact domain is like a folder that holds multiple repositories belonging to the same project or organization. We like using domains because they give you a single place to manage permissions and security settings that apply to all repositories inside it. This is much more convenient than setting up permissions for each repository separately, especially in large companies where many teams need access to different repositories.

With domains, you can ensure consistent security controls across all your package repositories in an efficient way.

### Step 2 : Create an IAM Policy for CodeArtifact Access

For Maven to start working with CodeArtifact, we need to create an IAM role that grants our EC2 instance the permission it needs to access CodeArtifact.

Otherwise, Maven can try all it wants to command your EC2 instance to store and retrieve packages from CodeArtifact, but your EC2 instance simple wouldn't be able to do anything! And going another layer deeper, IAM roles are made of policies; so we need to create policies first before setting up the role.

<img width="1530" height="592" alt="image" src="https://github.com/user-attachments/assets/1407364f-9994-4cdb-838c-c890d417e6a2" />

**Create a new IAM policy**
- IAM console - Policies
- Click Create Policy button
- On the Create policy page, select the JSON tab.
- Replace the default content in the text editor with the following JSON policy document. Copy and paste the entire JSON code block:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "codeartifact:GetAuthorizationToken",
                "codeartifact:GetRepositoryEndpoint",
                "codeartifact:ReadFromRepository"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "sts:GetServiceBearerToken",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "sts:AWSServiceName": "codeartifact.amazonaws.com"
                }
            }
        }
    ]
}
```
- After pasting the JSON policy document, click the Next button at the bottom right.
- On the Review policy page, in the Policy name field, enter *codeartifact-nextwork-consumer-policy.*
- In the Description - optional field, add a description like:
  *Provides permissions to read from CodeArtifact. Created as a part of NextWork CICD Pipeline series.*
- Click the Create policy button to create the IAM policy.

### Step 3 : Attach IAM Policy and Verify CodeArtifact Connection

Now that we've created the IAM policy for CodeArtifact access, let's attach it to an IAM role and then associate that role with our EC2 instance. This will grant our EC2 instance the permissions it needs to securely access CodeArtifact. Finally, we'll verify the connection to CodeArtifact from our EC2 instance.

This is important because attaching the IAM role to our EC2 instance is what actually grants the instance the permissions defined in the policy, enabling secure access to CodeArtifact.

<img width="1730" height="677" alt="image" src="https://github.com/user-attachments/assets/c05eb821-84a4-4744-a984-7c5629dcf247" />

**Create a new IAM role for EC2**

- IAM Console - Roles
- Create Role, Select entity type : AWS Service
- Choose a use case : EC2, Click NEXT
- In the Add permissions step, in the Filter policies search box, type *codeartifact-nextwork-consumer-policy.*
- Select the checkbox next to the *codeartifact-nextwork-consumer-policy* that you created in the previous step.
- Click **Next** to head to the *Name, review, and create* step.
- Role Name : *EC2-instance-nextwork-cicd*
- Description : *Allows EC2 instances to access services related to the NextWork CI/CD pipeline series.*
- Create Role

### Step 4 : Attach IAM role to EC2

- Head back to EC2 - Instances
- Select EC2 instance : nextwork-devops-yourname
- Click on Actions - Security - Modify IAM Role
- Under IAM Role : Select EC2-instance-nextwork-cicd
- Select Update IAM Role

### Step 5 : Get CodeArtifact connection instructions

- Head back to CodeArtifact - Repositories - nextwork-devops-cicd
- On repository's page, click the *View connection instructions* button at the top right corner.
- In the Connection instructions page, we're configuring how Maven will connect to your CodeArtifact repository.
- For Operating system, select Mac and Linux.
- For Package manager client, select mvn (Maven).
- 🚨 Double check that you're using Mac and Linux as the operating system. Even if you're doing this project on a Windows computer, Mac and Linux is the right       choice - your EC2 instance is an Amazon Linux 2023 instance!
- Make sure that Configuration method is set to Pull from your repository.
- Nice! The menu will now show you the steps and commands needed to connect Maven to your CodeArtifact repository.

**Export CodeArtifact authorization token**
- In the Connection instructions dialog, find Step 3: Export a CodeArtifact authorization token.
- Copy the entire command in Step 3.
- Paste the copied command into the terminal and press Enter to run it.


**💡 What is this step for?**
😳 "Export a CodeArtifact authorization token for authorization to your repository from your preferred shell" sounds a little technical!

It actually just means you need to run the command in Step 3 to give your terminal a temporary password. That password will grant your development tools (i.e. Maven) access to your repositories in CodeArtifact.

Maven uses this token whenever it needs to fetch something from your CodeArtifact repository.

**💡 Recap: What is a CodeArtifact authorization token?**
The authorization token is like a temporary ID badge for your build tools to access CodeArtifact. When you run that token command, AWS checks your identity and issues this digital badge that's valid for 12 hours. Your build tools (like Maven) then present this token whenever they need to grab something from your CodeArtifact repository.

Why use temporary tokens instead of permanent credentials? It's all about security! If a token is ever compromised, it automatically expires in hours instead of giving permanent access. Plus, you don't have to worry about storing sensitive credentials in your build configuration files. The system automatically handles the authentication process behind the scenes, making your life easier while keeping your repositories secure. Just remember that you'll need to refresh this token if you come back to the project after more than 12 hours!

### Step 6 : See Packages in CodeArtifact!

Let's make sure everything is set up correctly by verifying the connection to our CodeArtifact repository from our EC2 instance. We'll configure Maven to use CodeArtifact and then try to compile our web app, which should now download dependencies from CodeArtifact.

This verification is crucial to ensure that our EC2 instance can successfully access and retrieve packages from CodeArtifact, which is a key part of our CI/CD pipeline setup.

<img width="1701" height="532" alt="image" src="https://github.com/user-attachments/assets/8f723d28-66db-4b8d-a243-31eeb522f563" />

**In this step, you're going to:**
- Finish setting up the connection between Maven and CodeArtifact.
- Compile your Maven project using the settings.xml file.
- See your CodeArtifact repository automatically store your project's dependencies!
- You might notice that we still have a few steps left in CodeArtifact's connection settings panel!

```
💡 What are the code in Steps 4, 5, and 6 saying?
The code snippets define repository URLs, authentication details, and other settings so that Maven knows how to connect with CodeArtifact to fetch and store your project's dependencies.

Let's break each section down:

The servers section is where your store your access details to the repositories you're connecting with your web app project. In this example, you've added your authentication token to access your local CodeArtifact repository.

The profiles section is where you write a rulebook on when Maven should use which repository. We only have one package repository in this project, so our profiles section is more straightforward than other projects that might be pulling from multiple repositories! Our profiles section is telling Maven to go to the nextwork-packages repository to find the tools / packages needed to build your Java web app.

The mirrors section sets up backup locations that Maven can check if it can't find what it needs in the first local repository it goes to. The backup location that we'll set by default is... our CodeArtifact repository again. This means that for any repository requests (denoted by the asterisk * in the * line), Maven will redirect those requests to the same CodeArtifact repository since it's our only local repository. It might seem unnecessary now, but mirrors are great in complex scenarios and is a great fallback option to set up from the start!
```

**💡 What is settings.xml?**
**settings.xml** is like a settings page for Maven - it stores all the settings we saw in Steps 4-6 of the connection window. It tells Maven how to behave across all your projects. In our case, we need a settings.xml file to tell Maven where to find the dependencies and how to get access to the right repositories (e.g. the ones in CodeArtifact).

**💡 Extra for Experts: What's xml?**
xml is a markup language that lets you structure data and write instructions for a server. It's just like how html is a markup language that lets you structure data and write instructions for a web browser to display a web page.

You might also notice **pom.xml**, which is a file that was automatically created in your repository's root directory when you set up your web app for the first time.

pom.xml tells Maven the ingredients list (i.e. dependencies) for your web app and how to put them together to build the app. Then, once Maven knows what dependencies to look for, **settings.xml** tells Maven where to find the dependencies and how to get access to the right repositories (e.g. the ones in CodeArtifact).

- Create settings.xml file

```
<settings>
</settings>
```

- Go back to the CodeArtifact connection settings panel.
- From the Connection instructions dialog, copy the XML code snippet from Step 4: Add your server to the list of servers in your settings.xml.
- Paste the code in the settings.xml file, in between the <settings> tags.
- Let's copy the XML code snippet from Step 5: Add a profile containing your repository to your settings.xml.
- Paste the code snippet you copied right underneath the <servers> tags. Make sure the <profiles> tags are also nested inside the <settings> tags.
- Finally, paste the XML code snippet from Step 6: (Optional) Set a mirror in your settings.xml... right underneath the <profiles> tags.
- Save the settings.xml file.

**💡 Recap: What is settings.xml?**
The settings.xml file is Maven's control center - it's where you tell Maven how to behave across all your projects.

When we add CodeArtifact information to this file, we're essentially telling Maven: "Hey Maven, whenever you need to download a dependency, look in this CodeArtifact repository first, and here's how to authenticate yourself."

By configuring settings.xml properly, we're creating a seamless connection between Maven and CodeArtifact. Your builds will automatically authenticate and pull dependencies from the right place without you having to think about it again. It's one of those "set it up once, benefit forever" kinds of configurations that make a developer's life much easier.

**Compile your project and verify the CodeArtifact integration**

- Go to root directory of *nextwork-web-project*
- Run command : mvn -s settings.xml compile
- Press Enter to execute the command.
- As Maven compiles your project, observe the terminal output.
- You should see messages like Downloading from nextwork-devops-cicd telling us that Maven is downloading dependencies from your CodeArtifact repository. This is    a good sign that Maven is using CodeArtifact to manage dependencies!
- If the compilation is successful and dependencies are downloaded from CodeArtifact, you'll see a BUILD SUCCESS message at the end of the Maven output.

<img width="1067" height="382" alt="image" src="https://github.com/user-attachments/assets/b3be4e91-da61-4561-a3ff-1afad36779f8" />


**💡 Recap: What happens when Maven compiles with CodeArtifact?**
When you run mvn -s settings.xml compile, Maven first looks at your project's dependencies in the pom.xml file. Then, instead of downloading them directly from public repositories, it checks your CodeArtifact repository. If the dependency isn't already in CodeArtifact, it will fetch it from the upstream repository (Maven Central in our case), cache it in CodeArtifact, and then deliver it to your project. This process happens for each required dependency, ensuring that your build process is secure, controlled, and faster for subsequent builds when dependencies are already cached in CodeArtifact.

- See it to believe it! Let's head back to the CodeArtifact console in your browser.
- Close the connection instructions window.
- If you don't see any packages in your repository listed yet, click the refresh button in the top right corner of the Packages pane.
- After refreshing, you should now see a list of Maven packages in your CodeArtifact repository.
- These are the dependencies that Maven downloaded from Maven Central via CodeArtifact when you compiled your project.

**💡 Why are packages showing up in CodeArtifact?**
Those packages appearing in your CodeArtifact repository are proof that the entire system is working! Here's what happened behind the scenes: when you ran the Maven compile command, Maven checked your project's pom.xml file and determined which dependencies your application needs. It then requested these dependencies through CodeArtifact.

Since this was the first time these dependencies were requested, CodeArtifact didn't have them yet. So it reached out to Maven Central (the upstream repository we configured), downloaded the packages, stored copies in your repository, and then provided them to Maven. It's like ordering groceries online - the store delivers what you need and keeps a record of your order.

Now that these packages are stored in your CodeArtifact repository, anyone else in your organization who needs the same dependencies will get them directly from your repository instead of from Maven Central. This gives you faster builds, more reliability, and the ability to control exactly which package versions your organization uses. Pretty powerful stuff!
