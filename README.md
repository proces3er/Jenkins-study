# Jenkins-study



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


## Pipeline Stage Overview

- **Checkout**  
  Pulls source code from the Git repository.  

- **Restore Dependencies**  
  Downloads NuGet packages required for the build.  

- **Build**  
  Compiles the application in `Release` mode.  

- **Test**  
  Runs unit and/or integration tests to validate code.  

- **Publish**  
  Prepares build output for deployment (publishes to a `./publish` folder).  

- **Deploy to Staging**  
  Copies the published files to the staging server (example uses `scp`).  

- **Post Section**  
  Always runs after the pipeline (success or failure).  
  - Collects and publishes test results in Jenkins UI.  
  - Archives build artifacts for later use/retrieval.  

```groovy 
pipeline {
    agent any   // Run this pipeline on any available Jenkins agent (or the master node if no agents exist)

    tools {
        // Define external build tools if needed (example: MSBuild for .NET Framework projects)
        // msbuild 'MSBuild-16.0'
    }

    stages {
        stage('Checkout') {
            steps {
                // Pull code from your Git repository (replace with your on-prem Git server)
                git branch: 'main', url: 'http://<your-git-server>/your-dotnet-repo.git'
            }
        }

        stage('Restore Dependencies') {
            steps {
                // Restore NuGet packages / dependencies from your project configuration
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                // Compile the .NET project in Release configuration
                sh 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                // Run unit tests (without rebuilding since we already did that in the Build stage)
                sh 'dotnet test --no-build --configuration Release'
            }
        }

        stage('Publish') {
            steps {
                // Package the application for deployment
                // Output is placed into a ./publish folder
                sh 'dotnet publish -c Release -o ./publish'
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo 'Deploying .NET app to staging server...'
                // Copy the published files to the staging environment (via SCP in this example)
                // Adjust path, server, and credentials to match your infrastructure
                sh 'scp -r ./publish/* user@staging-server:/var/www/my-dotnet-app/'
            }
        }
    }

    post {
        always {
            // Capture and store test results so they show up in Jenkins UI
            junit '**/TestResults/*.xml'

            // Archive the published output files as build artifacts
            // Jenkins keeps these for later retrieval/verification
            archiveArtifacts artifacts: 'publish/**', fingerprint: true
        }
    }
}


h1. Jenkins Study

!https://www.jenkins.io/images/logos/sydney/sydney.png|width=200!

---

h2. What is Jenkins?

Jenkins is a name associated with classic British butlers – a helpful domestic worker ready to execute any command.  

Jenkins is an *open source*, automation *server* that enables *Continuous Integration (CI)* and *Continuous Deployment (CD)* by automatically *building, testing, and deploying* code *on demand*.  

Our Jenkins server is hosted *locally on a virtual machine*, running *offline (on-premise)* without direct internet access.

---

h2. What Problems Jenkins Solves

* *Monitor Repository*
** Automatically fetch code when changes occur:
*** Poll SCM: time-driven triggers (e.g., every 5 minutes)
*** Webhook: event-driven triggers (when commits are pushed)
* *Automating Builds*
** Jenkins can compile/build the code every time it changes
* *Automating Testing*
** Automatically run unit/integration tests on every commit → catch bugs early
* *Continuous Integration (CI)*
** Merge changes, build, and test continuously → reduce integration conflicts
* *Continuous Deployment (CD)*
** Deploy to a staging environment automatically after successful tests
* *Cross-tool Integration*
** Jenkins integrates with many build, test, and deployment tools through plugins
* *Consistent, Repeatable Process*
** All steps are executed the same way every time, reducing human error

---

h2. Why Jenkins?

*Cons of manual work without Jenkins:*
* Integration failures due to late merges
* Code conflicts that are not caught early
* Dependency/version mismatches across environments

*With Jenkins:*
* Automated workflows
* Faster feedback
* Reliable, consistent builds
* Centralised logging and monitoring

---

h2. Jenkins Setup (Offline, On-Prem VM)

Since our Jenkins instance is *offline*, we need to prepare plugins and dependencies manually:

h3. 1. Install Jenkins (on VM)
* Download `.war` or native packages (e.g., `.deb`, `.rpm`) from [jenkins.io|https://www.jenkins.io]
* Run Jenkins:
{code:bash}
java -jar jenkins.war --httpPort=8080
{code}

h3. 2. Access Jenkins
* Open browser → `http://<VM-IP>:8080`
* Unlock Jenkins with the initial admin password located at:
{code}
 /var/lib/jenkins/secrets/initialAdminPassword
{code}

h3. 3. Plugin Installation (Offline)
* On a machine with internet, download `.hpi` or `.jpi` plugin files from the Jenkins Plugin Index: [https://plugins.jenkins.io/]
* Transfer plugins to the Jenkins VM and place them in:
{code}
 /var/lib/jenkins/plugins/
{code}
* Restart Jenkins:
{code:bash}
sudo systemctl restart jenkins
{code}

h3. 4. Language-Specific Setup
* Java/Maven: install *Maven Integration Plugin*. Configure Maven under *Manage Jenkins → Global Tool Configuration*
* Python: install *ShiningPanda Plugin* or manually configure Python toolchain
* .NET: install *MSBuild Plugin*. Offline install via `.hpi` is required

---

h2. Example: Jenkins Pipeline (Jenkinsfile) – .NET Project

This pipeline builds, tests, and deploys a .NET application. The VM running Jenkins must have the *.NET SDK* installed.

---

h2. Pipeline Stage Overview

* *Checkout* – Pulls source code from the Git repository
* *Restore Dependencies* – Downloads NuGet packages required for the build
* *Build* – Compiles the application in `Release` mode
* *Test* – Runs unit/integration tests to validate code
* *Publish* – Prepares build output for deployment (published to `./publish`)
* *Deploy to Staging* – Copies the published files to the staging server (example uses `scp`)
* *Post Section* – Always runs after pipeline (success/failure)
** Collects and publishes test results in Jenkins UI
** Archives build artifacts for later use/retrieval

---

h2. Jenkinsfile – .NET Pipeline Example

{code:groovy}
pipeline {
    agent any   // Run this pipeline on any available Jenkins agent

    tools {
        // Optional MSBuild tool for .NET Framework projects
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
                sh 'scp -r ./publish/* user@staging-server:/var/www/my-dotnet-app/'
            }
        }
    }

    post {
        always {
            junit '**/TestResults/*.xml'
            archiveArtifacts artifacts: 'publish/**', fingerprint: true
        }
    }
}
{code}

