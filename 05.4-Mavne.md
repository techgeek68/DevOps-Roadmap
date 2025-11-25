# Apache Maven

  - Maven is a powerful build automation and project management tool used primarily for Java based projects.
  - It's more than just a build tool; it's a comprehensive project management tool based on the concept of a Project Object Model (POM).
  - Maven follows "convention over configuration." This means it provides sensible default behaviors, so you don't have to configure everything from scratch. A standard project layout is assumed.
  - It enables:
      - Reproducible builds (same inputs → same artifacts).
      - Standard lifecycle phases (validate → test → package → install → deploy).
      - Automatic transitive dependency resolution.
      - Plugin-driven extensibility (compilation, testing, packaging, reporting, site generation).
      - Seamless integration with CI/CD, artifact repositories (Nexus/Artifactory), quality and security scanners.

> Mnemonic: “Maven = Model + Lifecycle + Plugins + Dependencies”

---

**Why Maven?**
 - Before tools like Maven, developers faced:
   - Complex Build Processes: Building a project involved manually compiling, packaging, and managing classpaths.
     
   - Dependency Hell: Managing JAR files and their specific versions was a manual, error-prone process. "Which version of log4j does this project need? Is it on the classpath?"
     
   - Project Standardization: Every project had a different directory structure and build process, making it hard for new developers to onboard.
     
   - Maven solves these by providing a standardized project structure, a declarative way to manage dependencies, and a uniform build interface.

---

**Concepts & Terminology**
 - POM (pom.xml): The heart of every Maven project. It's an XML file that contains information about the project and configuration details used by Maven to build the project (dependencies, plugins, goals, etc.).
 - Coordinates (GAV): A unique identifier for a project/artifact.
   - GroupId: The unique identifier of the organization or group (e.g., com.company.project).
   - ArtifactId: The name of the project/jar (e.g., data-service).
   - Version: The version of the project (e.g., 1.0.0-SNAPSHOT).
    
 - Dependencies: External libraries (JARs) that your project needs to compile, test, or run. Maven automatically downloads them from a central repository.
   
 - Repositories:
   - Local Repository: A directory on your own machine (typically ~/.m2/repository) where Maven stores all downloaded project artifacts and plugins.
   - Central Repository: The default public repository provided by the Maven community where most common libraries are hosted.
   - Remote Repository: A custom, often private, repository (like JFrog Artifactory or Sonatype Nexus) set up by a company to host their own artifacts and proxy the central repo.
     
- Build Lifecycle: A predefined sequence of phases that define the order in which project goals are executed.
- Goals: A specific task that contributes to the building and managing of a project (e.g., mvn compile, mvn test). Goals are provided by Plugins.


---

**Maven Lifecycle (Build Lifecycle)**
  1. validate — Check project structure (pom correctness).
  2. compile — Compile the project's source code.
  3. test — Test the compiled source code using a suitable unit testing framework (e.g., JUnit). These tests should not require the code to be packaged or deployed.
  4. package — Take the compiled code and package it in its distributable format (e.g., JAR, WAR).
  5. verify — Run any checks on the results of integration tests to ensure quality criteria are met.
  6. install —  Install the package into the local repository, so it can be used as a dependency in other projects on the same machine.
  7. deploy — Done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.


Other Lifecycles:
  - Clean lifecycle: pre-clean → clean → post-clean (e.g., mvn clean).
  - Site lifecycle: pre-site → site → post-site → site-deploy.

> Important: Running a later phase implicitly runs all previous phases in the lifecycle (e.g., mvn package triggers validate, compile, test, then package).

---


**Common Maven commands**

```
mvn clean                  # Remove target/ (previous build outputs)
mvn compile                # Compile source
mvn test                   # Run unit tests
mvn package                # Build artifact (JAR/WAR)
mvn verify                 # Run additional verification steps
mvn install                # Install artifact to local ~/.m2/
mvn deploy                 # Deploy artifact to remote repo
mvn dependency:tree        # Show dependency graph
mvn help:effective-pom     # Show merged POM (with parents & profiles)
```

  - Useful flags:
```
mvn -DskipTests package          # Skip test execution (compilation still happens)
mvn -DskipITs verify             # Skip integration tests
mvn -T 1C clean package          # Parallel build: 1 thread per CPU core
mvn -U clean package             # Force update of SNAPSHOT dependencies
mvn -Pprod package               # Activate 'prod' profile
mvn -Denv=prod package           # Property-based activation
```

  - Maven Wrapper:
```
mvn -N io.takari:maven:wrapper   # Generate ./mvnw and Maven wrapper JAR
./mvnw clean package             # Use project-pinned Maven version
```

  - Utility Commands
```
mvn dependency:tree                         # Show dependency graph
mvn help:effective-pom                      # Show merged POM (with parents & profiles)
mvn versions:display-dependency-updates   # Check for dependency updates
mvn versions:display-plugin-updates       # Check for plugin updates
```

---

**Installation & Configuration**
  - These steps assume a Fedora/RHEL/CentOS system using `dnf`. Adjust for your distro as needed.

**Prerequisites: Java (JDK 11+)**
  - Create a VM named `developer` (as per your lab/notes).
  - Install Java and verify version:
```
sudo dnf -y install java-*-openjdk java-*-openjdk-devel
```
```
java -version
```

  - Discover JDK path and set `JAVA_HOME`.
```
cd /usr/lib/jvm
ls
cd java-21-openjdk
pwd
cd
```
  > Example output: /usr/lib/jvm/java-21-openjdk

- Set `JAVA_HOME` system-wide (or use `~/.bashrc` for per-user)
  - `sudo vi /etc/profile`
```
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk
export PATH="$PATH:$JAVA_HOME/bin"
```

  - Apply changes and verify:
```
source /etc/profile
echo $JAVA_HOME
which java
```

**Download and Install Maven**
  - Option A: Package Manager
```
sudo dnf info maven
sudo dnf -y install maven
mvn -version
```

  - Option B: Latest Binary (Recommended)
    - [Visit the website](https://maven.apache.org/index.html) Find a donwload link
```
cd /tmp
sudo wget https://dlcdn.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz
sudo tar -zxvf apache-maven-3.9.11-bin.tar.gz
sudo mv apache-maven-3.9.11 /opt/maven

# Optional: Create a dedicated user
sudo groupadd maven
sudo useradd -g maven -d /opt/maven -s /sbin/nologin maven
sudo chown -R maven:maven /opt/maven

# Set environment variables (add to /etc/profile)
export M2_HOME=/opt/maven
export PATH="$PATH:$M2_HOME/bin"

# Verify
source /etc/profile
mvn --version
```

<img width="873" height="233" alt="Screenshot 2025-11-24 at 1 05 16 PM" src="https://github.com/user-attachments/assets/28cad4a3-7920-4033-a5ef-02eb097d16fd" />

> Note: The `mvn` file is a shell script that sets up the environment and launches Maven’s core Java class.

---
**Project Structure & Creation**

**Directory Structure (Default Convention)**
```
project-root/
  pom.xml
  src/
    main/
      java/           # Production source
      resources/      # Classpath resources
      webapp/         # (WAR) JSPs, WEB-INF
    test/
      java/           # Test source
      resources/      # Test resources
  target/
    classes/
    test-classes/
    <artifact>.jar or <artifact>.war
```


**Examples**


**Step 1: Create a project structure (WebApp example)**

  - Web Application
```
mkdir -p ~/sampleapp
cd ~/sampleapp
```
```
mvn archetype:generate \
  -DgroupId=com.devopsclass \
  -DartifactId=sample-app \
  -DarchetypeArtifactId=maven-archetype-webapp \
  -DinteractiveMode=false
```
<img width="717" height="297" alt="Screenshot 2025-11-25 at 9 32 44 AM" src="https://github.com/user-attachments/assets/d6b5839b-11a1-4e61-a17a-961095bb8632" />

  - Inspect structure:
```
tree sample-app
```

  - Then:
```
cd sample-app
ls
  # Expect: pom.xml  src/

ls src/main/webapp
  # Expect: index.jsp (you can edit)
```


  - Spring Boot (via Spring Initializr)
```
mkdir -p ~/example
cd ~/example
```

```bash
curl -sSL --fail \
  "https://start.spring.io/starter.zip?type=maven-project&language=java&groupId=com.example&artifactId=demo&name=demo&packaging=jar&javaVersion=21&dependencies=web" \
  -o demo.zip
```
> -sSL --fail: silent mode, follows redirects, fails on error
```
unzip demo.zip -d .
```
<img width="805" height="461" alt="Screenshot 2025-11-25 at 12 18 03 PM" src="https://github.com/user-attachments/assets/2991de7a-b454-4230-b838-b3fe2156de20" />


  - Quarkus Application
```
mkdir -p ~/example1
cd ~/example1
```
```
mvn io.quarkus:quarkus-maven-plugin:3.15.0:create \
  -DprojectGroupId=com.example1 -DprojectArtifactId=quarkus-app \
  -DclassName="com.example1.GreetingResource" -Dpath="/hello"
```
<img width="800" height="519" alt="Screenshot 2025-11-25 at 12 18 28 PM" src="https://github.com/user-attachments/assets/03a498ff-6dca-4d8d-af35-e227149c2912" />


**Explanation:**
  - Command:
```
mvn archetype:generate
```

  - Parameters:
```
-DgroupId=com.example                         # Defines base package (reverse-domain)
-DartifactId=sample-app                       # Project name and root directory
-DarchetypeArtifactId=maven-archetype-webapp  # Web application (WAR)
-DinteractiveMode=false                       # Skip interactive prompts
```

  - Common archetypes:
```
maven-archetype-quickstart    # Simple Java project (JAR)
maven-archetype-webapp        # Basic web application (WAR)
```

Framework-specific tips:
  - Spring Boot: Typically generated via Spring Initializr rather than Maven archetypes.
```
curl https://start.spring.io/starter.zip -d type=maven-project \
  -d language=java -d bootVersion=3.3.4 -d groupId=com.example \
  -d artifactId=demo -d name=demo -d packaging=jar -d javaVersion=21 \
  -d dependencies=web -o demo.zip
unzip demo.zip -d .
```

---

**Role of Maven in SDLC**

Requirements → Design → Implementation → Testing → Deployment → Maintenance & Operations

  - Maven supports the Implementation, Testing, Packaging, and Release/Deployment stages with consistent, repeatable builds and integrations to CI/CD.

---

**The pom.xml**

  - `pom.xml` (Project Object Model) is the core configuration file for Maven projects. It defines project metadata, dependencies, build plugins, and other related information.

  - Minimal Web Application POM
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.devopsclass</groupId>
  <artifactId>sample-app</artifactId>
  <version>1.0.0</version>
  <packaging>war</packaging>

  <name>sample-app</name>
  <description>Sample web application built with Maven</description>

  <properties>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <!-- Container-provided APIs -->
    <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <version>6.0.0</version>
      <scope>provided</scope>
    </dependency>
    
    <!-- Test dependency -->
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.10.3</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- JUnit 5 Support -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.5.1</version>
        <configuration>
          <useModulePath>false</useModulePath>
        </configuration>
      </plugin>

      <!-- Embedded Server for Development -->
      <plugin>
        <groupId>org.eclipse.jetty</groupId>
        <artifactId>jetty-maven-plugin</artifactId>
        <version>12.0.10</version>
      </plugin>
    </plugins>
  </build>
</project>
```

**Advanced POM Features**
  - Parent POM & Dependency Management
```
<!-- Inherit corporate standards -->
<parent>
    <groupId>com.mycompany</groupId>
    <artifactId>company-parent-pom</artifactId>
    <version>1.0.0</version>
</parent>

<!-- Bill of Materials pattern -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.7.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**Profiles for Environment-Specific Configs**
```
<profiles>
  <profile>
    <id>prod</id>
    <properties>
      <app.env>production</app.env>
    </properties>
    <activation>
      <property>
        <name>env</name>
        <value>prod</value>
      </property>
    </activation>
  </profile>
</profiles>
```

---

**Build & Deployment Workflow**

  - Compilation & Packaging
    - Run inside the project directory where `pom.xml` resides. Maven will download required plugins and cache them in `~/.m2/repository`.

Compile:

```
# Navigate to project directory
cd ~/sampleapp/sample-app

# Compile source code
mvn compile

# Run tests and create a package
mvn package

# Check outputs
ls target/  # sample-app-1.0.0.war
```

<img width="798" height="101" alt="Screenshot 2025-11-25 at 12 23 25 PM" src="https://github.com/user-attachments/assets/9e4ef888-9c60-4540-a260-0cbaaa0b1f32" />


  - Local Development Testing
```
# Run with embedded Jetty
mvn jetty:run

# Access at: http://localhost:8080/sample-app/
```


**Deploying the WAR to Tomcat**

Assuming:
  - Tomcat server is powered on and accessible.
  - Developer machine has built `target/sample-app.war`.
  - Tomcat’s `webapps/` directory is on the remote host.
  - The default Tomcat HTTP connector is 8080 unless you reconfigure.

```
# Copy WAR to remote Tomcat
cd target/
scp sample-app.war cnode@10.10.5.9:~/apache-tomcat-9.0.106/webapps

# Access deployed application
http://10.10.5.9:8080/sample-app/

# Or if custom port: http://10.10.5.9:5080/sample-app/
```

  - Access the app in a browser (adjust port/context path if configured differently):
```
http://10.10.5.9:8080/sample-app/
# If your Tomcat uses port 5080 per your setup:
# http://10.10.5.9:5080/sample-app/
```

Make changes and redeploy:

```
# On developer machine
cd /home/cnode/javaapp/sample-app/src/main/webapp

sudo vim index.jsp
# ...change the content...

cd ../../../
mvn package
cd target
scp sample-app.war cnode@10.10.5.9:~/apache-tomcat-9.0.106/webapps
```

---

**Essential Maven Concepts**

  - Dependency Scopes
| Scope     | Usage                                      | Example                          |
|-----------|----------------------------------------------|----------------------------------|
| compile   | Default; available everywhere                | Core application dependencies    |
| provided  | Provided by container/JDK                    | servlet-api in Tomcat            |
| runtime   | Needed at runtime but not compile-time       | JDBC drivers                     |
| test      | Only available on test classpath             | JUnit, Mockito                   |
| system    | Like `provided` but requires explicit path   | Rare, avoid when possible        |


  - Dependency Conflict Resolution
```
<!-- Exclude problematic transitive dependency -->
<dependency>
    <groupId>problematic.dependency</groupId>
    <artifactId>bad-lib</artifactId>
    <exclusions>
        <exclusion>
            <groupId>conflicting-group</groupId>
            <artifactId>conflicting-artifact</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

  - Build Optimisation Techniques
```
# Parallel builds
mvn -T 4 clean install

# Offline mode (no remote repository checks)
mvn -o clean install

# Skip tests for faster packaging
mvn package -DskipTests
```

---

**CI/CD Integration**
  - Jenkins Pipeline Example

```groovy
pipeline {
  agent any
  tools { 
    maven 'Maven-3.9' 
    jdk 'OpenJDK-21'
  }
  
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    
    stage('Build & Test') {
      steps { 
        sh 'mvn -B -T 1C clean compile test' 
      }
      post {
        always { 
          junit 'target/surefire-reports/*.xml' 
        }
      }
    }
    
    stage('Security Scan') {
      steps {
        sh 'mvn org.owasp:dependency-check-maven:check'
      }
    }
    
    stage('Package') {
      steps { 
        sh 'mvn -B package -DskipTests' 
      }
      post {
        success {
          archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        }
      }
    }
    
    stage('Deploy to Nexus') {
      when { branch 'main' }
      steps { 
        sh 'mvn -B deploy -DskipTests' 
      }
    }
  }
  
  post {
    always {
      dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
  }
}
```

  - Repository Caching Strategy
```
# Cache Maven repository in CI (GitLab CI example)
cache:
  paths:
    - .m2/repository
  key: "$CI_COMMIT_REF_SLUG-maven"
```

---

**Repository Management & Security**
  - settings.xml Configuration
    - Mirror to Corporate Repository
```
<settings>
  <mirrors>
    <mirror>
      <id>corp-nexus</id>
      <name>Corporate Nexus</name>
      <url>https://nexus.example.com/repository/maven-group/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>

  <servers>
    <server>
      <id>corp-nexus</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
```

  - Proxy Configuration
```
<settings>
  <proxies>
    <proxy>
      <id>corp-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.example.com</host>
      <port>3128</port>
      <nonProxyHosts>localhost|127.0.0.1|*.example.com</nonProxyHosts>
    </proxy>
  </proxies>
</settings>
```

  - Security Scanning
```
# OWASP Dependency Check
mvn org.owasp:dependency-check-maven:check

# SpotBugs for code quality
mvn com.github.spotbugs:spotbugs-maven-plugin:spotbugs

# SonarQube analysis
mvn sonar:sonar -Dsonar.projectKey=my-project
```

---

**Maven Values in DevOps**

| Benefit | Explanation | Outcome |
|---------|-------------|---------|
| Declarative Build | pom.xml encodes project rules | Less custom scripting |
| Dependency Management | Transitive resolution with scopes | Fewer manual JAR wrangling errors |
| Convention Directory Layout | src/main/java, src/test/java, target | Predictability across teams |
| Lifecycle Abstraction | Standard phases tied to plugins | Easier CI pipeline composition |
| Artifact Versioning | groupId:artifactId:version coordinates | Traceable deployments |
| Extensible Plugins | Rich ecosystem (Surefire, Compiler, Shade, Jib) | Fast feature integration |
| Reproducibility | Locked versions + deterministic steps | Avoid “works on my machine” |
| CI/CD Friendly | Consistent exit codes + lifecycle | Faster automation adoption |

