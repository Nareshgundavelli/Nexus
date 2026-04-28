# 🚀 Sonatype Nexus Setup & Maven Artifact Deployment on AWS Ubuntu 22.04

# 📌 Project Overview

## What is Nexus?

**Sonatype Nexus Repository** is an artifact repository manager used to store, manage, and distribute build artifacts generated from applications.

Artifacts can include:

- JAR files
- WAR files
- ZIP files
- Docker Images
- Shared Internal Libraries

Nexus is widely used in DevOps and CI/CD pipelines to maintain a central repository for application packages.

---

## Why Do We Use Nexus?

Organizations use Nexus for:

- Centralized artifact storage
- Version management
- Backup of build files
- Sharing artifacts across teams
- Integration with CI/CD tools like Jenkins
- Faster deployments
- Dependency management

---

## Difference Between GitHub and Nexus

| GitHub | Nexus |
|--------|-------|
| Stores source code | Stores build artifacts |
| Used by developers | Used by DevOps / Build teams |
| Git version control | Binary repository manager |
| Code collaboration | Artifact storage & delivery |

---

# ☁️ AWS Server Setup

## Launch EC2 Instance

- OS: Ubuntu Server 22.04
- Instance Type: t2.micro / t2.small
- Key Pair: Existing or New Key
- Storage: Default

---

## Security Group Ports to Allow

| Port | Purpose |
|------|---------|
| 22 | SSH Access |
| 8081 | Nexus UI Access |

---

# 🔐 Connect to Server

```bash
ssh -i your-key.pem ubuntu@<public-ip>
⚙️ System Update
sudo apt update && sudo apt upgrade -y
☕ Install Java
sudo apt install openjdk-17-jdk -y
java -version
🐳 Install Docker
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
docker --version
📦 Run Nexus Container
sudo docker run -d \
--name nexus \
-p 8081:8081 \
-v nexus-data:/nexus-data \
sonatype/nexus3

Check running container:

sudo docker ps
🌐 Access Nexus in Browser

Open:

http://<AWS-Public-IP>:8081

Ensure port 8081 is allowed in AWS Security Group.

🔑 Get Nexus Username & Password
Username
admin
Password
sudo docker exec -it nexus cat /nexus-data/admin.password

Login to Nexus UI using the above credentials.

🗂 Create Repositories in Nexus
1️⃣ Snapshot Repository

Used for development builds.

Configuration
Type: maven2 (hosted)
Name: company-snapshot
Version Policy: Snapshot
Deployment Policy: Allow redeploy

Example Version:

<version>1.0.0-SNAPSHOT</version>
2️⃣ Release Repository

Used for production-ready stable builds.

Configuration
Type: maven2 (hosted)
Name: company-release
Version Policy: Release
Deployment Policy: Allow redeploy

Example Version:

<version>1.0.0</version>
📦 Install Maven
sudo apt install maven -y
mvn -version
🧱 Create Maven Project
mvn archetype:generate
cd myapp
⚙️ Configure pom.xml

Add below section inside <project>:

<distributionManagement>
    <repository>
        <id>nexus</id>
        <url>http://localhost:8081/repository/company-release/</url>
    </repository>

    <snapshotRepository>
        <id>nexus</id>
        <url>http://localhost:8081/repository/company-snapshot/</url>
    </snapshotRepository>
</distributionManagement>
🔐 Configure settings.xml

Create file:

mkdir -p ~/.m2
nano ~/.m2/settings.xml

Paste:

<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>YourPassword</password>
    </server>
  </servers>
</settings>
🚀 Deploy Snapshot Artifact

Set version in pom.xml

<version>1.0.0-SNAPSHOT</version>

Run:

mvn clean deploy

Artifact uploaded to:

company-snapshot
🚀 Deploy Release Artifact

Change version:

<version>1.0.0</version>

Run:

mvn clean deploy

Artifact uploaded to:

company-release
✅ Verify in Nexus

Go to:

Browse

Verify:

company-snapshot
Timestamped JAR file
POM file
maven-metadata.xml
company-release
myapp-1.0.0.jar
myapp-1.0.0.pom
🐞 Issues Faced and How I Solved Them
1. Snapshot Deploy Failed
Error
Could not find artifact myapp:myapp:pom:1.0.0-<timestamp>
Cause

Snapshot repository was created with incorrect configuration.

Solution

Deleted and recreated repository with:

Version Policy = Snapshot
Deployment Policy = Allow redeploy
Result

Deployment successful.

2. Release Deploy Failed
Error
Could not find artifact myapp:myapp:pom:1.0.0
Cause

Release repository had incorrect settings.

Solution

Deleted and recreated repository with:

Version Policy = Release
Deployment Policy = Allow redeploy
Result

Deployment successful.

3. Missing settings.xml
Error
/home/ubuntu/.m2/settings.xml: No such file or directory
Cause

Maven config was missing for ubuntu user.

Solution

Created:

mkdir -p ~/.m2
nano ~/.m2/settings.xml

Added credentials.

Result

Authentication successful.

4. Failed to Delete target Directory
Error
Failed to delete target/test-classes
Cause

Previous build executed as root user.

Solution
sudo chown -R ubuntu:ubuntu ~/myapp
rm -rf ~/myapp/target
Result

Build successful.

📚 Topics Covered
What is Nexus
Why Nexus is used
GitHub vs Nexus
AWS EC2 Ubuntu 22.04 Setup
Security Group Configuration
SSH Connection
Linux Updates
Java Installation
Docker Installation
Nexus Docker Setup
Nexus Login
Repository Creation
Maven Installation
Maven Project Setup
pom.xml Configuration
settings.xml Configuration
Snapshot Deployment
Release Deployment
Verification in Nexus
Troubleshooting Real Errors
🎯 Conclusion

This project provided complete hands-on experience with Sonatype Nexus Repository in a real-world environment using AWS Ubuntu Server.

I learned how to install, configure, deploy artifacts, troubleshoot issues, and manage repositories, which are essential skills for DevOps, CI/CD, and Build Release Engineer roles.
