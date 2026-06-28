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

## Part 4 : Continuous Integration with CodeBuild

<img width="1067" height="347" alt="image" src="https://github.com/user-attachments/assets/8acb10c4-aa30-4d97-a8b6-c5590ec9acef" />

**Get ready to:**
- 🛠️ Create and configure a CodeBuild project from scratch.
- 🔗 Connect your CodeBuild project to your GitHub repository.
- ⚙️ Define your build process using a buildspec.yml file.
- 💎 Automate testing using CodeBuild too!

**💡 Why am I learning about AWS CodeBuild?**
When you're developing software, you need to regularly "build" your code, which is the process of turning the code into a package that can be deployed.

AWS CodeBuild is a *continuous integration service* that automates this entire build process for you. When a developer pushes new code, CodeBuild automatically compiles the code, runs the tests, and packages everything up. This saves enormous time, eliminates human error, and helps you deliver better software faster! That's why CodeBuild is an essential part of a CI/CD pipeline - it automatically makes sure your application is always built consistently and correctly.

### Step 1 : Create a CodeBuild Project

- Log in to your AWS Management Console, search CodeBuild
- Select CodeBuild from the dropdown menu under Services.
- In the CodeBuild dashboard, find the left navigation menu; Select Build projects.

**💡 What is AWS CodeBuild?**
*AWS CodeBuild* is a fully build tool for your code. It takes your source code, compiles it, runs tests, and packages it up. Engineers love continuous integration tools like CodeBuild because you don't have to manually set up and manage any build servers yourself, and you only pay for the compute time you use for building your projects (instead of entire servers that are idle most of the time). Think of it as a super-efficient, scalable, and managed service that handles all the heavy lifting of building and testing your applications.

*Continuous Integration* is like having a quality control checkpoint that automatically kicks in whenever anyone on your team makes changes to your code. Instead of waiting until the end of a project to discover that something broke, CI helps you catch and fix issues early and often. CI helps you constantly check that everything still works as expected - running tests, compiling code, and making sure new changes play nicely with the existing codebase.

**💡 What is a CodeBuild project?**
A CodeBuild project is basically the blueprint for your CI process. It's where you tell AWS everything it needs to know about how to build your application. This includes things like where your code lives (like GitHub), what kind of environment you need (Linux or Windows? Java or Python?), exactly what commands to run during the build, and where to store the results when it's done. Think of it as a recipe that CodeBuild follows every time it needs to build your application.

**Configure Your Build Project**

- On the Create build project page, scroll to the Project configuration section.
- Under Project name, enter *nextwork-devops-cicd*
- Under Project type, make sure Default project is selected.
- Scroll down to the Source section.
- Under Source provider, select GitHub.


**💡 What are the CodeBuild project types?**
CodeBuild gives you two main types of projects, each designed for different CI/CD needs:

*Default project*: This is your standard option that most teams use. It's perfect when you want to manage your entire build process within AWS. You get full control over how your build runs, what goes in, and what comes out - all without leaving the AWS ecosystem.

*Runner project*: This option is for teams who already have CI systems like GitHub Actions or GitLab CI but want to tap into the power of CodeBuild's build environment. It's like having CodeBuild do the heavy lifting while your existing CI system orchestrates the overall process.

For this project, we're using a Default project to directly manage our CI process within CodeBuild.

**💡 What does source mean?**
CodeBuild doesn't know where your web app's code lives! Source means the location of the code that CodeBuild will fetch, compile, and package into a file you can deploy.

We're choosing GitHub as our source provider because that's where we've stored our web app's code. By connecting CodeBuild directly to GitHub, we're creating a seamless pipeline where code changes automatically flow into our build process.

CodeBuild plays well with lots of different code repositories - AWS CodeCommit, Bitbucket, GitHub, and even plain S3 buckets. This flexibility lets you keep using whichever code storage system your team prefers while still getting all the benefits of AWS's build capabilities.

## Connect CodeBuild to your GitHub Repository

To allow CodeBuild to access your private GitHub repository, we need to establish a connection using AWS CodeConnections.

**In this step, you are going to:**
- Set up a connection between your AWS account and GitHub using AWS CodeConnections.
- In the Source section, under Credential, you might see the message You have not connected to GitHub. Manage account credentials.
- Click on Manage account credentials.

<img width="682" height="392" alt="image" src="https://github.com/user-attachments/assets/b51f6049-a0ad-4f49-b808-128c20f49cf9" />

- You will be taken to the Manage default source credential page.
- Ensure GitHub App is selected for Credential type.
- Select create a new GitHub connection.
- On the Create connection page, under Connection details, enter *nextwork-devops-cicd* as the Connection name.
- Click Connect to GitHub.
- You will be taken to GitHub to authorize the AWS Connector for GitHub application.
- Select your GitHub user account where your repository is located.
- Select **Select**.
- After authorization on GitHub, you'll get taken back to the AWS console
- Under GitHub Apps, you'll see that your GitHub username is an option now!.
- Select your GitHub username.
- Click Connect.

*Awesome! You've successfully connected to GitHub.* It might seem like a few steps, but this process securely links your AWS account to your GitHub account using the AWS Connector for GitHub.

- You should get redirected back to CodeBuild's Manage default source credential page after successful connection.
- On the Manage default source credential page, you should see your newly created connection listed.
- Click Save at the bottom of the page.
- Now, back in the Create build project page, in the Source section, you should see a success message in green: "Your account is successfully connected by using     an AWS managed GitHub App."

Well done! So... which service do you think connected 🤝 your AWS environment to GitHub? 🤝
When we were taken to different pages to connect to GitHub, that was CodeBuild passing us to another Code service (called AWS CodeConnections) behind the scenes!

**💡 What is AWS CodeConnections?**
AWS CodeConnections is like a secure bridge between AWS and your external code repositories. Instead of dealing with the headache of managing API keys, tokens (like GitHub's Personal Access Tokens!), or SSH credentials, CodeConnections handles all that authentication complexity for you - so you can focus on building your application.

If you'd like, you can open the left hand navigation menu, expand Settings at the bottom of the list, and open the Connections page. You can manage all connections you set up with CodeConnections there!

- You can now select your GitHub repository nextwork-devops-webapp as the source.

**Finish Setting Up Your CodeBuild Project**

Next, we need to define the environment where our builds will run. This includes the operating system, runtime, and compute resources.

*In this step, you are going to:*

- Configure CodeBuild's environment settings.
- Configure Amazon S3 to store build artifacts.
- Enable CloudWatch logs for monitoring build processes.

Ready to configure your build environment? Let's dive in!

- Scroll down to the Primary source webhook events section.
- Untick the Webhook checkbox that says "Rebuild every time a code change is pushed to this repository."* Scroll down to the Environment section.
- Under Compute, for Provisioning model, choose On-demand.
- For Environment image, choose Managed image.
- For Compute type, choose EC2.
- Under Environment, for Operating system, select Amazon Linux.
- For Runtime(s), select Standard.
- For Image, choose aws/codebuild/amazonlinux-x86_64-standard:corretto8.
- Keep Image version as Always use the latest image for this runtime version.
- Under Service role, select New service role.

**💡 Why did we pick on-demand?**
Provisioning model determines how AWS will set up and manage everything needed for your build. Choosing On-demand means AWS will create the resources you need for your build only when you start it, and tear them down when the build is done. This is cost-effective and efficient!

Reserved capacity gives you dedicated build resources always at your disposal. It costs more overall but gives you consistent performance and no wait times. Great for teams that are building constantly throughout the day. If on-demand is like ordering a taxi, reserved capacity is like renting your own car that you can access anytime you need it.

**💡 Why did we pick managed image?**
Environment image is like a template for your build environment (just like how AMI's are templates for your EC2 instances). In more technical terms, environment images are pre-configured versions of the build environment so you won't need to install all the software/tools/settings required to build a project. We choose Managed image here, which means we're using a template that AWS has already created for us. The next few settings we pick underneath this will then tell CodeBuild what kind of image we're looking for.

Custom image lets you bring your own Docker image with exactly the tools and configurations your project needs. It's like designing your own custom workspace from scratch - more work to set up, but perfectly tailored to your specific requirements.

Now, let's define how CodeBuild will actually build your application using a buildspec.yml file.

- Scroll down to the *Buildspec section.*
- Under Buildspec format, select Use a buildspec file.
- Leave Buildspec name as default buildspec.yml.
- Scroll down to the Batch configuration section.


**💡 What is buildspec.yml?**
The buildspec.yml file is like a detailed instruction manual for CodeBuild. Placed in the root of your repository, it tells CodeBuild exactly what to do at each stage of the build process - what tools to install, what commands to run, and what files to package up when it's done.

CodeBuild automatically looks for a file named buildspec.yml in the root directory of your source code. If it finds one, it uses it to execute the build. If not, the build will fail (as we'll see later) 👀

**💡 What is Batch configuration?**
Batch configuration lets you run multiple builds at once as part of a single batch job. It's like being able to cook several dishes simultaneously instead of one after another.

This becomes super useful when you need to test your code across different environments (like various operating systems or browser configurations) or when you want to parallelize parts of your build process to save time. While we won't use it in this project, it's a powerful feature for more complex workflows.

**Configure Build Artifacts**

- Scroll down to the Artifacts section. We need to configure where CodeBuild will store the build artifacts.

**💡 What are build artifacts?**
Build artifacts are the tangible outputs of your build process. They're what you'll actually deploy to your servers or distribute to users. That's why storing them properly in S3 is so important - they're the whole reason we're running the build in the first place.

For our project, we want our build process to create one build artifact that packages up everything a server could need to host our web app in one neat bundle. This bundle is called a WAR file (which stands for Web Application Archive, or Web Application Resource) and it works just like a zip file - a server will simply "unzip" your WAR file to find a bunch of files and resources (which are also build artifacts, i.e. a WAR file is a build artifact that bundles up other build artifacts) and host your web app straight away. Notice how you haven't been able to view your web app on a web browser so far in this project series - that's because we haven't created and deployed the WAR file yet!

Note: our build process will create a .war file (a packaged Java web application) as the build artifact, but artifacts could be executables, libraries, documentation, or any output your build creates.

*For Type, select Amazon S3.*

**💡 Why store artifacts in Amazon S3?**
Your compiled applications, libraries, or any output files from your build need a safe, accessible home after the build finishes. S3 is perfect for this - it's a highly reliable and scalable storage solution that's also in our AWS environment (which makes the artifact easily accessible for deployment later).



- Let's head to the S3 console. In the AWS Management Console search bar, type S3 and select S3 from the dropdown menu under Services.
- Make sure you're still in the same region where you set up the CodeBuild build project.

**💡 Extra for Experts: Why create S3 bucket in the same AWS region?**
Creating your S3 bucket in the same region as your CodeBuild project might seem like a small detail, but it matters for three key reasons:

**Speed**: When data doesn't have to travel across geographic regions, everything happens faster. Your build artifacts upload more quickly, and any services that need those artifacts can access them without delay.

**Cost**: AWS charges for data transfer between regions, but transfers within the same region are either free or much cheaper. Keeping everything in one region helps your AWS bill stay lower.

**Compliance**: Some industries have strict rules about where data can be stored geographically. Keeping your resources in a specific region helps you meet these requirements if they apply to your organization.

- On the Buckets page, click Create bucket.
- In the Create bucket page, under General configuration, for Bucket name, enter nextwork-devops-cicd-yourname.
- Leave all other settings as default.
- Click Create bucket at the bottom of the page.
- If you just created the S3 bucket, it might not show as an option in the CodeBuild project configuration.
- Refresh the build project page in your browser - you might need to configure your build project again. Challenge yourself to re-do the setup with less guidance!
- In your CodeBuild project, head back to the Artifacts section.
- For Type, select Amazon S3.
- For Bucket name, choose your newly created bucket nextwork-devops-cicd from the dropdown.
- In Name, enter nextwork-devops-cicd-artifact. This names our artifact, so it's easy to spot it in the S3 bucket.
- For Artifacts packaging, select Zip.

**💡 Why package artifacts as a Zip file?**
Packaging artifacts as a Zip file is a small detail that's actually quite useful!

*Smaller size*: Zip compression can significantly reduce the size of your artifacts, which means faster uploads to S3, less storage costs, and quicker downloads when you need to use them.

*Organization*: Instead of managing multiple individual files, you get one tidy package. It's like putting all your vacation photos in a single album rather than having them scattered across your phone.

*Simplicity*: When it comes time to deploy your application or share it with teammates, having a single zip file makes the process much more straightforward - just download one file and you have everything you need.

**Configure CloudWatch Logs**

- Scroll down to the Logs section.
- Make sure CloudWatch logs is checked.
- In Group name, enter /aws/codebuild/nextwork-devops-cicd.
- Scroll to the bottom of the page and click Create build project.

**💡 What are CloudWatch logs?**
*Amazon CloudWatch Logs* is a monitoring service that collects and tracks logs from AWS services. In this project, CloudWatch will record everything that happens during the build process, including the commands that are run, the output of those commands, and any errors that occur. This is incredibly useful for debugging and understanding what went wrong if a build fails.

### Run the Build and Troubleshoot Failures

Now that our CodeBuild project is fully configured, let's initiate our first build and see our CI pipeline in action!

**In this step, you are going to:**

- Start your first build in CodeBuild.
- Troubleshoot a build failure by adding a buildspec.yml file to your web app repository.

Let's get ready to run our first build! It's okay if it fails initially – that's part of the learning process.

- Navigate to your newly created CodeBuild project nextwork-devops-cicd.
- Click the Start build button.
- You will be redirected to the build execution page.
- You should see the build status change to In progress.
- Wait for the build to complete.
- Oooo, looks like the build failed!
- By checking the logs and phase details, we can pinpoint exactly where and why our build is failing.
- Select the Phase details tab to understand the failure.
- Aha, the DOWNLOAD_SOURCE phase has failed with the error message YAML_FILE_ERROR: YAML file does not exist.

**🙋‍♀️ Why did I see this error?**
Good news - this is exactly what we expected to happen! This error simply means CodeBuild couldn't find the *buildspec.yml* file in your GitHub repository, which makes sense because we haven't created it yet.

This is an important learning moment - without this file, CodeBuild doesn't know how to build your project. We'll create this file in the next steps.

- Click on the Build details tab.
- Scroll down to the Buildspec section.
- Let's make sure that it says Using the buildspec.yml in the source code root directory.
- This confirms that CodeBuild is looking for buildspec.yml in the root directory of our web app's nextwork-web-project code repository... we haven't created that   file yet!
- In fact, let's check our GitHub repository. Select the Repository link in your CodeBuild project.
- Welcome back to your GitHub repo!
- As you might've noticed, there's no buildspec.yml file - it should be in the root of your web app repository 🙈 Let's create it now.

**💡 Remind me: why do we need buildspec.yml?**
CodeBuild reads your buildspec.yml file like a step-by-step instruction manual. It goes through each phase in order - install, pre_build, build, then post_build - running the commands you've specified in each section.

The beauty of using buildspec.yml is that your build process is defined as code right alongside your application. This means your build process can be versioned, reviewed, and evolved just like any other part of your codebase. Before the era of continuous integration (CI), you would have to manually run a bunch of commands to build your application - buildspec.yml automates this process!

**Now, Go to your terminal**

- Go to root of the project (nextwork-web-project)
- buildspec.yml - Create this file
- Paste the following code in buildspec.yml

```
version: 0.2

phases:
  install:
    runtime-versions:
      java: corretto8
  pre_build:
    commands:
      - echo Initializing environment
      - export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain nextwork --domain-owner 123456789012 --region us-east-2 --query authorizationToken --output text`

  build:
    commands:
      - echo Build started on `date`
      - mvn -s settings.xml compile
  post_build:
    commands:
      - echo Build completed on `date`
      - mvn -s settings.xml package
artifacts:
  files:
    - target/nextwork-web-project.war
  discard-paths: no
```

**💡 What's inside buildspec.yml?**
This buildspec.yml file is like a recipe for how to build your Java web app. Let's break it down into simpler terms:

**version**: 0.2: This just tells AWS which version of the buildspec format we're using.

**phases**: Think of these as the different stages your build goes through: install is the "prep work" phase - here, we're telling CodeBuild to use Java 8. pre_build are tasks to do before the main building starts. Here, we're grabbing a security token so we can access our dependencies. build is where the actual building happens. We're using Maven (a popular Java build tool) to compile our code. post_build are the finishing touches after the main build is done. Here, we're packaging everything into a WAR file (a format for web applications).

**artifacts** tells CodeBuild which files to save as the output of the build. In our case, we want that WAR file we created during the post_build phase.

In the buildspec.yml file:

- Replace the placeholder AWS Account ID 123456789012 with your actual AWS Account ID.
- Check that the region code is correct! Update the region section from --region us-east-2 to the AWS region you're using.
- Save the buildspec.yml file

**Run the following Git commands:**

```
git add .
git commit -m "Adding buildspec.yml file"
git push
```

**💡 Why push buildspec.yml to GitHub?**
We need to push our buildspec.yml file to GitHub because CodeBuild doesn't look inside your local computer for build instructions - it looks in your GitHub repository.

When CodeBuild connects to your GitHub repo, it first looks for this special file. Without it, CodeBuild has no idea what to do with your code. By pushing the file to GitHub, we're making sure our instructions travel alongside our code.

- Go to your GitHub repository nextwork-devops-webapp in your browser and refresh the page to verify that buildspec.yml is now present in the repository.

**Verify Successful Build and Artifacts**

Now that we've fixed our CodeBuild setup, let's re-run the build process. We'll also check our S3 bucket to see if it's storing the build artifact correctly.

In this step, you are going to:
- Re-run the build process in CodeBuild.
- Troubleshoot a second build failure, this time by giving CodeBuild the permission to access CodeArtifact.
- Run and verify a successful build!

**Retry your build**

- Return to the CodeBuild console in your browser.
- Retry the build by clicking the Retry build button.
- The build status should now show In progress. Wait for this build to complete.
- Damn it, we failed again!
- Check the Phase details tab again after this retry.
- You will likely see that the DOWNLOAD_SOURCE phase succeeded this time, but other phases might have failed!

**🙋‍♀️ What does this error mean?**
Goooood question. This error typically means CodeBuild still can't access the settings.xml file.

This usually happens because our CodeBuild service role doesn't have permission to access CodeArtifact, which is needed to download project dependencies.

- To fix this, we need to grant CodeBuild's IAM role the permission to access CodeArtifact.
- Head to the IAM console.
- In the IAM console, select Roles in the left navigation menu.
- In the roles search bar, type codebuild to filter the roles.
- Select the role that starts with codebuild-nextwork-devops-cicd-service-role. This is the new service role that CodeBuild created when we set up our build
  project.
- Select your CodeBuild service role.
- Click on the Add permissions button.
- Choose Attach policies.
- In the Filter policies search bar, type codeartifact-nextwork-consumer-policy.
- Check the checkbox next to the policy named codeartifact-nextwork-consumer-policy.
- You can even expand the policy name to review the permissions granted by this policy.
- After selecting the policy, click the Add permissions button.
- You should see a green banner confirming Policy was successfully attached to role. and the policy should now be listed under the Permissions policies of your      CodeBuild service role.
- Return to the CodeBuild console, navigate to your nextwork-devops-cicd project.
- Retry the build one more time by clicking Retry build.

*The build status should again be In progress. With the added permissions, this time the build should proceed successfully! 🙏*

*NICE - the build status should now be Succeeded with a green checkmark, indicating a successful CI pipeline run!*

Let's check if our build process has successfully created and stored the artifact 🏃‍♀️

- To verify the successful build, let's check if the build artifact was correctly uploaded to our S3 bucket.
- Head to the S3 console.
- Select Buckets from the left navigation menu.
- Select the nextwork-devops-cicd bucket you created earlier.
- Initially, the bucket might be empty. Click Refresh to update the bucket content.
- Click Refresh again if needed.
- You should now see the artifact nextwork-devops-cicd-artifact.zip listed in your bucket 👀

If you choose to download and inspect the artifact, you should find the nextwork-web-project.war file inside the downloaded zip archive, confirming the build process produced the expected output.

**Automate Testing Too!**

In this secret mission, you're going to:

- Create a simple unit test for your Java web application
- Trigger a build to see your tests run automatically
- Showcase advanced continuous integration skills in your documentation!

**Create a Simple Test Script**

- touch run-tests.sh
- Add following code:

```
#!/bin/bash

echo "==== RUNNING SIMPLE TESTS ===="
echo "Test 1: Checking project structure..."
if [ -d "src" ]; then
  echo "✅ PASS: src directory exists"
else
  echo "❌ FAIL: src directory not found"
  exit 1
fi

echo "Test 2: Checking for web app files..."
if [ -f "src/main/webapp/index.jsp" ]; then
  echo "✅ PASS: index.jsp exists"
else
  echo "❌ FAIL: index.jsp not found"
  exit 1
fi

echo "Test 3: Simple validation test..."
echo "✅ PASS: This test always passes"

echo "==== ALL TESTS PASSED ===="
exit 0
```

- chmod +x run-tests.sh
- ./run-tests.sh

**Update Your Buildspec File**

- Update everything from the build stage in your file to include your test, and some markers that indicate your build process' progress:

```
  build:
    commands:
      - echo "====== BEGINNING TEST PHASE ======"
      - echo "Running tests on `date`"
      - chmod +x run-tests.sh
      - ./run-tests.sh
      - echo "====== TEST PHASE COMPLETE ======"
      - echo "====== BEGINNING BUILD PHASE ======"
      - echo "Build started on `date`"
      - mvn -s settings.xml compile
      - echo "====== BUILD PHASE COMPLETE ======"
  
  post_build:
    commands:
      - echo "Build completed on `date`"
      - mvn -s settings.xml package

artifacts:
  files:
    - target/nextwork-web-project.war
```

- Double check that you didn't completely rewrite the whole file - these edits should only start from line 11 of your buildspec.yml file!
- Save the file.

**Commit and Push Your Changes**

```
git add .
git commit -m "Add automated testing to CI pipeline"
git push
```

- Visit your GitHub repository - let's verify the updated buildspec.yml the CodeBuild should be using now.
- Cool beans! Your changes are reflected in your GitHub repo.

**Watch Your Tests in Action**

- Head back to the CodeBuild console.
- Let's start a new build that includes the testing phase.
- Select the button next to your build project.
- Select Start build.
- Select Start now.
- Off we goooooo! CodeBuild will grab the updated code in GitHub, and package up your project based on the new steps in buildspec.yml.
- Your build will show In progress to start with.
- Select the Build logs tab.

## Part 5 : Deploy a Web App with CodeDeploy

<img width="1077" height="352" alt="image" src="https://github.com/user-attachments/assets/03dd105b-b9e6-42a0-a64f-aaa5e93ac753" />

**Get ready to:**
☁️ Launch a deployment environment using AWS CloudFormation.
⚙️ Write deployment scripts to automate deployment commands.
🚀 Deploy your web app with CodeDeploy and see it live!
💎 Implement a disaster recovery technique - roll back a deployment!

**💡 Why am I learning about AWS CodeDeploy?**
When you're developing software, you need a reliable way to release new versions of your app to the world.

AWS CodeDeploy is a continuous deployment service, which means it automates how you get new software versions onto your servers. Instead of manually moving files and restarting services yourself, CodeDeploy automatically runs a deployment using settings and commands that you define.

As part of a CI/CD pipeline, CodeDeploy makes software releases faster, more consistent, and way less stressful.

**🌐 What is deployment?**

Deployment is a process that takes your code from development to ➡️ a live environment where users can access it.
Deployment usually comes with many manual steps, like copying files to servers or installing depdencies. This can be time-consuming, error-prone, and difficult to reproduce consistently.

**🤔 Then... What is AWS CodeDeploy?**
AWS CodeDeploy is a continuous deployment service. This means CodeDeploy...
Automates deployments: Eliminates error-prone manual steps - no more manually copying files and running commands to deploy your application.
Enables consistent rollouts: Your application deploys the same way every time.
Minimizes downtime: Can deploy in ways that keep your application available.
Handles failures: Can automatically roll back if something goes wrong.

<img width="1071" height="352" alt="image" src="https://github.com/user-attachments/assets/3de8409c-cd10-4bf8-ad1c-49f90575d4f8" />

### Step 1 : Launch EC2 with CloudFormation

Let's start our deployment by setting up the deployment infrastructure. We'll use CloudFormation to launch an EC2 instance and its networking resources.

**In this step, you're going to:**

- Launch an EC2 instance using CloudFormation.
- Configure network settings for the EC2 instance.
- Understand Infrastructure as Code.

- Head to the CloudFormation console.

**💡 What is AWS CloudFormation?**
Think of CloudFormation is AWS' infrastructure as code tool. Instead of clicking around the AWS console to set up resources (which gets tedious fast!), you write a single template file that describes everything you need - your EC2 instances, security groups, databases, and more. Then, CloudFormation reads this file and builds your entire environment for you, exactly the same way every time.


**💡 What is Infrastructure as Code?**
Just like software developers write code to build applications, Infrastructure as Code (IaC) is a type of software that lets you write code to create your servers, networks, and other infrastructure. Instead of manually configuring each server (time-consuming and error-prone), you have a script that sets everything up perfectly every time. Your infrastructure becomes predictable, repeatable, and much easier to manage!

**Create a CloudFormation Stack**
- Select Create stack.
- Select With new resources (standard) from the dropdown menu.

**💡 What is a CloudFormation stack?**
When you deploy a CloudFormation template, you're creating a stack - think of it as a project folder that holds all your connected resources. The cool thing is that CloudFormation treats this stack as a single unit, so you can create, update, or delete all those resources together with one command.

- Select Template is ready.
- Select Upload a template file.
- Select Choose file.
Upload the following CloudFormation template: *nextworkwebapp.yaml*

```
This is attached in this project file
```

**💡 What is a CloudFormation template?**
A CloudFormation template is a text file where you tell CloudFormation the AWS resources you want to create and how they should be configured - like specifying "I want a t2.micro EC2 instance with this security group and permissions to access this S3 bucket." The beauty is that this template is both human-readable AND machine-executable. In our project, we're using YAML because it's a bit friendlier to read than JSON (fewer brackets and quotes to keep track of!).

- Verify the file nextworkwebapp.yaml is uploaded and select Next.

**Configure Stack Details**

- Select Next.
- Enter NextWorkCodeDeployEC2Stack as the Stack name.
- Next, we'll need to add our IP address to the template.
- Head to https://checkip.amazonaws.com/ and copy your IP address.
- Paste your IP address into the MyIP parameter field and add /32 at the end.
- Select Next.

**Configure Stack Options**

- In Configure stack options, under Stack failure options, select Roll back all stack resources.
- Next, select Delete all newly created resources.

**💡 What are Stack failure options?**
Stack failure options are your safety net when things don't go as planned. They determine what CloudFormation should do if it runs into an error while creating your resources:

**Roll back all stack resources**: This is like having an "undo" button for your entire deployment. If anything fails, CloudFormation will automatically revert everything back to how it was before you started. This prevents you from ending up with a half-built environment that might not work or cost you money unnecessarily.

**Delete all newly created resources**: This makes sure CloudFormation cleans up after itself during a rollback. No resources left behind to surprise you on your bill next month!

These options are especially valuable when you're learning - they let you experiment without worrying about leaving a mess behind.

- Scroll down to Capabilities and check the box I acknowledge that AWS CloudFormation might create IAM resources.

**💡 Why would CloudFormation create an IAM role?**
IAM roles are like special visitor passes that AWS services can "wear" to temporarily access other services. In our case, our deployment EC2 instance will use a role to access files from S3.

Hmmm... why do you think it'll need access to S3 (have you created an S3 bucket anywhere)? Aha - your deployment environment will need to use the build artifacts stored in your S3 bucket!

- Select Next.

**Review and Submit**

Let's review our stack's details!

- Template: Template is ready
- Stack name: NextWorkCodeDeployEC2Stack
- Parameters: MyIP is filled with your IP and /32
- Rollback on failure: Activated
- Delete all newly created resources during rollback: Activated
- Select Submit.

**Monitor Stack Creation**

CloudFormation is now launching the stack in the background! This process will create the EC2 instance and networking resources in the background.
While we're wating...

Let's check out the Resources tab.
You can see a list of the resources being created!

**💡 What resources are being created by the CloudFormation template?**
The CloudFormation template is creating a:

**VPC (Virtual Private Cloud)**: A virtual network in the cloud for your AWS resources.
**Subnet**: A subdivision of the VPC where you can place resources.
**Route Tables**: Define how network traffic is routed within the VPC.
**Internet Gateway**: Allows resources in your VPC to connect to the internet.
**Security Group**: Acts as a virtual firewall to control inbound and outbound traffic for your EC2 instance.
**EC2 Instance**: The virtual server where your web application will be deployed.

**💡 Why are we deploying networking resources too?**
By defining these networking resources in the template, we're not just launching an EC2 instance, but creating a complete, secure, and configurable infrastructure that can be easily replicated or modified. This is an especially good idea for EC2 instances that are hosting web apps, because they have more complex needs like connecting with multiple databases and controlling both public and private network traffic.

- Next, you can also check the Events tab.
- You can watch your CloudFormation stack's events in real-time in the Events tab - just keep refreshing it!

**💡 What is a CloudFormation stack event?**
Every time CloudFormation creates, updates, or deletes something, it records an event - "Starting to create EC2 instance," "Security group created successfully," "Oh no, there isn't enough capacity to create a new VPC." These events give you visibility into exactly what's happening behind the scenes, which is super helpful for troubleshooting if something goes wrong.

- Wait for the stack's status to become CREATE_COMPLETE.

### Step 2 : Prepare Deployment Scripts and AppSpec

Now that our EC2 instance is up and running, can we start deploying our application?

Mmm, not yet. To start deploying our application, we need to prepare a set of scripts and configuration files for CodeDeploy. It's like we need to write a set of instructions for CodeDeploy to follow - otherwise, it wouldn't know how to deploy our application!

**In this step, you're going to:**

- Create deployment scripts for installing dependencies, starting, and stopping the server.
- Create an appspec.yml file to tell CodeDeploy how to handle the deployment process.
- Update buildspec.yml to package deployment files in your build artifact.

<img width="1071" height="322" alt="image" src="https://github.com/user-attachments/assets/fc579636-0421-4898-863b-0f760f7f20b9" />

**Create Scripts Folder**
- In your IDE, create a new folder at the root of your project directory.
- Name the folder scripts

**Create install_dependencies.sh**

- Inside the scripts folder, create a new file.
- Let's name the file install_dependencies.sh
- Add the following content to install_dependencies.sh:

```
#!/bin/bash
sudo yum install tomcat -y
sudo yum -y install httpd
sudo cat << EOF > /etc/httpd/conf.d/tomcat_manager.conf
<VirtualHost *:80>
  ServerAdmin root@localhost
  ServerName app.nextwork.com
  DefaultType text/html
  ProxyRequests off
  ProxyPreserveHost On
  ProxyPass / http://localhost:8080/nextwork-web-project/
  ProxyPassReverse / http://localhost:8080/nextwork-web-project/
</VirtualHost>
EOF
```

**💡 What does install_dependencies.sh do?**
The install_dependencies.sh script sets up all the software needed to run our website by installing programs (called Tomcat and Apache) that handle internet traffic and host our application. It then creates special settings that let these programs to work together, making our website accessible to visitors on the internet.


**💡 Extra for Experts**: Can we break down each line in install_dependencies.sh?
This script sets up everything our application needs to run:

**#!/bin/bash:** This is like telling the computer "Hey, run this file using bash" (it's called a shebang line).
**sudo yum install tomcat -y:** Installs Tomcat (our Java application server) - the -y means "yes to all prompts" so it won't stop and ask questions.
**sudo yum -y install httpd:** Installs Apache HTTP server - this is like the front desk receptionist for our web application.
**sudo cat << EOF > /etc/httpd/conf.d/tomcat_manager.conf:** 
This starts a special block that writes multiple lines to a configuration file at once - like dictating a whole letter instead of typing it line by line.

The lines between <VirtualHost *:80> and </VirtualHost> create a configuration file for Apache to act as a reverse proxy for Tomcat. This is like telling Apache: "When visitors come to our website, don't talk to them directly - instead, silently pass their requests to Tomcat who's running our actual application."
ProxyRequests off: Tells Apache not to act as a general proxy for any website (a security measure).
ProxyPreserveHost On: Makes sure Apache remembers who the original request was for.
ProxyPass and ProxyPassReverse: These are the magic lines that actually connect Apache to our Tomcat application.
EOF: Marks the end of our configuration block - everything between the first EOF and this one gets written to the file.


**💡 Extra for Experts: What is a reverse proxy?**
A reverse proxy is like a receptionist for your web application. When visitors (users) come to your site, they talk to the receptionist (Apache) first, who then forwards their requests to the right person (Tomcat) behind the scenes. The visitors never interact directly with the backend server! This is a common pattern in web architecture that gives you more control over how requests are handled.

**Create start_server.sh**

- Still inside the scripts folder, let's create a new file again.
- This time, let's name the file start_server.sh
- Add the following content to start_server.sh:

```
#!/bin/bash
sudo systemctl start tomcat.service
sudo systemctl enable tomcat.service
sudo systemctl start httpd.service
sudo systemctl enable httpd.service
```

**💡 What does this script do?**
This script starts both Tomcat (our Java application server) and Apache (our web server) and makes sure they'll restart automatically if the EC2 instance ever reboots.


**💡 What is systemctl?**
systemctl is the command-line tool for controlling services on modern Linux systems. Think of it as the master control panel for all the programs running in the background on your server. With systemctl, you can start services ("Hey Apache, time to wake up!"), stop them ("Tomcat, take a break"), check their status ("Is MySQL actually running?"), or set them to start automatically on boot. It's an essential tool for server management that gives you a standardized way to control just about any service on your Linux instance.


**💡 What does each line in start_server.sh do?**

**#!/bin/bash:** This tells the system to use the bash shell to interpret this script.
**sudo systemctl start tomcat10:** Fires up Tomcat (our Java application server) - like turning the key in the ignition.
**sudo systemctl enable tomcat10:** Sets Tomcat to auto-start whenever the server reboots - like setting your car to automatically start every morning.
**sudo systemctl start httpd:** Starts Apache HTTP server - turning on our web server frontend.
**sudo systemctl enable httpd:** Sets Apache to auto-start on reboot too.

Together, these commands ensure our application is up and running and will stay that way even if the server restarts.

**Create stop_server.sh script**
- Inside the scripts folder, create a new file named stop_server.sh
- Add the following content to stop_server.sh:

```
#!/bin/bash
isExistApp="$(pgrep httpd)"
if [[ -n $isExistApp ]]; then
sudo systemctl stop httpd.service
fi
isExistApp="$(pgrep tomcat)"
if [[ -n $isExistApp ]]; then
sudo systemctl stop tomcat.service
fi
```

**💡 What does stop_server.sh do?**
This script safely stops web server services by first checking if they're running. It uses pgrep to check for running processes of Apache (httpd) and Tomcat, and only attempts to stop the services if they are actually active. This prevents errors that could occur from trying to stop services that aren't running.

Specifically, the script:

Checks if Apache (httpd) is running
Stops Apache if it is active
Checks if Tomcat is running
Stops Tomcat if it is active


**💡 Extra for Experts:** Why do we check if the server is running before stopping it?
We check if the server is running first because trying to stop something that's not running can cause errors that might interrupt our deployment.

This makes our script more robust and reliable. If we blindly tried to stop services regardless of their state, we might get error messages that could cause our deployment to fail unnecessarily. Good scripts anticipate potential issues instead of assuming everything is in an ideal state - that's the difference between code that works in perfect conditions versus code that works in the real world!

- Save the stop_server.sh file by pressing Ctrl+S.
- Check that you have install_dependencies.sh, start_server.sh, and stop_server.sh inside the scripts folder.

**Create appspec.yml**

- Create a new file, but this time at the root of your project.
- Make sure this file is NOT inside the scripts folder!
- Let's name the file appspec.yml
- Add the following content to appspec.yml:

```
version: 0.0
os: linux
files:
  - source: /target/nextwork-web-project.war
    destination: /usr/share/tomcat/webapps/
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
```

**💡 What does each section in appspec.yml mean?**
The appspec.yml file is essentially the instruction manual for CodeDeploy. Here's what each part does:

version: 0.0: This is just the format version CodeDeploy expects (yes, starting at 0.0 is a bit odd).
os: linux: Tells CodeDeploy we're deploying to a Linux system, not Windows.
files: This section maps what files go where:
source: /target/nextwork-web-project.war: "Take this WAR file from our artifact..."
destination: /usr/share/tomcat10/webapps/: "...and put it here on the EC2 instance."
hooks: These are like event triggers that run at specific points in the deployment:
BeforeInstall: "Before you install the new version, run this script" - we're stopping the server first.
AfterInstall: "After the files are copied, run this" - we're installing dependencies.
ApplicationStart: "Once everything's ready, run this" - we're starting the server.
Each hook specifies the script location, a timeout (5 minutes max), and that it should run as the root user.
Think of appspec.yml as the choreographer that ensures everyone moves in the right sequence during deployment.

**💡 Extra for Experts**: CodeDeploy lifecycle events
Each phase in the appspec.yml file is a CodeDeploy lifecycle event. Lifecycle events are like the chapters in your deployment story - predefined points where you can hook in custom scripts to perform specific actions. They're the key to customizing exactly how your application gets deployed.

Think of it like baking a cake: there are distinct phases (mixing ingredients, baking, cooling, frosting) where different actions need to happen. CodeDeploy's lifecycle events are similar - BeforeInstall, Install, AfterInstall, ApplicationStart, etc.

- Save the appspec.yml file by pressing Ctrl+S.

**Update buildspec.yml File**
- Open buildspec.yml and modify the artifacts section to include appspec.yml and the scripts folder:

```
artifacts:
  files:
    - target/nextwork-web-project.war
    - appspec.yml
    - scripts/**/*
  discard-paths: no
```

**Commit and Push Changes to GitHub**

```
git add .
git commit -m "Adding CodeDeploy files"
git push
```
- Head to your GitHub repository in the browser. Let's check that the scripts folder and appspec.yml file are in your repository!

### Step 3 : Set Up CodeDeploy

Now, let's get to know CodeDeploy and set it up to automate the deployment of our web app!

**In this step, you're going to:**

- Create a CodeDeploy application, which is an application that you want to deploy.
- Create a CodeDeploy deployment group, which is like a folder of deployment settings for the same application.
- Give CodeDeploy the permission to access CodeArtifact.

<img width="1076" height="327" alt="image" src="https://github.com/user-attachments/assets/95dbd724-26d7-4f6a-bae2-3ca319e22a32" />


**Create CodeDeploy Application**

- Head to the CodeDeploy console.
- Select Applications in the left hand navigation menu.
- Select Create application.

**💡 What is a CodeDeploy application?**
A CodeDeploy application is like the main folder for your deployment project. It doesn't do much on its own, but it helps you organize everything related to deploying one application.


In more technical, AWS terms, a CodeDeploy application is a namespace or container that groups deployment configurations, deployment groups, and revisions for a specific application. Having separate CodeDeploy applications helps you manage multiple applications without mixing up their deployment resources.

- Enter nextwork-devops-cicd as the Application name.
- Choose EC2/On-premises as the Compute platform.
- Select Create application.

**Create Deployment Group**

- Select Create deployment group.

**💡 What is a CodeDeploy deployment group?**
A deployment group is a collection of EC2 instances that are grouped to deploy something together.

It's also where you plan out exactly where and how your application gets deployed on this group of instances. In other words, it's where you tell CodeDeploy "let's deploy this app to these specific servers, using this deployment pattern, with these load balancing settings, and handle failures this way."


The power of deployment groups is that you can have multiple groups within the same application - maybe one for your test environment, another for staging, and a third for production. Each can have different settings and target different sets of servers, but they all deploy the same core application. This adds to the (many) reasons why CI/CD tools are so powerful - in this case, CodeDeploy saves you the time it'd take to manually deploy the same app to each instance in each environment.

- Enter *nextwork-devops-cicd-deploymentgroup* as the Deployment group name.
- Under Service role, check if you have a service role available...
- Maybe not!

**💡 Why does CodeDeploy need IAM roles?**
CodeDeploy needs IAM roles to get permissions to access and manage AWS resources on your behalf. These permissions let CodeDeploy do things like:

Accessing EC2 instances to deploy applications.
Reading application artifacts from S3 buckets.
Updating Auto Scaling groups.
Write CloudWatch logs about what it's doing

**Create an IAM Role for CodeDeploy**

- Head to the IAM console.
- In the IAM console, select Roles from the left hand navigation bar.
- Select Create role.
- Choose AWS service as the trusted entity type.
- Choose CodeDeploy as the service and select CodeDeploy as the use case.
- Select Next.
- You'll notice the AWSCodeDeployRole default policy is suggested already - nice! That's all we need.
- Click on the plus button to take a look at the permissions this grants. There are many actions that we're allowing in this policy!
- Phew, this saves us the time from defining all these permission policies ourselves.
- Select Next.
- Expand the AWSCodeDeployRole policy to review its permissions.
- Review the EC2 permissions within the policy

**💡 What permissions does AWSCodeDeployRole policy include?**
The AWSCodeDeployRole policy lets CodeDeploy work with your EC2 instances. It includes permissions for:

Auto Scaling, so CodeDeploy can handle deployments when you're automatically scaling instances up or down
EC2, so CodeDeploy can interact with your instances - tag them, query them, and deploy to them
Elastic Load Balancing, so CodeDeploy can temporarily remove instances from load balancers during deployment
CloudWatch, so CodeDeploy can send logs and metrics so you can monitor what's happening
S3, so CodeDeploy can access the build artifacts stored in S3 buckets

Not that these permissions are carefully scoped to only what CodeDeploy actually needs to do its job, instead of everything in your AWS account. This is a best practice in security called the "principle of least privilege."

- Select Next.
- Enter NextWorkCodeDeployRole as the Role name.
- Add a description to help you remember why you created this role.

```
Allows CodeDeploy to call AWS services such as Auto Scaling on your behalf.
Created as a part of NextWork's Cl/CD Pipeline series.
```

- Review the Permissions policies and make sure AWSCodeDeployRole is attached.
- Select Create role.
- Let's double check that the NextWorkCodeDeployRole is created.

**Select Service Role in CodeDeploy**

- Head back to the CodeDeploy deployment group configuration tab.
- Looks like we'll need to refresh the page to use the new service role!
- Refresh the page, and re-enter nextwork-devops-cicd-deploymentgroup as Deployment group name.
- Select the newly created NextWorkCodeDeployRole as the Service role.
- Choose In-place as the Deployment type.
- Under Environment configuration, select Amazon EC2 instances.
- In Tag group 1, enter role as the Key.
- Enter webserver as the Value.
- Check the line below your tag settings - you might notice that 1 unique matched instance is found.
- Click Click here for details to view the matched instance.
- You'll see an EC2 instance called NextWorkCodeDeployEC2Stack::WebServer - that's the EC2 instance we launched from our CloudFormation template. This confirms     to us that the web app will be deployed onto that instance instance.

**Configure Agent and Deployment Settings**

- Now let's head back to your CodeDeploy Deployment group set up.
- Under Agent configuration with AWS Systems Manager, select Now and schedule updates and Basic scheduler with 14 Days.
- In Deployment settings, keep the default CodeDeployDefault.AllAtOnce

**💡 What are deployment settings?**
Deployment settings help you control how quickly you'd like to roll out your application.

**CodeDeployDefault.AllAtOnce** deploys the application to all instances in the deployment group at the same time. It's the fastest option, but also the riskiest - if something goes wrong, all your instances could be affected at once.


**💡 Extra for Experts:** How do you know what deployment setting to choose?
For production environments, you might choose more conservative options like OneAtATime (update one instance, make sure it works, then move to the next) or HalfAtATime (update 50% of instances, verify, then do the rest). These slower approaches reduce risk by limiting the blast radius of any potential issues.

For production systems with hundreds of instances, these settings become crucial. Imagine updating your banking app on all servers simultaneously vs. trying it on 10% first to make sure customers can still access their accounts! But since we only have one instance in our project, the cautious approach doesn't offer much benefit.

We're using AllAtOnce in this project because we only have one instance and we're learning - speed is more important than caution here!

- Deselect *Enable load balancing*.
- Select Create deployment group

### Step 4 : Create and Verify Deployment

It's time to put everything together and deploy our web application to the EC2 instance!

**In this step, you're going to:**

- Create a CodeDeploy deployment.
- Monitor the deployment process.
- Access and verify the deployed web application.

<img width="1072" height="337" alt="image" src="https://github.com/user-attachments/assets/2c599a98-1ae5-48a1-a042-937746c1f92c" />

**Create Deployment**

- In the deployment group details page, select **Create deployment**.

**💡 What is a CodeDeploy deployment?**
A CodeDeploy deployment represents a single update to your application, with its own unique ID and history. When you create a deployment, you're telling CodeDeploy:

Which version of the application to deploy (the revision)
Where to deploy it (the deployment group)
How to deploy it (the deployment settings) 

CodeDeploy then orchestrates the entire process - stopping services, copying files, running scripts, starting services - and keeps track of whether it succeeds or fails. You can monitor it happening in real-time and see a detailed log of each step.

- Under Revision type, make sure My application is stored in Amazon S3 is selected. That's because our deployment artifact is inside an S3 bucket!
- Head back to your S3 bucket called nextwork-devops-cicd.
- Click into the nextwork-devops-cicd-artifact build artifact.
- Copy the file's S3 URI.
- Paste the S3 URI into the Revision location field in CodeDeploy.
- Select .zip as the Revision file type.

**💡 What is a revision location? Why did we use our WAR/zip file?**
The revision location is the place where CodeDeploy looks to find your application's build artifacts. We're using the S3 bucket that's storing our WAR file, so CodeDeploy knows where to find the latest version of our web app it's deploying to the deployment EC2 instances!

- Next, we'll leave Additional deployment behavior settings as default.

**💡 Extra for Experts: What are Additional deployment behaviors settings and Deployment group overrides?**
These settings give you extra flexibility when you need to handle special cases without changing your standard deployment settings:

**Additional deployment behaviors** settings let you configure options like how to handle file permissions during deployment or whether to immediately allow traffic to new instances.

**Deployment group overrides** let you to override deployment group settings for a specific deployment - maybe you normally deploy one instance at a time (safe but slow), but for a critical security fix, you want to override that and deploy to all instances at once (faster but riskier).

- Select Create deployment.
- Off we go! CodeDeploy kicks off a deployment of your web app.
- Scroll down to Deployment lifecycle events and monitor the events by clicking View events.
- See the lifecycle events progressing, such as BeforeInstall, ApplicationStart, etc. These are the events you defined in appspec.yml!
- Whoops! After a few minutes of waiting, you might notice that you hit an error...
- Why do you think your deployment failed?
- Don't forget, CodeDeploy is deploying your web app by grabbing the latest build artifact from your S3 bucket.
- When was the last time you ran a build on CodeBuild? 🤔
- Aha - our last time running a build was before we added appspec and the deployment scripts!
- Because of this, your deployment instance isn't getting any of the scripts you've written - causing the error you're seeing now.
- Head back to your CodeBuild build project, and rebuild your project.
- Once your second build is a success, return to CodeDeploy and retry the deployment.

**Check Your Deployed App!**
- Wait until the deployment status says Success.
- Select the Instance ID in the Deployment lifecycle events panel. This takes you to the deployment EC2 instance you launched with CloudFormation.
- Get the Public IPv4 DNS of your EC2 instance from the EC2 console.
- Open the Public IPv4 DNS in a web browser.
- It might take a minute or two for the application to become fully accessible after deployment.
- WOOOHOOO! Welcome to your application!

## Part 6 : Build a CI/CD Pipeline with AWS

**🙋‍♀️ What is a CI/CD pipeline?**
A CI/CD pipeline automatically builds, tests, and deploys your code changes. Instead of manually running these steps, CodePipeline manages the entire process from code commit to deployment.

This means faster releases, fewer errors, and more time to focus on writing great code rather than managing the deployment process.

<img width="1077" height="387" alt="image" src="https://github.com/user-attachments/assets/44233aac-b5b0-479e-8e54-1ded3a05462d" />

**Get ready to:**

- Create a complete CI/CD pipeline with AWS CodePipeline.
- Connect your source code, build, and deployment stages.
- Test automatic deployments and learn how to roll back changes!

**💡 What is AWS CodePipeline? Why are we using it?**

So far, you've built CodeBuild projects that compile and test your code, and CodeDeploy applications that handle deployment. But there's still a missing piece: how do these services work together automatically?


Right now, you would need to:
- Manually build your project again if the code changes.
- Manually track which build artifacts should be deployed.
- Manually start deployments for new build artifacts.
- Manually restart the process if something fails.

**CodePipeline** solves these problems by orchestrating your entire workflow, and giving you visibility into the entire process in one place. It can:

✅ Automatically detect code changes in your GitHub repository.
✅ Automatically trigger CodeBuild to build a new project.
✅ Automatically start CodeDeploy with the new build artifacts.
✅ Automatically roll back a change if something fails.

Using CodePipeline transforms separate tools into a true CI/CD system, making deployments consistent, reliable, and fully automated - with no manual steps required from you!

**Here's the game plan...**

<img width="1070" height="387" alt="image" src="https://github.com/user-attachments/assets/470f8daa-fe02-421c-aebd-1646fd54002a" />

### Step 1 : Set Up your Pipeline
Let's dive right into creating our first pipeline! We'll start by setting up the basic pipeline structure and configuring its settings.

**In this step, you're going to:**

- Start creating a new pipeline in CodePipeline.

<img width="1077" height="375" alt="image" src="https://github.com/user-attachments/assets/9dbe4bcf-d838-4ed1-b955-8b9a4ef26fd9" />

**Start a new pipeline**

- Head to the CodePipeline console.
- Select the Getting started page. Welcome to CodePipeline!

**💡 What is AWS CodePipeline? Why are we using it?**
With **CodePipeline**, you can create a workflow that automatically moves your code changes through the build and deployment stage. In our case, you'll see how a new push to your GitHub repository automtically triggers a build in CodeBuild (continuous integration), and a then a deployment in CodeDeploy (continuous deployment)!

Using CodePipeline makes sure your deployments are consistent, reliable and happen automatically whenever you update your code - with less risk of human errors! It saves you time too.

- In the CodePipeline dashboard, Select Create pipeline.
- Select Build custom pipeline.
- Click Next.

**💡 Why build a custom pipeline?**
By choosing to build a custom pipeline, we get to define each stage and action step-by-step. This gives us a deeper understanding of how CodePipeline works and allows us to tailor the pipeline precisely to our needs. For learning purposes, building a custom pipeline from scratch is the best way to go !

**Configure Pipeline Settings**

- Name your pipeline nextwork-devops-cicd
- Under Execution mode, select Superseded.

**💡 What is Execution mode?**
**Execution mode** determines how CodePipeline handles multiple runs of the same pipeline.

In **Superseded mode**, if a new pipeline execution is triggered while another execution is already in progress, the newer execution will immediately take over and cancel the older one. This is perfect for making sure only the latest code changes are processed, which is exactly what we want for our CI/CD pipeline!

There are other execution modes available in CodePipeline:

In **Queued mode**, executions are processed one after another. If a pipeline is already running, any new executions will wait in a queue until the current execution finishes.

**Parallel mode** allows multiple executions to run at the same time, completely independently of each other. This can speed up the overall processing time if you have multiple branches or code changes that can be built and deployed concurrently.

- Under Service role, select New service role. Keep the default role name.

**💡 What is a service role?**
A **service role** is a special type of IAM role that AWS services like CodePipeline use to perform actions on your behalf. It's like giving CodePipeline permission to access other AWS resources it needs to run your pipeline, such as S3 buckets for storing artifacts or CodeBuild for building your code.

- Expand *Advanced settings*.
- Leave the default settings for *Artifact store, Encryption key, and Variables.*
- Click Next.

**💡 What are Artifact store, Encryption key, and Variables?**

**Artifact store**: Without an artifact store, there's no way for your build outputs to be passed to deployment! This S3 bucket is where CodePipeline automatically saves the files created at each stage - like your source code from GitHub and the build artifacts from CodeBuild - making them available to the next stage in your pipeline.

**Encryption key**: By default, CodePipeline encrypts everything in your artifact store using AWS managed keys. This keeps your code and build artifacts secure while they're being stored and transferred between stages. For most projects, this default encryption is perfectly sufficient.

**Variables**: Right now you might be manually tracking information like version numbers or build timestamps. Pipeline variables solve this by letting you pass dynamic values between different stages automatically. While we won't use variables in this project, they become essential in more complex pipelines when you need information generated in one stage (like a build number) to be available in another stage (like deployment).


### Step 2 : Configuring the Source, Build and Deploy Stages

Are you ready to pull together all the different parts of your CI/CD architecture?!

**In this step, you're going to:**

- Connect CodePipeline to your GitHub repository and branch.
- Connect CodePipeline to your CodeBuild project
- Connect CodePipeline to your CodeDeploy deployment group.
- Configure webhook events to automatically trigger the pipeline.

<img width="1072" height="390" alt="image" src="https://github.com/user-attachments/assets/ad706888-dd18-49cf-8cef-d50d9a7a120a" />

**Source Stage**

Now, let's configure the **Source stage** of our pipeline. This is where we'll tell CodePipeline where to fetch our source code from.
- In the **Source provider** dropdown, select *GitHub (via GitHub App).*

**💡 What is the Source stage?**
The Source stage is the very first step in any CI/CD pipeline. Its job is simple but crucial: it fetches the latest version of your code from your chosen repository whenever there are updates. Without this stage, your pipeline would have nothing to build or deploy.

CodePipeline supports various source providers, but for this project, we're using GitHub because that's where our web app's code is stored.

- Under Connection, select your existing GitHub connection
- Under Repository name, select nextwork-web-project
- Under Default branch, select master.
- Under Output artifact format, leave it as CodePipeline default.
- Click Next.

**💡 What is Output artifact format?**
*Output artifact format* determines how CodePipeline packages the source code it fetches from GitHub.

- *CodePipeline default*: This option packages the source code as a ZIP file, which is efficient for most deployment scenarios. It does not include Git metadata about the repository.

- *Full clone*: This option provides a full clone of the Git repository as an artifact, including Git history and metadata. This is useful if your build process requires Git history, but it results in a larger artifact size.

- Make sure that Webhook events is checked under Detect change events.

**💡 What are Webhook events?**
Webhook events let CodePipeline automatically start your pipeline whenever code is pushed to your specified branch in GitHub. This is what makes our pipeline truly "continuous" – it reacts to code changes in real-time!


**💡 How do Webhooks work?**
Webhooks are like digital notifications. When you enable webhook events, CodePipeline sets up a webhook in your GitHub repository. This webhook is configured to listen for specific events, such as code pushes to the master branch.

Whenever you push code to the master branch, GitHub sends a webhook event (a notification) to CodePipeline. CodePipeline then automatically starts a new pipeline execution in response to this event. It's a seamless way to automate your CI/CD process!


**Build stage**

The Build stage is where our source code gets transformed into a deployable build artifact.
We'll tell CodePipeline to use AWS CodeBuild to compile and package our web application.

- In the *Build provider* dropdown, select *AWS CodeBuild* from *Other build providers.*

**💡 What is the Build stage?**
The *Build stage* is where your source code gets compiled and packaged into something that can be deployed.

- Under *Project name*, select your existing CodeBuild project
- In the Project name dropdown, search for and select nextwork-devops-cicd.
- Leave the default settings for Environment variables, Build type, and Region.
- Under Input artifacts, SourceArtifact should be selected by default.
- Click Next.

**💡 What are Input artifacts?**
Input artifacts are the outputs from the previous stage that are used as inputs for the current stage. In our Build stage, we're using SourceArtifact, which is the ZIP file containing our source code that was outputted by the Source stage.

**Skip Test Stage**

- On the Add test stage page, click Skip test stage.

**Deploy Stage**

- In the Deploy provider dropdown, select AWS CodeDeploy.

**💡 What is the Deploy stage?**
The **Deploy stage** is the final step in our pipeline. It's responsible for taking the application artifacts (the output from the Build stage) and deploying them to the target environment, which in our case is an EC2 instance.

- Under Input artifacts, BuildArtifact should be selected by default.
- Under Application name, select your existing CodeDeploy application
- Under Deployment group, select your existing CodeDeploy deployment group
- Check the box for Configure automatic rollback on stage failure.
- Click Next.

**💡 What is automatic rollback?**
Automatic rollback is a safety net for your deployments. By enabling it, you're telling CodePipeline that if the Deploy stage fails for any reason, it should automatically revert to the last successful deployment. This helps minimize downtime and ensures that your application remains stable, even if a new deployment goes wrong.

### Step 3 : Run Your Pipeline!

Let's watch our pipeline run for the first time! This will help us verify that everything is working correctly.

**In this step, you're going to:**

- Finish creating your pipeline.
- Watch your pipeline start up and connect GitHub, CodeBuild and CodeDeploy!

<img width="1075" height="386" alt="image" src="https://github.com/user-attachments/assets/5ef4ed76-642c-47d3-b722-f009508fd67b" />

**Review Your Pipeline**

- On the Review page, take a moment to review all the settings you've configured for your pipeline.

**Confirm that the Pipeline settings are:**

- Pipeline name: nextwork-devops-cicd
- Pipeline type: V2
- Execution mode: SUPERSEDED
- Artifact store location: Default location
- Service role: New service role

**Confirm that the Source stage settings are:**

- Source provider: GitHub (via GitHub App)
- Output artifact format: CODE_ZIP
- Detect changes: true
- Connection ARN: Your CodeConnection ARN
- Full repository ID: Your GitHub account/nextwork-web-project
- Default branch: master
- Enable automatic retry on stage failure: Enabled

**Confirm that the Build stage settings are:**

- Action provider: AWS CodeBuild
- Project name: nextwork-devops-cicd
- Enable automatic retry on stage failure: Enabled

**Confirm that the Deploy stage settings are:**

- Action provider: AWS CodeDeploy
- Application name: nextwork-devops-cicd
- Deployment group name: nextwork-devops-cicd-deploymentgroup
- Configure automatic rollback on stage failure: Enabled

- Once you've reviewed all the settings and confirmed they are correct, click Create pipeline.

**Run Your Pipeline**

- After clicking Create pipeline, you will be taken to the pipeline details page.
- Note the pipeline diagram at the top of the page.

- CodePipeline automatically starts executing the pipeline as soon as it's created.
- You can see the progress of each stage in the pipeline diagram. The stages will transition from grey to blue (in progress) to green (success) as the pipeline     executes.

**💡 What are all the different kinds of statuses?**
As your pipeline runs, each stage will display a status:

Grey: Stage has not started yet.
Blue: Stage is currently in progress.
Green: Stage has completed successfully.
Red: Stage has failed.

- Wait for the pipeline execution to complete. You can monitor the status of each stage in the pipeline diagram.
- To see more details about each execution, click on the Executions tab above the pipeline diagram.

**💡 What are Pipeline executions?**
Pipeline executions represent each instance of your pipeline running. Every time your pipeline is triggered (either manually or automatically by a webhook), a new execution is created. Each execution has a unique ID and shows the status and details of each stage in that particular run.

- To view details of a specific stage execution, click on the Stage ID link in the Executions tab. For example, click on the Source stage ID to see details about   the source code retrieval.
- Wait for all stages in the pipeline diagram to turn green, which means your pipeline is all set up using the latest code change!

### Step 4 : Test Your Pipeline!

It's time for the ULTIMATE test for this project... let's see how CodePipeline handles a code change!
Testing with a code change will confirm that our pipeline is automatically triggered and deploys our updates.

**In this step, you're going to:**

- Test the pipeline by making a code change and pushing it to GitHub.

**Test Pipeline with Code Change**

- Open your web app code in your local IDE (e.g., VS Code).
- Open the index.jsp file located in src/main/webapp/.
- Add a new line in the <body> section of index.jsp:

```
<p>If you see this line, that means your latest changes are automatically deployed into production by CodePipeline!</p>
```

- Save the index.jsp file.
- Open your terminal and navigate to your local git repository for the web app.
- Commit and push the changes to your GitHub repository using the following commands:

```
git add .
git commit -m "Update index.jsp with a new line to test CodePipeline"
git push origin master
```

- Go back to the CodePipeline console and watch your pipeline react to the code change 👀
- You should see a new execution starting automatically after you push the changes to GitHub.
- Click on the Source stage box in the pipeline diagram.
- Scroll down in the stage details panel to see the commit message.
- Click on the Commit ID link in the Source stage details panel.
- This should open the commit page in your GitHub repository in a new browser tab.
- Verify that the commit page shows the code changes you just pushed (the new line you added to index.jsp)!
- Wait for the Build and Deploy stages to complete successfully (turn green) in the CodePipeline console.

**Verify Automated Deployment**

Let's try accessing the web app to see your code change live!

- To find the Public IPv4 DNS, in the CodePipeline console, click on the Deploy stage, then click on the CodeDeploy link in the details panel.
- In the CodeDeploy console, scroll down to Deployment lifecycle events and click on the Instance ID.
- On the EC2 instance summary page, copy the Public IPv4 DNS.
- Paste the copied Public IPv4 DNS in a new browser tab and press Enter.
- You should see your web application with the new line you added:

  <img width="707" height="415" alt="image" src="https://github.com/user-attachments/assets/6a72d71f-5f7b-4d46-82df-66549e5d03bb" />

  ## WOOOOOO 😭🙏 This confirms that your latest code changes were automatically deployed by CodePipeline.


  ### Step 5 : Trigger a Rollback in CodePipeline

  Before you finish this project - want to test an important emergency procedure?

Your secret mission, should you choose to accept it, is to trigger a manual rollback in your CI/CD pipeline. This is going to give you hands-on experience with one of the most critical operational procedures in a production environment - handling deployment failures and restoring service quickly!

**In this secret mission, you're going to:**

- Manually trigger a rollback in your CodePipeline deploy stage.
- Verify that your web application reverts correctly to its previous state.
- Demonstrate your disaster recovery skills for your DevOps portfolio!

**Trigger Rollback**

- Go back to the CodePipeline console and select your pipeline (nextwork-devops-cicd).
- In the pipeline diagram, locate the Deploy stage.
- Click on the three dots (...) in the top-right corner of the Deploy stage box.
- Select Start rollback from the dropdown menu.

- In the Rollback to dialog box, select the previous execution ID from the dropdown menu.
- Click Start rollback.
- Observe the pipeline execution in the CodePipeline console.
- You should see the Deploy stage now showing Rollback in progress.
- Wait for the Deploy stage to complete the rollback (turn green again).
- The Deploy stage should now be using the commit message from the previous successful deployment, indicating a rollback. Compare this to the commit messages in    the Deploy and Source stages - they're still using the latest commit!
- You can also check the Stage tab on your Deploy card to see the rollback's success.

💡 In what scenarios would this rollback be useful?
Rolling back only the Deploy stage is useful when the latest deployment has errors, but the source code and build processes are correct and we don't want to undo some valid code and build changes.

**Scenarios where the deployment might have errors include:**

One of your deployment scripts e.g. install_dependecies.sh had a bug that failed to set up the environment correctly.
Your latest deployment started using a new third-party service that is currently down or unstable.
The new deployment had performance issues (e.g. assets were loading slower than expected) that didn't come up during testing.

**Verify Rollback in Web Application**
- Refresh your web application in the browser using the same Public IPv4 DNS.
- The new line you added (<p>If you see this line, that means your latest changes are automatically deployed into production by CodePipeline!</p>) is now gone!
- The web app should be reverted to the previous version, confirming the rollback.

## Delete your Resources

**Cloudformation**

- Head to the CloudFormation console.
- Select your deployment EC2 stack.
- Click Delete.
- Confirm the deletion by clicking Delete stack.

**Code Services**

*CodePipeline*

- Head to the CodePipeline console.
- Select Pipelines from the left-hand menu.
- Select the pipeline named nextwork-devops-cicd.
- Select Delete.
- Type delete in the confirmation field.
- Select Delete.

*CodeDeploy*

- Head to the CodeDeploy console.
- Select Applications from the left hand menu.
- Select the nextwork-devops-cicd application.
- Click Delete application.
- Confirm the deletion by typing delete and clicking Delete.

*CodeBuild*

- Head to the CodeBuild console.
- Select Build projects from the left hand menu.
- Select the nextwork-devops-cicd project.
- Click Delete build project.
- Confirm the deletion by typing delete and clicking Delete.

*CodeArtifact*

- Head to the CodeArtifact console.
- Select Repositories from the left hand menu.
- Select the nextwork-devops-cicd repository.
- Click Delete repository.
- Confirm the deletion by typing delete and clicking Delete repository.
- Select Domains from the left hand menu.
- Select the nextwork domain.
- Click Delete domain.
- Confirm the deletion by typing delete and clicking Delete domain.

*CodeConnection*

- Expand the Settings arrow at the bottom of the left hand navigation panel.
- Select Connections.
- Select your connection.
- Select Delete
- Confirm the deletion by typing delete and clicking Delete

**IAM**

- Head to the IAM console.
- Select Roles from the left hand menu.
- Search for and delete the following roles:
- ec2-instance-nextwork-cicd
- aws-codedeploy-role
- codebuild-nextwork-devops-cicd-service-role
- AWSCodePipelineServiceRole

*Select Policies from the left hand menu.*

- Search for and delete the following polcies:
- codeartifact-nextwork-consumer-policy
- CodeBuildBasePolicy-nextwork-devops-cicd
- CodeBuildCloudWatchLogsPolicy-nextwork-devops
- CodeBuildCodeConnectionsSourceCredentialsPolicy-nextwork
- AWSCodePipelineServiceRole

**EC2**

- Head to the EC2 console.
- Select Instances from the left hand menu.
- Select the nextwork-devops-yourname instance.
- Click Instance state, then Terminate instance.
- Confirm termination by clicking Terminate.

**S3**
- Head to the S3 console.
- Select Buckets from the left hand menu.
- Select the nextwork-devops-cicd S3 bucket (your build artifacts bucket)
- Click Empty bucket.
- Confirm emptying the bucket by typing permanently delete and clicking Empty bucket.
- Once the bucket is empty, select the bucket again and click Delete bucket.
- Confirm deletion by typing the bucket name and clicking Delete bucket.
- 🚨 Also delete the bucket created by CloudFormation!
- CloudFormation automatically creates a new bucket to store templates you upload when you create a new stack. The bucket's name should start with cf
- CodePipeline also creates a new bucket to store artifacts created in the pipeline. The bucket's name should start with codepipeline

**You've learned how to:**

🚀 Set up a CodePipeline pipeline to automate your software release process.
📦 Configure a Source stage to fetch code changes from GitHub.
🛠️ Set up a Build stage using AWS CodeBuild to compile your web application.
⚙️ Configure a Deploy stage using AWS CodeDeploy to deploy your web application to EC2.
💎 Test rollbacks on the deployment - without affecting your pipeline's Source or Build!
