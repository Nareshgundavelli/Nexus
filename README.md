Sonatype Nexus Project Runbook
Project Overview

This project demonstrates a complete hands-on setup of Sonatype Nexus Repository on Ubuntu and its integration with a Maven application.

The objective of this project was to understand how organizations store and manage build artifacts such as JAR and WAR files using Nexus Repository.

During this project, I installed Nexus, created repositories, configured Maven, deployed Snapshot and Release artifacts, and resolved real-time issues.

Tools & Technologies Used
Ubuntu Linux
Java (OpenJDK 17)
Apache Maven
Docker
Sonatype Nexus Repository 3
Git & GitHub
What is Nexus?

Sonatype Nexus Repository is an artifact repository manager.

It is used to store and manage:

JAR files
WAR files
ZIP files
Docker images
Shared internal libraries
Difference Between GitHub and Nexus
GitHub stores source code
Nexus stores build artifacts
Project Architecture

Developer writes code → Pushes to GitHub → Maven builds the project → Artifact stored in Nexus → Deployment server uses artifact

Implementation Steps
1. Update Ubuntu Server
sudo apt update && sudo apt upgrade -y
2. Install Java
sudo apt install openjdk-17-jdk -y
java -version
3. Install Docker
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
4. Run Nexus Using Docker
sudo docker run -d \
--name nexus \
-p 8081:8081 \
-v nexus-data:/nexus-data \
sonatype/nexus3
5. Get Nexus Admin Password
sudo docker exec -it nexus cat /nexus-data/admin.password

Access Nexus UI:

http://<server-ip>:8081
Repositories Created in Nexus
Snapshot Repository

Used for development builds.

Name: company-snapshot
Type: Maven2 (hosted)
Version Policy: Snapshot
Deployment Policy: Allow redeploy

Example:

<version>1.0.0-SNAPSHOT</version>
Release Repository

Used for stable production builds.

Name: company-release
Type: Maven2 (hosted)
Version Policy: Release
Deployment Policy: Allow redeploy

Example:

<version>1.0.0</version>
Maven Configuration
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
      <password>YourPassword</password>
    </server>
  </servers>
</settings>
Deploy Snapshot Artifact

Update version:

<version>1.0.0-SNAPSHOT</version>

Run:

mvn clean deploy

Artifact uploaded to:

company-snapshot
Deploy Release Artifact

Update version:

<version>1.0.0</version>

Run:

mvn clean deploy

Artifact uploaded to:

company-release
Validation

In Nexus UI → Browse

Verify:

Snapshot Repo Contains
Timestamped JAR
POM file
maven-metadata.xml
Release Repo Contains
myapp-1.0.0.jar
myapp-1.0.0.pom
Real-Time Errors Faced and Resolved
1. Snapshot Deployment Failed
Error
Could not find artifact myapp:myapp:pom:1.0.0-<timestamp>
Cause

Snapshot repository was created with incorrect settings.

Resolution

Deleted and recreated repository with:

Version Policy = Snapshot
Deployment Policy = Allow redeploy
Result

Deployment successful.

2. Release Deployment Failed
Error
Could not find artifact myapp:myapp:pom:1.0.0
Cause

Release repository had incorrect configuration.

Resolution

Deleted and recreated repository with:

Version Policy = Release
Deployment Policy = Allow redeploy
Result

Deployment successful.

3. Missing settings.xml
Error
/home/ubuntu/.m2/settings.xml: No such file or directory
Cause

Credentials were configured for root user, not ubuntu user.

Resolution

Created:

~/.m2/settings.xml

Added Nexus credentials.

Result

Authentication successful.

4. Failed to Delete target Directory
Error
Failed to delete target/test-classes
Cause

Previous build executed as root user.

Resolution
sudo chown -R ubuntu:ubuntu ~/myapp
rm -rf ~/myapp/target
Result

Build successful.

Key Learnings
Installed Nexus on Ubuntu
Ran Nexus using Docker
Created Snapshot and Release repositories
Integrated Maven with Nexus
Deployed artifacts successfully
Solved real-time deployment issues
Understood repository policies and permissions
Real-World Usage

In organizations:

Developers upload Snapshot builds regularly
QA team tests latest builds
Release team uses stable versions
Jenkins automates deployments to Nexus

Conclusion

This project helped me understand how real organizations manage build artifacts using Nexus Repository.

It provided both implementation and troubleshooting experience, which is valuable for DevOps and Build/Release Engineer roles.
