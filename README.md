# Sonatype Nexus Setup & Maven Artifact Deployment

## Overview

This project demonstrates a complete hands-on setup of Sonatype Nexus Repository on Ubuntu, integration with Maven, and deployment of both Snapshot and Release artifacts.

The main objective of this project was to understand how organizations store and manage build artifacts such as JAR and WAR files using Nexus Repository.

During this implementation, I installed Nexus, created repositories, configured Maven, deployed artifacts, and resolved real-time issues.

---

# Objectives

- Install and run Nexus using Docker on Ubuntu
- Configure Maven credentials using `settings.xml`
- Create Snapshot and Release repositories in Nexus
- Build a Java Maven application
- Deploy Snapshot and Release artifacts to Nexus
- Troubleshoot real-time deployment issues

---

# Tech Stack

- Ubuntu Linux
- Java (OpenJDK 17)
- Apache Maven
- Docker
- Sonatype Nexus Repository 3
- Git & GitHub

---

# Project Architecture

Developer -> GitHub -> Maven Build -> Nexus Repository -> Deployment Server

---

# Installation Steps

## 1. Update Ubuntu Server

```bash
sudo apt update && sudo apt upgrade -y
2. Install Java
sudo apt install openjdk-17-jdk -y
java -version
3. Install Docker
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
docker --version
4. Run Nexus Using Docker
sudo docker run -d \
--name nexus \
-p 8081:8081 \
-v nexus-data:/nexus-data \
sonatype/nexus3
5. Get Nexus Admin Password
sudo docker exec -it nexus cat /nexus-data/admin.password

Access UI:

http://<server-ip>:8081
Repository Configuration

Create the following repositories in Nexus:

Snapshot Repository

Used for development builds.

Type: maven2 (hosted)
Name: company-snapshot
Version Policy: Snapshot
Deployment Policy: Allow redeploy

Example:

<version>1.0.0-SNAPSHOT</version>
Release Repository

Used for stable production builds.

Type: maven2 (hosted)
Name: company-release
Version Policy: Release
Deployment Policy: Allow redeploy

Example:

<version>1.0.0</version>
Maven Project Configuration
pom.xml
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
settings.xml

Location:

~/.m2/settings.xml
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>admin</username>
      <password>YOUR_PASSWORD</password>
    </server>
  </servers>
</settings>
Build and Deploy
Snapshot Deployment

Set version in pom.xml

<version>1.0.0-SNAPSHOT</version>

Run:

mvn clean deploy

Result:

Artifact uploaded to company-snapshot

Release Deployment

Change version:

<version>1.0.0</version>

Run:

mvn clean deploy

Result:

Artifact uploaded to company-release

Validation

In Nexus UI -> Browse

Verify:

company-snapshot
Timestamped JAR file
POM file
maven-metadata.xml
company-release
myapp-1.0.0.jar
myapp-1.0.0.pom
Errors Faced During Implementation and Resolutions
1. Snapshot Deployment Failed
Error
Could not find artifact myapp:myapp:pom:1.0.0-<timestamp>
Root Cause

Snapshot repository was created with incorrect settings.

Resolution

Recreated repository with:

Version Policy: Snapshot
Deployment Policy: Allow redeploy
Result

Snapshot deployment completed successfully.

2. Release Deployment Failed
Error
Could not find artifact myapp:myapp:pom:1.0.0
Root Cause

Release repository had incorrect settings.

Resolution

Recreated repository with:

Version Policy: Release
Deployment Policy: Allow redeploy
Result

Release deployment completed successfully.

3. Missing settings.xml
Error
/home/ubuntu/.m2/settings.xml: No such file or directory
Root Cause

Credentials were configured for root user, but build was executed using ubuntu user.

Resolution

Created:

~/.m2/settings.xml

Added Nexus credentials.

Result

Authentication successful.

4. Failed to Delete target Directory
Error
Failed to delete target/test-classes
Root Cause

Previous build was executed as root user.

Resolution
sudo chown -R ubuntu:ubuntu ~/myapp
rm -rf ~/myapp/target
Result

Build completed successfully.

Key Learnings
Installed Nexus on Ubuntu
Ran Nexus using Docker
Created Snapshot and Release repositories
Integrated Maven with Nexus
Deployed artifacts successfully
Solved real-time deployment issues
Learned repository policies and permissions
Real-World Usage
Developers publish Snapshot builds regularly
QA team tests latest builds
Release team uses stable versions
Jenkins automates deployment to Nexus
Resume Points
Configured Sonatype Nexus Repository on Ubuntu using Docker
Created and managed Snapshot and Release repositories
Integrated Maven builds with Nexus credentials
Deployed Java artifacts to internal repositories
Troubleshot Maven deployment and Linux permission issues
Gained hands-on experience in artifact management
Future Enhancements
Integrate Jenkins Pipeline
Host Docker Images in Nexus
Configure Role-Based Access Control
Use Proxy Repositories
Implement Backup Strategy
Conclusion

This project provided real-time hands-on experience in artifact management using Sonatype Nexus Repository.

It helped me understand both implementation and troubleshooting, which is highly valuable for DevOps, CI/CD, and Build Engineer roles.
