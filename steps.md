# Java Application CI/CD with GitHub Actions

## Project Overview
This project demonstrates an automated CI/CD pipeline for deploying a Java web application to Apache Tomcat using GitHub Actions.
The pipeline automatically builds the Java application using Maven, performs SonarQube code analysis, stores the generated WAR file as an artifact, and deploys the application to Tomcat.


## Architecture / CI-CD Flow

Developer

   ↓

GitHub Repository

   ↓

GitHub Actions

   ↓

Self-Hosted Runner

   ↓

Checkout Code

   ↓

JDK 17

   ↓

Maven

   ↓

Build WAR

   ↓

SonarQube Analysis

   ↓

Upload Artifact

   ↓

Download Artifact

   ↓

Tomcat Deployment

   ↓

Verify Application


## Prerequisites

- AWS EC2
- Amazon Linux 2023
- Java 17
- Maven
- GitHub Repository
- GitHub Actions
- Self-hosted Runner
- Apache Tomcat
- SonarQube

## 4. GitHub Repository Setup

#### Create repository

Repository : java-project-maven-new  (https://github.com/chhatrapal7/java-project-maven-new.git)
Go to https://github.com/chhatrapal7/java-project-maven-new and download the zip file
Create a new repo in your account and upload the files expect .github/workflow

#### Github Actions --> search for Java with Maven --> Configure
- Add pom.xml
- Add src/

## Tomcat Server Setup

Launch Amazon Linux 2023 and setup Tomcat with c7i.xlarge instance
connect to SSH terminal software

```bash
sudo -i

vi tom.sh

dnf install java-17-amazon-corretto -y
wget https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.25/bin/apache-tomcat-11.0.25.tar.gz
tar -zxvf apache-tomcat-11.0.25.tar.gz
sed -i '56  a\<role rolename="manager-gui"/>' apache-tomcat-11.0.25/conf/tomcat-users.xml
sed -i '57  a\<role rolename="manager-script"/>' apache-tomcat-11.0.25/conf/tomcat-users.xml
sed -i '58  a\<user username="tomcat" password="root123456" roles="manager-gui, manager-script"/>' apache-tomcat-11.0.25/conf/tomcat-users.xml
sed -i '59  a\</tomcat-users>' apache-tomcat-11.0.25/conf/tomcat-users.xml
sed -i '56d' apache-tomcat-11.0.25/conf/tomcat-users.xml
sed -i '21d' apache-tomcat-11.0.25/webapps/manager/META-INF/context.xml
sed -i '22d' apache-tomcat-11.0.25/webapps/manager/META-INF/context.xml
sh apache-tomcat-11.0.25/bin/startup.sh
```

http://Public IP:8080 


## GitHub Actions Workflow

Add the below code to the workflow, this will just build but will not deploy because we need to 
add environment secrets , the below code will create a env called production

```bash
name: Java CI/CD Pipeline to Tomcat

on:
  push:
    branches: [ main ]   # Trigger on push to main branch

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 1. Checkout code
      - name: Checkout code
        uses: actions/checkout@v4

      # 2. Set up JDK
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      # 3. Build with Maven
      - name: Build with Maven
        run: mvn clean package -DskipTests

      # 4. Upload WAR as artifact (optional, for later jobs)
      - name: Upload WAR artifact
        uses: actions/upload-artifact@v4
        with:
          name: myapp
          path: target/*.war

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment: production

    steps:
      # 5. Download WAR artifact
      - name: Download WAR artifact
        uses: actions/download-artifact@v4
        with:
          name: myapp
```

### GitHub Secrets

In my repository, add these under Settings → Secrets and variables → Actions: -->  Environment secrets --> Configure Name production

TOMCAT_USER → t****t

TOMCAT_PASSWORD → r********6

TOMCAT_HOST → tomcat ec2 ip:8080

### Now Add Deployment for the workflow

```bash
name: Java CI/CD Pipeline to Tomcat

on:
  push:
    branches: [ main ]   # Trigger on push to main branch

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 1. Checkout code
      - name: Checkout code
        uses: actions/checkout@v4

      # 2. Set up JDK
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      # 3. Build with Maven
      - name: Build with Maven
        run: mvn clean package -DskipTests

      # 4. Upload WAR as artifact (optional, for later jobs)
      - name: Upload WAR artifact
        uses: actions/upload-artifact@v4
        with:
          name: myapp
          path: target/*.war

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment: production

    steps:
      # 5. Download WAR artifact
      - name: Download WAR artifact
        uses: actions/download-artifact@v4
        with:
          name: myapp

      # 6. Deploy WAR via Tomcat Manager
      - name: Deploy to Tomcat
        run: |
          WAR_FILE=$(ls *.war)
          echo "Deploying $WAR_FILE to Tomcat..."
          curl -v --upload-file "$WAR_FILE" \
            "http://${{ secrets.TOMCAT_USER }}:${{ secrets.TOMCAT_PASSWORD }}@${{ secrets.TOMCAT_HOST }}/manager/text/deploy?path=/myapp&update=true"
```

War is stored in runner 
find . -name *.war


## Self-Hosted Runner Setup

Launch Amazon Linux 2023 , t3.large

connect to SSH terminal software

#### Be in ec2-user, don't use root

```bash
mkdir actions-runner && cd actions-runner
```

### Download Latest Runner Package
```bash
curl -o actions-runner-linux-x64-2.329.0.tar.gz -L
https://github.com/actions/runner/releases/download/v2.329.0/actions-runner-linux-x64-2.329.0.tar.gz
```

### To validate if the hash to ensure actions runner will be installed correctly on the instance, first install shasum a package that computes SHA messages using the command below.
```bash
sudo yum install perl-Digest-SHA -y
```

#### Validate the hash
```bash
echo "194f1e1e4bd02f80b7e9633fc546084d8d4e19f3928a324d512ea53430102e1d
actions-runner-linux-x64-2.329.0.tar.gz" | shasum -a 256 -c
```

#### Extract the installer
```bash
tar xzf ./actions-runner-linux-x64-2.329.0.tar.gz
```

#### First, install lld one of Dotnet core dependencies
```bash
sudo dnf makecache --refresh
```

#### install lld
```bash
sudo dnf -y install lld
```

#### install libicu
```bash
sudo yum install libicu -y

```

### Head Back to the Add self hosted runner page on your GitHub repo and copy the config command specific to your repository in the configure section and paste in the linux machine.
```bash
./config.sh --url https://github.com/ReyazShaik/730pm-githubactions-project --token AKYYAHYVLQTZTYKSB6NZT53I57EMC
./run.sh

```

### Install the Runner as a Service
```bash

sudo ./svc.sh install

sudo ./svc.sh start

sudo ./svc.sh status

```

### Now go to repo and change runs_on  ubuntu-latest to self-hosted

```bash
name: Java CI/CD Pipeline to Tomcat

on:
  push:
    branches: [ main ]   # Trigger on push to main branch

jobs:
  build:
    runs-on: [self-hosted, linux]

    steps:
      # 1. Checkout code
      - name: Checkout code
        uses: actions/checkout@v4

      # 2. Set up JDK
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      # 3. Install Maven if missing
      - name: Ensure Maven is installed
        run: |
          if ! command -v mvn &> /dev/null; then
            echo "Maven not found. Installing..."
            sudo dnf install -y maven
          else
            echo "Maven is already installed"
          fi
          mvn -version

      # 3. Build with Maven
      - name: Build with Maven
        run: mvn clean package -DskipTests

      # 4. Upload WAR as artifact (optional, for later jobs)
      - name: Upload WAR artifact
        uses: actions/upload-artifact@v4
        with:
          name: myapp
          path: target/*.war

  deploy:
    runs-on: [self-hosted, linux]
    needs: build
    environment: production

    steps:
      # 5. Download WAR artifact
      - name: Download WAR artifact
        uses: actions/download-artifact@v4
        with:
          name: myapp

      # 6. Deploy WAR via Tomcat Manager
      - name: Deploy to Tomcat
        run: |
          WAR_FILE=$(ls *.war)
          echo "Deploying $WAR_FILE to Tomcat..."
          curl -v --upload-file "$WAR_FILE" \
            "http://${{ secrets.TOMCAT_USER }}:${{ secrets.TOMCAT_PASSWORD }}@${{ secrets.TOMCAT_HOST }}/manager/text/deploy?path=/myapp&update=true"
```

## SonarQube Setup

#### Launch SonarQube EC2 with t2.medium
```bash
sudo -i

vi sonar.sh

#! /bin/bash
cd /opt/
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.6.50800.zip
unzip sonarqube-8.9.6.50800.zip
sudo dnf install java-17-amazon-corretto -y
useradd sonar
chown sonar:sonar sonarqube-8.9.6.50800 -R
chmod 777 sonarqube-8.9.6.50800 -R
su - sonar
# use the below command manually after installation
#sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start
#echo "user=admin & password=admin"
```

#### Run Sonar File 
```bash
sh sonar.sh
```

#### Start the SOnar
```bash
sh /opt/sonarqube-8.9.6.50800/bin/linux-x86-64/sonar.sh start
```

http://ip:9000 username: admin, password: admin

Add Project --> Manuallly --> projectname: hotstar

copy this token --> e52e9d827cf0f5ff8ad765c5dfa229dc8a391d80

#### Secrets in GitHub:

Go to GitHub repo ---> settings --> secrets and variables --> actions --> Repository secrets --> New Repository secrets
we can use env secrets or repo secrets also

#### Note:     
Repository secrets → available to one repo
Environment secrets → scoped to specific environments (dev, prod)



SONAR_HOST → e.g., http://your-sonarqube-server:9000
SONAR_TOKEN → a token generated in SonarQube with Execute Analysis permission


```bash
name: Java CI/CD Pipeline to Tomcat

on:
  push:
    branches: [ main ]   # Trigger on push to main branch

jobs:
  build:
    runs-on: [self-hosted, linux]

    steps:
      # 1. Checkout code
      - name: Checkout code
        uses: actions/checkout@v4

      # 2. Set up JDK
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      # 3. Install Maven if missing
      - name: Ensure Maven is installed
        run: |
          if ! command -v mvn &> /dev/null; then
            echo "Maven not found. Installing..."
            sudo dnf install -y maven
          else
            echo "Maven is already installed"
          fi
          mvn -version
      # 4. Cache Maven dependencies
      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: |
            ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: |
            ${{ runner.os }}-maven-

      # 5. Build with Maven
      - name: Build with Maven
        run: mvn clean package -DskipTests

      # 6. Upload WAR as artifact (optional, for later jobs)
      - name: Upload WAR artifact
        uses: actions/upload-artifact@v4
        with:
          name: myapp
          path: target/*.war

      # 7. Install SonarQube Scanner CLI
      - name: Install SonarQube Scanner CLI
        run: |
          if [ ! -d "$HOME/sonar-scanner-cli" ]; then
            echo "Downloading SonarQube Scanner CLI..."
            wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.3.0.5189-linux-x64.zip -O /tmp/sonar-scanner.zip
            unzip /tmp/sonar-scanner.zip -d $HOME
            mv $HOME/sonar-scanner-7.3.0.5189-linux-x64 $HOME/sonar-scanner-cli
          fi
          export PATH=$HOME/sonar-scanner-cli/bin:$PATH
          sonar-scanner -v
      # 8. SonarQube analysis
      - name: SonarQube Analysis
        env:
          SONAR_HOST: ${{ secrets.SONAR_HOST }}
          SONAR_LOGIN: ${{ secrets.SONAR_TOKEN }}
        run: |
          echo "SonarQube server URL: $SONAR_HOST"
          curl -I $SONAR_HOST
          mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar \
            -Dsonar.projectKey=hotstar \
            -Dsonar.host.url=$SONAR_HOST \
            -Dsonar.login=$SONAR_LOGIN

  deploy:
    runs-on: [self-hosted, linux]
    needs: build
    environment: production

    steps:
      # 9. Download WAR artifact
      - name: Download WAR artifact
        uses: actions/download-artifact@v4
        with:
          name: myapp

      # 10. Deploy WAR via Tomcat Manager
      - name: Deploy to Tomcat
        run: |
          WAR_FILE=$(ls *.war)
          echo "Deploying $WAR_FILE to Tomcat..."
          curl -v --upload-file "$WAR_FILE" \
            "http://${{ secrets.TOMCAT_USER }}:${{ secrets.TOMCAT_PASSWORD }}@${{ secrets.TOMCAT_HOST }}/manager/text/deploy?path=/myapp&update=true"
     
      # 11. Optional: verify deployment
      - name: Verify deployment
        run: |
          echo "HOST=$TOMCAT_HOST"
          echo "USER=$TOMCAT_USER"
          curl -I http://${{ secrets.TOMCAT_HOST }}/myapp
```

## Deployment Verification

```bash
curl -I http://${TOMCAT_HOST}/myapp
```

## Final CI/CD Flow

Push code

   ↓

GitHub Actions Trigger

   ↓
   
Self-hosted Runner

   ↓
Checkout

   ↓
   
Java Setup

   ↓

Maven

   ↓
   
SonarQube

   ↓
   
Build WAR

   ↓
   
Artifact

   ↓

Tomcat

   ↓

Verify


## What I Learned

- GitHub Actions
- YAML workflow
- Self-hosted runner
- Maven build
- WAR artifact
- GitHub Secrets
- Environment
- SonarQube
- Tomcat deployment
- CI/CD automation
