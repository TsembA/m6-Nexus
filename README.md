

### 6. Artifact Repository Manager with Nexus

Nexus supports various types of artifacts, including Docker images, npm packages, and Maven packages, making it a solid choice for an artifact repository. It is easy to set up and use.

One of Nexus's key features is its ability to act as a PROXY FOR OTHER REPOSITORIES, such as npmjs and Maven Central. This allows you to use your Nexus repository as an intermediary cache. When you fetch a package from an external repository, it gets stored in Nexus, improving performance and reliability for future downloads.

Nexus also offers a REST API, enabling automation for pushing and pulling artifacts. This is particularly useful for integrating Nexus into CI/CD pipelines.

Additionally, Nexus allow you to create multiple repositories for different teams or users. Access can be restricted using role-based permissions, ensuring security and proper resource management.

Nexus can be deployed inside a Docker container, making it easy to run in containerized environments such as Kubernetes. This allows for better scalability and streamlined deployment.

### Nexus REST API

# List all repositories:
curl -u user:pwd -X GET 'http://{host}:8081/service/rest/v1/repositories'

# List all components in a specific repository:
curl -u user:pwd -X GET 'http://{host}:8081/service/rest/v1/components?repository=<INSERT_REPO_NAME>'

# List all assets for a specific component:
curl -u user:pwd -X GET 'http://{host}:8081/service/rest/v1/components/<ID>'



### The way to install nexus manualy

1. Create the instance with min requirement recomended RAM
2. Open port 8081 and 8085 | You might use 8085 for internal communication.
3. Install java
4. Mkdir /opt/ and download Nexus from official SONATYPE page
5. Untar package tar -zxvf (nexus-31231 folder- contains runtime and app of Nexus | sonatype- contains your own config for Nexus and data. Can be used for backups)
6. adduser nexus| chown -R nexus:nexus <folder name>| Create nexus user and give acces to those folders 
7. Configure nex-12351/bin/nexus.rc by adding nexus user
8. Swithc to user su - nexus and start Nexus | /opt/nex-1234/bin/nexus start
9. Go to port <IP>8081

### Set up Nexus with Docker

1. docker volume create --name nexus-data | Create volume to keep data presistent
2. docker run -d -p 8081:8081 --name nexus -v nexus-data:/nexus-data sonatype/nexus3 | run docker container
3. docker container exec nexus cat nexus-data/admin.password | Get the pwd to be able to login into nexus’s admin panel

### Publish Gradle-Java app artifact in to the repopository 
1. Create role to give user permissions and access to the repo
2. Assing the role to your user
3. Add publish comand logic and repository url to your build.gradle file

```groovy
publishing {
    publications {
        maven(MavenPublication) {
            artifact("build/libs/my-app-$version"+".jar"){
                extension 'jar'
            }
        }
    }
}
repositories {
    maven {
        name 'nexus'
        url "http://xxxxxx:8081/repository/maven-snapshots/" 
        allowInsecureProtocol = true
        credentials {
            username project.repoUser 
            password project.repoPassword
        }
    }
}

``` 
4. Run gradle build and gradle publish command to deploy artifact to the repo


### Publish Java-Maven app artifact in to the repopository

1. Create role to give user permissions and access to the repo
2. Assing the role to your user
3. Add plugin in to pom.xml file which will allow maven deploy JAR to Nexus
``
   <plugin> 
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-deploy-plugin</artifactId>
     <version>3.1.1</version>
   </plugin>

   <plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-deploy-plugin</artifactId>
   </plugin>

4. Configure location of the Nexus repo

    <distributionManagement>
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <url>http://xxxxxxxxx:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
``   
5. Configure local credentials for the Maven in ~/.m2/settings.xml
``
<settings>
    <servers>
        <server>
            <id>nexus-snapshots</id>
            <username>maven</username>
            <password>xxxxx</password>
        </server>
    </servers>
</settings>
``

6. Run mvn package command to build the artifact
7. Run mvn deploy to deploy artifact to the Nexus repo
