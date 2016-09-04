#### psgsjenkins2
#####2 Setting up Jenkins
######Running Jenkins with Docker
[jenkins docker image](hub.docker.com/r/library/jenkins)  
use this:
```
docker run -p 8080:8080 -p 50000:50000 jenkins
```
or
```
docker run -p 8080:8080 -p 50000:50000 -v ~/:var/jenkins_home jenkins
```
#####3
######sample pj
[test material](https://github.com/g0t4/jenkins2-course-spring-boot)

```
git clone https://github.com/g0t4/jenkins2-course-spring-boot
cd jenkins2-course-spring-boot/
cd spring-boot-sample
cd spring-boot-sample-atmosphere/
```
######Manual Compilation with Maven
compile
```
apt install maven openjdk-8-jdk
mvn compile
```
######Manually Testing, Packaging, and Running the App1m 43s
```
mvn test
```
######Compiling in Jenkins
Build->Git repo URL
```
https://github.com/g0t4/jenkins2-course-spring-boot.git
```
in build->Invoke top-level Maven targets->Goals (compile)  
POM
```
spring-boot-samples/spring-boot-sample-atmosphere/pom.xml
```
######Cleaning up Past Builds
in build->Invoke top-level Maven targets->Goals (clean package)  
POM
```
spring-boot-samples/spring-boot-sample-atmosphere/pom.xml
```

######Checking out a Git Repository in a Pipeline
version1: groovy pipeline code
```
git 'https://github.com/g0t4/jenkins2-course-spring-boot.git'

sh 'mvn -f spring-boot-samples/spring-boot-sample-atmosphere/pom.xml clean package'

archiveArtifacts artifacts: 'spring-boot-samples/spring-boot-sample-atmosphere/target/*.jar', excludes: null
```

######Changing Directories in a Pipeline
######The Master Agent Model
generate node:Allocate node groovy syntax
```
node{

    git 'https://github.com/g0t4/jenkins2-course-spring-boot.git'
    
    dir('spring-boot-samples/spring-boot-sample-atmosphere'){
    
        sh 'mvn clean package'
    
        archiveArtifacts artifacts: 'target/*.jar', excludes: null
    
    }
}
```
(failed)
or maybe (old version 2.7.1)
```
node{

    git 'https://github.com/g0t4/jenkins2-course-spring-boot.git'
    
    dir('spring-boot-samples/spring-boot-sample-atmosphere'){
    
        sh 'mvn clean package'
    
        step([$class: 'ArtifactArchiver', artifacts: 'target/*.jar', excludes: null])
    
    }
}
```
######High-level Progress with Pipeline Stages
```
node{
    stage 'ke'
    git 'https://github.com/g0t4/jenkins2-course-spring-boot.git'
    
    dir('spring-boot-samples/spring-boot-sample-atmosphere'){
    
        sh 'mvn clean package'
    
        archiveArtifacts artifacts: 'target/*.jar', excludes: null
    
    }
}
```
######Triggering Automatic Builds
build Triggers->Build Triggers->check poll SCM(check repo in intervals)   
syntax is similar to cron  
```
* * * * *
```
######Configuring an Email Server
mailhog
```
docker run --restart unless-stopped --name mailhog -p 1025:1025 -p 8025:8025 -d mailhog/mailhog
```
manage jenkins->configure->E-mail Notification->  
SMTP server (localhost)  
SMTP port(1025)
content type(HTML)
######
refer  
https://gist.github.com/g0t4/747cd20e8563aefc3eac444166983142  
```
def notify(status){
    emailext (
      to: "you@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
```
######Notifications When a Build Fails
```
node{
    notify('started')
    try{
    stage 'ke'
    git 'https://github.com/g0t4/jenkins2-course-spring-boot.git'
    
    dir('spring-boot-samples/spring-boot-sample-atmosphere'){
    
        sh 'mvn clean package'
        junit 'target/surefile-reports/TEST-*.xml'
        archiveArtifacts artifacts: 'target/*.jar', excludes: null
    
    }
    }catch(err){
        notify ("caught: ${err}")
        //echo "caught: ${err}"
        currentBuild.result = 'FAILURE'
    }
    notify('Done')
}

def notify(status){
    emailext (
      to: "you@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
```
######Duplicating a Job
New item->if you want to create a new item from other existing, you can use this option:
######Visualizing Test Results
pipeline->syntax->general build step->publish junit test result report -> test report XMLs:
```
target/surefile-reports/TEST-*.xml
```
result:
```
junit 'target/surefile-reports/TEST-*.xml'
```
so finally our pipeline
```
node{
    notify('started')
    try{
    stage 'ke'
    git 'https://github.com/g0t4/jenkins2-course-spring-boot.git'
    
    dir('spring-boot-samples/spring-boot-sample-atmosphere'){
    
        sh 'mvn clean package'
        junit 'target/surefile-reports/TEST-*.xml'
        archiveArtifacts artifacts: 'target/*.jar', excludes: null
    
    }
    }catch(err){
        notify ("caught: ${err}")
        //echo "caught: ${err}"
        currentBuild.result = 'FAILURE'
    }
    notify('Done')
}

def notify(status){
    emailext (
      to: "you@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
```
