# Project 2: Jenkins CI Pipeline with Java and Maven

Welcome to the second CI/CD project. In Project 1 you built a Jenkins pipeline for a Python app. Here you build the same kind of pipeline for a Java app, and you will see exactly what changes when the language is compiled instead of interpreted.

You do not need Project 1 open to follow this, but it helps to have done it first. The Jenkins setup and the job creation are the same as before. The new material is Maven, compiling, and packaging a JAR.

If you have not read it yet, open `BUILD-TOOLS.md` first. It explains why Java has a real compile and package step that Python did not.

## What is new in this project

- The app is written in Java, which is a compiled language.
- The build tool is Maven, configured in a `pom.xml`, instead of `pip`.
- There is a new `Package` stage that produces a runnable JAR.
- You will see a failure mode Python could never show you: a build that fails because the code does not compile, which is different from a build that fails because a test did not pass.

## Learning objectives

After completing this project you will be able to:

1. Explain the difference between a compiled and an interpreted language inside a pipeline.
2. Describe what Maven does and when you would choose it over Gradle.
3. Build a five-stage Jenkins pipeline for a Java application.
4. Read a Maven `Jenkinsfile` and map each stage to its Maven command.
5. Tell a compile failure apart from a test failure by reading the pipeline output.

## Concepts recap (only the new parts)

You already met CI, Jenkins, pipelines, and the `Jenkinsfile` in Project 1. Here is what is genuinely new.

### Compiled vs interpreted

Python runs your source code directly, so its `Build` stage only installed dependencies. Java is compiled: the source must be turned into bytecode before it can run. That is why this pipeline has a real compile step and an extra `Package` step.

### Maven and the pom.xml

Maven is Java's build tool. It compiles your code, runs your tests, manages your dependencies, and packages the result. You configure it in a file called `pom.xml`, which lists your dependencies (here, JUnit for testing) and how to build the JAR. Maven is the convention-heavy, widely used choice for learning and for enterprise Java. Gradle is the more flexible alternative, common in Android. See `BUILD-TOOLS.md` for the full comparison.

### The JAR

A JAR is the packaged, runnable form of a Java app. The `Package` stage produces `target/app.jar`, and the Docker image simply runs it with `java -jar app.jar`.

## Prerequisites

- Docker and Docker Compose.
- A GitHub account and a repository for this project.
- About 45 to 60 minutes, less if you did Project 1, since the Jenkins setup is the same.

If you still have Project 1's Jenkins running, stop it first with `docker compose down` in that project's `jenkins/` folder so the two do not fight over port 8080.

## Project structure

```
jenkins-cicd-project-2-java/
├── app/
│   ├── pom.xml                              # Maven build file (deps, plugins, JAR config)
│   ├── Dockerfile                           # Runs the built JAR on a slim JRE
│   └── src/
│       ├── main/java/guru/elevatehub/
│       │   ├── App.java                     # Entry point (main method)
│       │   └── Calculator.java              # The code under test
│       └── test/java/guru/elevatehub/
│           └── CalculatorTest.java          # JUnit 5 tests
├── Jenkinsfile                              # The 5-stage pipeline
├── jenkins/
│   ├── Dockerfile                           # Jenkins image with Maven + Docker CLI
│   └── docker-compose.yml                   # Starts Jenkins and a Docker daemon
├── BUILD-TOOLS.md                           # When Maven, Gradle, and pip are used
├── INSTALL-MAVEN.md                         # Install Java + Maven on Windows and Linux
├── RUN-ON-AWS.md                            # Run the project on an AWS EC2 server
├── ASSIGNMENT.md                            # What to submit
└── README.md                               # This file
```

## Where to run this: locally or on AWS

You can do this project in two places, and you should try both.

- **Locally**, on your own laptop, reaching Jenkins at `http://localhost:8080`. Follow the steps below.
- **On AWS**, on an EC2 server, reaching Jenkins at `http://<server-ip>:8080`. Follow `RUN-ON-AWS.md`, which has the full EC2 setup from launching the instance to cleaning up.

The project does not change between the two. The compose file, the `Jenkinsfile`, and the pipeline are identical. Only the address you open in your browser is different. That is the lesson: a pipeline you build locally runs the same way on a server.

If you want Maven on your own machine for the optional manual build below, see `INSTALL-MAVEN.md` for Windows and Linux steps.

## Step 0: Build and test on your own machine first (optional)

If you have Maven installed locally, do the manual version once so you know what the pipeline automates. If you do not have it yet, see `INSTALL-MAVEN.md` for Windows and Linux install steps.

```bash
cd app
mvn clean compile     # compile the code
mvn test              # run the tests
mvn package           # build target/app.jar
java -jar target/app.jar
```

If you do not have Maven locally, skip this. Jenkins has Maven built into its image and will do it for you.

## Step 1: Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit: Java CI/CD project"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

## Step 2: Start Jenkins

```bash
cd jenkins
docker compose up -d --build
```

Get the one-time unlock password:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Open `http://localhost:8080`, paste the password, choose Install suggested plugins, and create your admin user.

## Step 3: Create the pipeline job

1. New Item, name it `cicd-java`, choose Pipeline, click OK.
2. Under Pipeline, set Definition to Pipeline script from SCM.
3. Set SCM to Git and paste your repository URL.
4. Set Branch Specifier to `*/main`, leave Script Path as `Jenkinsfile`.
5. Save.

## Step 4: Run it

Click Build Now and watch the five stages. The first run is slower because Maven downloads its dependencies once. A green run means your code compiled, the tests passed, the JAR was packaged, and the Docker image was built.

Confirm the image exists:

```bash
docker exec jenkins-docker docker images | grep cicd-mini-project-java
```

## Understanding the pom.xml

The `pom.xml` (Project Object Model) is Maven's configuration file. It tells Maven who your project is, what it depends on, and how to build it. Open `app/pom.xml` and read it next to this explanation.

### Project coordinates

```xml
<groupId>guru.elevatehub</groupId>
<artifactId>cicd-mini-project</artifactId>
<version>1.0.0</version>
<packaging>jar</packaging>
```

These four lines identify your project. `groupId` is your organisation, usually a reversed domain name. `artifactId` is the project name. `version` is the release number. `packaging` is what Maven produces, here a `jar`. Together the coordinates form a unique name for your artifact.

### Properties

```xml
<properties>
  <maven.compiler.release>17</maven.compiler.release>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <junit.version>5.10.2</junit.version>
</properties>
```

Properties are reusable settings. `maven.compiler.release` tells Maven to compile for Java 17. Defining `junit.version` here lets you reference it below with `${junit.version}`, so the version lives in one place.

### Dependencies

```xml
<dependencies>
  <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>${junit.version}</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

This is what Maven downloads for you. Each dependency is named by its own coordinates. `<scope>test</scope>` means JUnit is only needed to run tests, not to run the app, so it is not bundled into the final JAR. If your app needed a library at runtime, you would leave the scope out.

### Build plugins

```xml
<build>
  <finalName>app</finalName>
  <plugins>
    <plugin>
      <artifactId>maven-surefire-plugin</artifactId> ...
    </plugin>
    <plugin>
      <artifactId>maven-jar-plugin</artifactId>
      ... <mainClass>guru.elevatehub.App</mainClass> ...
    </plugin>
  </plugins>
</build>
```

`finalName` makes the output predictable: `target/app.jar` instead of a long versioned name. The surefire plugin runs your tests. The jar plugin builds the JAR and writes a `mainClass` into it, so `java -jar app.jar` knows where to start. This is why the Docker image can simply run the JAR.

## Understanding the Jenkinsfile

Open `Jenkinsfile` and read it next to this list. The stages have the same shape as Project 1, with one extra step.

- `Checkout` pulls your code.
- `Build` runs `mvn -B -ntp clean compile`. This is the real difference from Python: your code is compiled into Java bytecode here.
- `Test` runs `mvn -B -ntp test`, then publishes the JUnit reports Maven writes to `target/surefire-reports`.
- `Package` runs `mvn -B -ntp package -DskipTests` to produce `target/app.jar`. Tests are skipped here because the `Test` stage already ran them.
- `Docker Build` copies that JAR into a small JRE image.

The flags `-B` (batch mode) and `-ntp` (no transfer progress) just keep the Jenkins logs clean.

## Writing a Jenkinsfile from scratch

You should be able to write one yourself, not just read ours. A declarative `Jenkinsfile` is built from a small set of blocks. Here is how to assemble one piece by piece.

### 1. The outer shell

Every declarative pipeline starts the same way. `agent any` tells Jenkins to run on any available worker.

```groovy
pipeline {
    agent any

    stages {
    }
}
```

### 2. Add a stage

A `stage` is one step of work. Its `steps` block holds the commands. `sh` runs a shell command on Linux.

```groovy
stages {
    stage('Build') {
        steps {
            sh 'mvn -B -ntp clean compile'
        }
    }
}
```

### 3. Add more stages in order

Stages run top to bottom. Add as many as your process needs.

```groovy
stage('Test') {
    steps {
        sh 'mvn -B -ntp test'
    }
}
```

### 4. Run commands in a subfolder

If your app is not at the repo root, wrap the steps in `dir('subfolder')`. Our app lives in `app/`, so every Maven command runs inside `dir('app')`.

```groovy
stage('Build') {
    steps {
        dir('app') {
            sh 'mvn -B -ntp clean compile'
        }
    }
}
```

### 5. Reuse values with environment

Variables defined in `environment` are available in every stage as `${NAME}`. `BUILD_NUMBER` is provided by Jenkins automatically.

```groovy
environment {
    IMAGE_NAME = 'cicd-mini-project-java'
    IMAGE_TAG  = "${BUILD_NUMBER}"
}
```

### 6. Run actions after a stage or the whole run

A `post` block runs after steps finish, whatever the result. `always`, `success`, and `failure` decide when. We use it to publish the test report and to print a clear final message.

```groovy
post {
    success { echo 'Pipeline succeeded.' }
    failure { echo 'Pipeline failed. Read the first red error above.' }
}
```

Put those pieces together and you have exactly the `Jenkinsfile` in this project. Nothing in it is magic; it is six building blocks stacked in order.

## How the Jenkinsfile lives in your repo

This is what "pipeline as code" means in practice.

1. The `Jenkinsfile` sits at the root of your repository, committed alongside your code. It is version-controlled like everything else, so the pipeline and the code that it builds always travel together.
2. When you created the job, you chose Pipeline script from SCM and set Script Path to `Jenkinsfile`. That tells Jenkins: do not store the pipeline in the Jenkins UI, read it from my repo instead.
3. On every build, Jenkins clones your repo, finds the `Jenkinsfile` at that path, and runs it. If you edit the `Jenkinsfile` and push, the next build uses the new version automatically.

The practical result: to change your pipeline, you edit a file and push. No clicking around in Jenkins. The whole team sees the same pipeline because it is in the repo.

## Exercises

1. **Break a test.** Change an expected value in `CalculatorTest.java`, push, and watch the `Test` stage fail while `Package` and `Docker Build` are skipped. Then fix it.
2. **Add a feature with a test.** Add a `multiply(int a, int b)` method to `Calculator`, write a JUnit test for it, push, and watch the pipeline validate your change.
3. **Cause a compile failure.** Remove a semicolon in `Calculator.java`, push, and notice the pipeline fails at the `Build` stage, not the `Test` stage. This is the failure mode Python could not demonstrate, because Python is not compiled. Then fix it.

## Troubleshooting

- **`mvn: command not found` in the pipeline.** The Jenkins image build did not finish. Rerun `docker compose up -d --build` in the `jenkins/` folder.
- **First build is very slow.** Maven is downloading dependencies into the Jenkins container. This is normal once; later builds are faster.
- **`Cannot connect to the Docker daemon`.** The `jenkins-docker` container is not ready. Confirm both containers are up and rerun the build.
- **Port 8080 already in use.** Project 1's Jenkins is still running. Stop it with `docker compose down` in that project first.

## Cleanup

```bash
cd jenkins
docker compose down        # stop Jenkins, keep data
docker compose down -v     # stop and wipe data for a clean reset
```

## What comes next

You have now built a CI pipeline in two languages and seen that the shape stays the same while the build tool changes. The natural next step is Project 3: continuous delivery, where you push the image to a registry and deploy it. That turns the CI you have built into full CI/CD.
