
# 🔑 Artifact Repository Manager with Nexus

Nexus is a widely used repository manager for storing and managing artifacts such as Docker images, npm packages, and Maven packages. It acts as an intermediary between developers and the central repositories, improving performance and reliability.

---

## ✅ What is Nexus?

- A repository manager for storing artifacts.
- Acts as a **proxy** for external repositories like Maven Central or npmjs.
- Stores your build artifacts locally, speeding up access and caching dependencies.

### ⭐ Features:
- Supports multiple types of artifacts (Docker, npm, Maven, etc.)
- REST API for automation (ideal for CI/CD)
- Role-based access control for secure repository management
- Compatible with Docker, Kubernetes, and containerized environments

---

## 🔌 Nexus REST API Examples

Here are some basic REST API commands to interact with Nexus:

### List all repositories:

```bash
curl -u user:pwd -X GET 'http://<host>:8081/service/rest/v1/repositories'
```

### List all components in a specific repository:

```bash
curl -u user:pwd -X GET 'http://<host>:8081/service/rest/v1/components?repository=<repo_name>'
```

### List all assets for a specific component:

```bash
curl -u user:pwd -X GET 'http://<host>:8081/service/rest/v1/components/<ID>'
```

---

## 🛠️ Manual Installation Steps

### Prerequisites:
- A server with at least **2GB RAM** recommended
- Open ports **8081** and **8085** (8085 can be for internal communication)

### Steps:

1. Install Java on the server.
2. Download Nexus from the [official Sonatype page](https://www.sonatype.com/download).
3. Extract Nexus:

```bash
mkdir /opt && cd /opt
wget https://download.sonatype.com/nexus/3/nexus-<version>.tar.gz
tar -zxvf nexus-<version>.tar.gz
```

4. Create a Nexus user and set folder permissions:

```bash
adduser nexus
chown -R nexus:nexus nexus-<version> sonatype-work
```

5. Edit the `nexus.rc` file to specify the Nexus user:

```bash
su - nexus
./nexus/bin/nexus start
```

6. Access Nexus at `http://<server-ip>:8081`

---

## 🐳 Run Nexus with Docker

For containerized deployments, you can run Nexus as a Docker container.

### Steps:

1. Create a persistent Docker volume for Nexus data:

```bash
docker volume create --name nexus-data
```

2. Run the Nexus container:

```bash
docker run -d -p 8081:8081 --name nexus   -v nexus-data:/nexus-data sonatype/nexus3
```

3. Retrieve the admin password:

```bash
docker exec nexus cat /nexus-data/admin.password
```

---

## 📤 Publish Gradle Artifact to Nexus

To publish a Gradle-based Java artifact to Nexus:

1. Set up a Nexus user and assign permissions.
2. Add the following configuration to your `build.gradle`:

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

3. Run the following Gradle commands to build and publish the artifact:

```bash
./gradlew build
./gradlew publish
```

---

## 📤 Publish Maven Artifact to Nexus

To deploy a Maven-based Java artifact to Nexus:

1. Add the Maven deploy plugin in your `pom.xml`:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-deploy-plugin</artifactId>
  <version>3.1.1</version>
</plugin>
```

2. Define the Nexus repository:

```xml
<distributionManagement>
  <snapshotRepository>
    <id>nexus-snapshots</id>
    <url>http://<ip>:8081/repository/maven-snapshots/</url>
  </snapshotRepository>
</distributionManagement>
```

3. Set up local credentials for Maven in `~/.m2/settings.xml`:

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

4. Run Maven commands to build and deploy the artifact:

```bash
mvn package
mvn deploy
```

---

This document outlines basic Nexus setup and how to integrate it into your build pipelines. Whether you're working with Gradle, Maven, or Docker, Nexus makes managing your artifacts easier and more efficient.
