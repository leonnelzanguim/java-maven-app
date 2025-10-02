# 📄 README – CI/CD Pipeline with Jenkins, Docker, and AWS EC2

## 🚀 Overview

This project demonstrates a **Jenkins CI/CD pipeline** that automates the build, versioning, containerization, and deployment of a **Java Maven application** to an **AWS EC2 instance** using Docker and Docker Compose.

The pipeline is powered by a **Jenkins Shared Library**, which contains reusable functions such as `buildJar()`, `buildImage()`, `dockerLogin()`, and `dockerPush()`.

---

## ⚙️ Pipeline Stages

### 1. **Build App**

- Runs Maven to compile the Java application and generate a JAR.
- Uses a shared library function `buildJar()` for consistency across pipelines.

```groovy
stage('build app') {
    steps {
        script {
            echo 'building application jar...'
            buildJar()
        }
    }
}
```

---

### 2. **Increment Version**

- Automatically increments the application version using the **Maven Build Helper Plugin** and `versions:set`.
- Extracts the new version from `pom.xml`.
- Builds a Docker image tag using the **app version + Jenkins build number**.
- Example image tag:
  ```
  10012975/demo-app:1.2.4-15
  ```

```groovy
sh 'mvn build-helper:parse-version versions:set -DnewVersion=\\${parsedVersion.majorVersion}.\\${parsedVersion.minorVersion}.\\${parsedVersion.nextIncrementalVersion} versions:commit'
```

---

### 3. **Build Image**

- Builds a Docker image for the application.
- Logs into Docker Hub using stored credentials.
- Pushes the image to Docker Hub.

```groovy
buildImage(env.IMAGE_NAME)
dockerLogin()
dockerPush(env.IMAGE_NAME)
```

---

### 4. **Deploy**

- Copies deployment scripts (`server-cmds.sh`, `docker-compose.yaml`) to the **AWS EC2 instance** using `scp`.
- Executes the deployment script remotely via `ssh`.
- Runs the containerized application on EC2, exposing it on **port 3080**.

```groovy
sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
```

---

### 5. **Commit Version Update**

- Updates the Git repository with the new version in `pom.xml`.
- Configures Git author as Jenkins.
- Commits the change with message `"ci: version bump"`.
- Pushes to the branch **`jenkins-jobs`** on GitHub using stored credentials.

```groovy
sh 'git add .'
sh 'git commit -m "ci: version bump"'
sh 'git push origin HEAD:jenkins-jobs'
```

---

## 🔑 Features

- **Automated build and versioning** with Maven.
- **Docker image creation and push** to Docker Hub.
- **Secure deployment** to AWS EC2 via SSH Agent.
- **Version control update**: commits new versions back to GitHub.
- **Reusable Jenkins Shared Library** for clean and maintainable pipelines.

---

## 🛠️ Requirements

- Jenkins with:
  - **Pipeline Plugin**
  - **Pipeline: Shared Groovy Libraries Plugin**
  - **SSH Agent Plugin**
- Docker & Docker Compose installed on EC2 instance.
- GitHub repository for application source code.
- Docker Hub account for hosting images.
- AWS EC2 instance (Amazon Linux 2 recommended).

---

## ✅ Workflow Summary

1. Jenkins builds the Java JAR.
2. Pipeline increments the version and generates a new Docker image tag.
3. Docker image is built and pushed to Docker Hub.
4. EC2 instance receives deployment scripts and runs the container.
5. Jenkins commits the new version back to GitHub for traceability.
