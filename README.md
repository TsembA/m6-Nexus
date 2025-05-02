
# 🚀 DevOps Learning Notes

This document covers core DevOps tools and practices including Git, Build Tools, Artifact Repositories, and Cloud Infrastructure.

---

## 1. 🧠 Version Control Basics (Git)

**Also known as "source control"**  
Git is the most widely used version control system. It allows teams to collaborate on code by tracking and managing changes.

### ✅ Core Concepts:
- Keep a history of code changes
- Track all files and updates
- Revert to previous versions easily
- Each update is saved with a **commit message**
- Everyone has a full local copy of the code

### 🔗 Popular Git Platforms:
- GitHub
- GitLab
- Bitbucket
- Self-hosted company Git servers

### ⚙️ Common Git Commands

```bash
git clone <repo>                   # Clone remote repo
git add <file>                     # Stage file
git commit -m "msg"                # Commit changes
git push                           # Push to remote
git pull                           # Pull latest changes
git pull --rebase                  # Rebase local changes
git init                           # Initialize repo
git checkout <branch>              # Switch branch
git checkout -b <branch>           # Create new branch
git branch                         # List branches
git status                         # Check current status
git branch -d <branch>             # Delete branch
git merge <branch>                 # Merge branch
git rebase <branch>                # Rebase changes
git stash                          # Temporarily save changes
git stash pop                      # Reapply stashed changes
git reset --hard HEAD~1            # Discard last commit
git revert <commit>                # Create revert commit
git rm --cached <file>             # Remove file from repo only
git commit --amend                 # Edit last commit
git push --force                   # Force push changes
```

### 🧑‍🏫 Best Practices
- Don’t push directly to `main/master`
- Use feature/bugfix branches (`feature/xyz`, `bugfix/xyz`)
- Use `.gitignore` to exclude unnecessary files (e.g., `node_modules/`, `.env`)
- Commit small, related changes
- Write clear and meaningful commit messages
- Perform code reviews via Pull/Merge Requests
- Regularly pull updates from the remote branch

---

## 2. 📦 Build and Package Manager Tools

When preparing apps for production, you often **build the code** into a single **artifact** using a build tool.

### 🔍 What is an Artifact?
A moveable file that contains the application and its dependencies (e.g., `.jar`, `.whl`, Docker image).

### 🏗️ What does “Building the Code” mean?
- Compiling source code
- Compressing files
- Packaging into a single deployable artifact

### 🗂️ What is an Artifact Repository?
Storage for artifacts like `.jar`, `.whl`, or Docker images. Common options:
- DockerHub
- Nexus
- JFrog Artifactory
- AWS ECR

### 🔧 Common Build Tools:
| Language     | Build Tools             |
|--------------|--------------------------|
| Java         | Maven, Gradle            |
| JavaScript   | npm, yarn, webpack       |
| Python       | pip                      |
| C/C++        | Conan                    |
| C#           | NuGet                    |
| Go           | Go modules (`go mod`)    |
| Ruby         | RubyGems                 |

### 🐳 Why Docker?
- Universally portable artifact
- Run the same image in dev, test, and prod
- Fewer artifacts to manage

---

## 3. ☁️ Cloud & IaaS Basics with DigitalOcean

Cloud providers offer infrastructure as a service (IaaS) to host virtual resources like VMs, databases, and containers.

### 🧩 What is DigitalOcean?
- Simple IaaS platform
- Developer-friendly
- Quick provisioning of **Droplets** (VMs)
- Pre-installed images (e.g., Docker)

### 🔑 Easy Setup:
1. Choose an OS or image (e.g., Docker, Ubuntu)
2. Select VM size (RAM, CPU)
3. Add SSH key for secure access

### 🛠️ Use Cases:
- Run apps in containers (e.g., Jenkins in Docker)
- Host artifact repositories like Nexus

### ☁️ Other IaaS Providers:
- Amazon Web Services (AWS)
- Microsoft Azure
- Google Cloud Platform (GCP)
- Linode, Hetzner, Vultr

Each provider has its own set of tools, CLI utilities, and pricing models.

---

## 4. 🗄️ Artifact Repository Manager with Nexus

### ✅ What is Nexus?
A central place to manage and store build artifacts (e.g., Maven packages, npm packages, Docker images).

### ⭐ Features:
- Acts as **proxy** for external repos (Maven Central, npmjs)
- Speeds up builds via local caching
- REST API support for automation
- Role-based access and security
- Docker & Kubernetes compatible

### 🔌 Nexus REST API Examples:

```bash
# List repositories
curl -u user:pwd -X GET 'http://<host>:8081/service/rest/v1/repositories'

# List components in a repo
curl -u user:pwd -X GET 'http://<host>:8081/service/rest/v1/components?repository=<repo_name>'

# List assets of a component
curl -u user:pwd -X GET 'http://<host>:8081/service/rest/v1/components/<ID>'
```

### 🛠️ Manual Installation Steps

```bash
# 1. Provision a server (2GB+ RAM)
# 2. Open ports 8081 and 8085
# 3. Install Java
# 4. Download & extract Nexus
mkdir /opt && cd /opt
wget https://download.sonatype.com/nexus/3/nexus-<version>.tar.gz
tar -zxvf nexus-<version>.tar.gz

# 5. Create user and set permissions
adduser nexus
chown -R nexus:nexus nexus-<version> sonatype-work

# 6. Edit nexus.rc to run as nexus user
# 7. Start Nexus
su - nexus
./nexus/bin/nexus start
```

Access UI at: `http://<server-ip>:8081`

---

### 🐳 Run Nexus with Docker

```bash
docker volume create --name nexus-data

docker run -d -p 8081:8081 --name nexus   -v nexus-data:/nexus-data sonatype/nexus3

docker exec nexus cat /nexus-data/admin.password
```

---

### 📤 Publish Gradle Artifact to Nexus

1. Set up Nexus user with permissions
2. Add the following to `build.gradle`:

```groovy
publishing {
  publications {
    maven(MavenPublication) {
      artifact("build/libs/my-app-${version}.jar") {
        extension 'jar'
      }
    }
  }
  repositories {
    maven {
      name 'nexus'
      url "http://<ip>:8081/repository/maven-snapshots/"
      allowInsecureProtocol = true
      credentials {
        username project.repoUser
        password project.repoPassword
      }
    }
  }
}
```

3. Run:

```bash
./gradlew build
./gradlew publish
```

---

### 📤 Publish Maven Artifact to Nexus

1. Add plugin to `pom.xml`:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-deploy-plugin</artifactId>
  <version>3.1.1</version>
</plugin>
```

2. Define repository:

```xml
<distributionManagement>
  <snapshotRepository>
    <id>nexus-snapshots</id>
    <url>http://<ip>:8081/repository/maven-snapshots/</url>
  </snapshotRepository>
</distributionManagement>
```

3. Configure `~/.m2/settings.xml`:

```xml
<settings>
  <servers>
    <server>
      <id>nexus-snapshots</id>
      <username>your-username</username>
      <password>your-password</password>
    </server>
  </servers>
</settings>
```

4. Run Maven commands:

```bash
mvn package
mvn deploy
```

---
