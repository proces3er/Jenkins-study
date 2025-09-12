# Jenkins-study



# Welcome to Jenkins

![Jenkins Logo](https://www.jenkins.io/images/logos/sydney/sydney.png)

---

## What is Jenkins?

Jenkins is a name associated with classic British butlers – a helpful domestic worker ready to execute any command.  

Jenkins is an **open source**, automation **server** that enables **Continuous Integration (CI)** and **Continuous Deployment (CD)** by automatically **building, testing, and deploying** code **on demand**.  

Our Jenkins server is hosted **locally on a virtual machine**, running **offline (on-premise)** without direct internet access.

---

## What Problems Jenkins Solves

- **Monitor Repository**  
  - Automatically fetch code when changes occur:
    - **Poll SCM**: time-driven triggers (e.g., every 5 minutes).  
    - **Webhook**: event-driven triggers (when commits are pushed).  

- **Automating Builds**  
  - Jenkins can compile/build the code every time it changes.

- **Automating Testing**  
  - Automatically run unit/integration tests on every commit → catch bugs early.

- **Continuous Integration (CI)**  
  - Merge changes, build, and test continuously → reduce integration conflicts.

- **Continuous Deployment (CD)**  
  - Deploy to a staging environment automatically after successful tests.

- **Cross-tool Integration**  
  - Jenkins integrates with many build, test, and deployment tools through plugins.

- **Consistent, Repeatable Process**  
  - All steps are executed the same way every time, reducing human error.

---

## Why Jenkins?

**Cons of manual work without Jenkins:**
- Integration failures due to late merges.  
- Code conflicts that are not caught early.  
- Dependency/version mismatches across environments.  

**With Jenkins:**
- Automated workflows.  
- Faster feedback.  
- Reliable, consistent builds.  
- Centralised logging and monitoring.  

---

## Jenkins Setup (Offline, On-Prem VM)

Since our Jenkins instance is **offline**, we need to prepare plugins and dependencies manually:

1. **Install Jenkins (on VM):**
   - Download `.war` or native packages (e.g., `.deb`, `.rpm`) from [jenkins.io](https://www.jenkins.io).
   - Run Jenkins with:
     ```bash
     java -jar jenkins.war --httpPort=8080
     ```

2. **Access Jenkins:**
   - Open browser → `http://<VM-IP>:8080`.
   - Unlock Jenkins with the initial admin password located in:
     ```
     /var/lib/jenkins/secrets/initialAdminPassword
     ```

3. **Plugin Installation (Offline):**
   - Go to [Jenkins Plugin Index](https://plugins.jenkins.io/) (on a machine with internet).
   - Download `.hpi` or `.jpi` plugin files.  
   - Transfer plugins to Jenkins VM.  
   - Place them in:
     ```
     /var/lib/jenkins/plugins/
     ```
   - Restart Jenkins:
     ```bash
     sudo systemctl restart jenkins
     ```

4. **Language-Specific Setup:**
   - For **Java/Maven**: install `Maven Integration Plugin`. Configure Maven installation under *Manage Jenkins → Global Tool Configuration*.  
   - For **Python**: install `ShiningPanda Plugin` (or manually configure Python toolchain).  
   - For **.NET**: use `MSBuild Plugin`.  
   - Each requires downloading the `.hpi` manually (due to offline setup) and uploading via Jenkins UI or filesystem.

---

## Example: Jenkins Pipeline (Jenkinsfile) – .NET Project

This pipeline builds, tests, and deploys a .NET application.  
The VM running Jenkins must have the **.NET SDK** installed (e.g., via `apt`, `yum`, or manual download).  

```groovy
pipeline {
    agent any

    tools {
        // optional: if using MSBuild plugin for older .NET Framework
        // msbuild 'MSBuild-16.0'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'http://<your-git-server>/your-dotnet-repo.git'
            }
        }

        stage('Restore Dependencies') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test --no-build --configuration Release'
            }
        }

        stage('Publish') {
            steps {
                sh 'dotnet publish -c Release -o ./publish'
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo 'Deploying .NET app to staging server...'
                // Example deployment step (adjust paths/servers as needed)
                sh 'scp -r ./publish/* user@staging-server:/var/www/my-dotnet-app/'
            }
        }
    }

    post {
        always {
            junit '**/TestResults/*.xml'   // capture test results if available
            archiveArtifacts artifacts: 'publish/**', fingerprint: true
        }
    }
}
