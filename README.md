# 🚀 Java Application CI/CD with GitHub Actions

A complete **CI/CD pipeline for a Java Web Application** using **GitHub Actions, Self-Hosted Runner, Maven, SonarQube, and Apache Tomcat**.

In this project, a Disney+ Hotstar-inspired Java web application is automatically built, analyzed, packaged as a WAR file, deployed to Apache Tomcat, and verified using an automated GitHub Actions pipeline.

---

# 📖 Project Overview

The purpose of this project is to implement an automated **CI/CD pipeline** for a Java Web Application.

The application source code is maintained in **GitHub**. Whenever a developer pushes new code to the `main` branch, GitHub Actions automatically starts the pipeline.

The pipeline performs:

1. Checkout source code
2. Set up JDK 17
3. Install/check Maven
4. Build the application
5. Perform SonarQube code quality analysis
6. Generate WAR file
7. Upload WAR artifact
8. Download WAR artifact
9. Deploy WAR to Apache Tomcat
10. Verify the deployed application

This removes many repetitive manual deployment activities and provides a consistent application delivery process.

---

# ☁️ What is CI/CD?

## CI — Continuous Integration

**Continuous Integration** means developers frequently integrate their code into a shared repository.

Whenever new code is pushed, automated processes can:

* Checkout the code
* Compile the application
* Run tests
* Perform code quality checks
* Generate build artifacts

The main goal of CI is to detect problems early.

---

## CD — Continuous Delivery / Continuous Deployment

**Continuous Delivery** means keeping the application ready for deployment through an automated process.

**Continuous Deployment** goes one step further and automatically deploys the application to the target environment after successful pipeline stages.

In this project, the pipeline automatically deploys the generated WAR file to Apache Tomcat.

---

# 🤔 Why Do We Need CI/CD?

Without CI/CD, many software delivery activities are performed manually.

For every new code change, someone may need to:

* Pull the latest source code
* Configure/check Java
* Run Maven
* Build the application
* Create the WAR file
* Perform code quality checks
* Copy the WAR file to the server
* Deploy the application
* Verify the application

When these activities happen frequently, the process becomes repetitive and time-consuming.

CI/CD automates these activities and makes the software delivery process more:

* ⚡ Fast
* 🤖 Automated
* 🔄 Consistent
* 🔐 Controlled
* ✅ Reliable
* 📈 Scalable

---

# 🔄 Project CI/CD Flow

```text
                  ┌──────────────────┐
                  │    Developer     │
                  └────────┬─────────┘
                           │
                           │ git push
                           ▼
                  ┌──────────────────┐
                  │ GitHub Repository│
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │  GitHub Actions  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Self-Hosted      │
                  │ Runner           │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │  Checkout Code   │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │     JDK 17       │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │      Maven       │
                  │      Build       │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │    SonarQube     │
                  │     Analysis     │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │    WAR File      │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Upload Artifact  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Download Artifact│
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │ Tomcat Deployment│
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │Verify Application│
                  └──────────────────┘
```

---

# 🛠️ Technologies Used

| Technology         | Purpose                               |
| ------------------ | ------------------------------------- |
| Git                | Source code version control           |
| GitHub             | Source code repository                |
| GitHub Actions     | CI/CD automation                      |
| Self-Hosted Runner | Executes GitHub Actions jobs          |
| Java 17            | Application runtime/build environment |
| Maven              | Build and package Java application    |
| SonarQube          | Code quality analysis                 |
| Apache Tomcat      | Java web application server           |
| AWS EC2            | Cloud infrastructure                  |
| Linux              | Server operating system               |
| YAML               | GitHub Actions workflow configuration |

---

# 🏗️ Project Architecture

The project uses AWS EC2 instances for the required infrastructure.

```text
                     GitHub
                       │
                       │
                       ▼
               GitHub Actions
                       │
                       ▼
             Self-Hosted Runner
                 AWS EC2
                       │
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
         Maven Build         SonarQube
             │              Code Analysis
             │
             ▼
          WAR File
             │
             ▼
        Tomcat Server
          AWS EC2
             │
             ▼
       Java Web Application
```

---


# 🖥️ Self-Hosted Runner

Instead of using only GitHub-hosted infrastructure, this project uses a **Self-Hosted Runner**.

The runner is installed on an AWS EC2 instance running Amazon Linux.

The runner receives jobs from GitHub Actions and executes the workflow commands on the EC2 instance.

Basic setup:

```text
GitHub
   ↓
GitHub Actions
   ↓
Self-Hosted Runner
   ↓
AWS EC2
```

This provides more control over the execution environment.

---

# 🔍 SonarQube Analysis

SonarQube is used to analyze the source code and identify potential code quality issues.

It helps developers understand areas such as:

* Code quality
* Bugs
* Vulnerabilities
* Code smells
* Maintainability

SonarQube provides a centralized view of the application's code quality.

---

# 📦 Artifact Management

After the Maven build, the generated WAR file is treated as a build artifact.

The pipeline:

```text
Maven Build
     ↓
WAR File
     ↓
Upload Artifact
     ↓
Download Artifact
     ↓
Deployment
```

This separates the **build process** from the **deployment process**.

The deployment job can download the WAR artifact produced by the build job.

---

# 🚀 Tomcat Deployment

Apache Tomcat is used as the Java web application server.

The generated WAR file is deployed to Tomcat through the Tomcat Manager interface.

The deployment process is automated through the GitHub Actions workflow.

The application is deployed with a context path such as:

```text
/myapp
```

---

# ✅ Application Verification

After deployment, the pipeline verifies whether the application is accessible.

For example:

```bash
curl -I http://${{ secrets.TOMCAT_HOST }}/myapp
```

This provides an automated verification step after deployment.

The goal is to detect deployment problems immediately rather than relying only on manual browser testing.

---

# 🔐 GitHub Secrets

Sensitive information should not be written directly inside the workflow file.

The project uses GitHub Environment Secrets for Tomcat credentials and host information.

Example secrets:

```text
TOMCAT_USER
TOMCAT_PASSWORD
TOMCAT_HOST
```

These values are referenced inside the workflow using:

```yaml
${{ secrets.TOMCAT_USER }}
${{ secrets.TOMCAT_PASSWORD }}
${{ secrets.TOMCAT_HOST }}
```

This helps prevent sensitive credentials from being exposed in the source code.

---

# 📁 Repository Structure

The project repository is organized approximately as follows:

```text
Java-Application-CI-CD-with-GitHub-Actions/
│
├── .github/
│   └── workflows/
│       └── main.yml
│
├── src/
│   ├── main/
│   └── test/
│
├── pom.xml
│
├── README.md
│
└── step.md
```

### Important files

### `.github/workflows/main.yml`

Contains the GitHub Actions CI/CD pipeline configuration.

### `pom.xml`

Contains Maven project configuration and dependencies.

### `src/`

Contains the Java application source code and related resources.

### `step.md`

Contains the detailed implementation/setup steps followed during the project.

### `README.md`

Contains the project overview, architecture, technologies, CI/CD explanation, and learning outcomes.

---

# 📚 What I Learned

Through this project, I gained practical experience with:

### Git & GitHub

* Repository management
* Git commands
* Branches
* Commits
* Push and pull workflow

### GitHub Actions

* Creating workflow files
* Defining jobs and steps
* Configuring triggers
* Using GitHub Actions
* Working with artifacts
* Using secrets
* Managing self-hosted runners

### Maven

* Maven project structure
* `pom.xml`
* Maven lifecycle
* `mvn clean package`
* WAR generation

### SonarQube

* Integrating SonarQube with CI/CD
* Code quality analysis
* Understanding code quality reports

### Apache Tomcat

* Tomcat setup
* Tomcat Manager
* WAR deployment
* Application verification

### AWS EC2

* Launching Linux EC2 instances
* Server configuration
* Running DevOps tools on cloud infrastructure

### Linux

* Package installation
* File management
* Permissions
* Services
* Server administration

---

# ⭐ Benefits of This CI/CD Project

The biggest benefit is **automation**.

Instead of manually performing the complete deployment process after every change, the pipeline automates the process.

### Before CI/CD

```text
More Manual Work
       ↓
More Time
       ↓
Higher Chance of Human Error
       ↓
Slower Delivery
```

### After CI/CD

```text
Code Push
    ↓
Automatic Pipeline
    ↓
Automatic Build
    ↓
Automatic Analysis
    ↓
Automatic Artifact Handling
    ↓
Automatic Deployment
    ↓
Automatic Verification
```

### Key Benefits

* ⚡ Faster software delivery
* 🤖 Less manual work
* 🔄 Repeatable deployment process
* ✅ Consistent build and deployment
* 🔍 Automated code quality analysis
* 📦 Proper artifact management
* 🚀 Automated Tomcat deployment
* 🛠️ Easier troubleshooting through pipeline logs
* 📈 Better foundation for scalable DevOps practices

---

# 🎯 Why This Project is Important

This project demonstrates how multiple DevOps tools can work together as one automated workflow.

Instead of learning GitHub Actions, Maven, SonarQube, Tomcat, Linux, and AWS separately, this project connects them into a practical CI/CD implementation.

The complete flow is:

```text
Source Code
     ↓
Version Control
     ↓
CI/CD Automation
     ↓
Build
     ↓
Code Quality
     ↓
Artifact
     ↓
Deployment
     ↓
Verification
```

This represents an important part of a real-world DevOps workflow.

---

# 🔗 Project Repository

The complete source code and implementation details are available here:

**GitHub Repository:**

https://github.com/chhatrapal7/Java-Application-CI-CD-with-GitHub-Actions

The repository contains:

* `README.md`
* `step.md`
* Source code
* GitHub Actions workflow
* `pom.xml`
* Screenshots
* Complete project implementation details

---

# 👨‍💻 Project Summary

This project demonstrates a complete Java application CI/CD pipeline using:

```text
GitHub
   +
GitHub Actions
   +
Self-Hosted Runner
   +
JDK 17
   +
Maven
   +
SonarQube
   +
Artifact Management
   +
Apache Tomcat
   +
AWS EC2
```

The project helped me understand how to automate the journey from **developer code commit to application deployment and verification**.

---

# 🙌 Conclusion

CI/CD is not only about making deployment faster.

It is about creating an **automated, consistent, repeatable, and reliable software delivery process**.

In this project, GitHub Actions acts as the automation engine that connects source code, build, code quality analysis, artifact management, deployment, and verification into a single workflow.

This project gave me practical hands-on experience in implementing a complete CI/CD pipeline for a Java Web Application.

---

# Author

**Chhatrapal Janghel**

AWS Cloud | DevOps Engineer | CI-CD Pipelines

- **Portfolio:** https://chhatrapal.in

- **LinkedIn:** https://www.linkedin.com/in/chhatrapal7/
