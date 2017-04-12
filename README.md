# psgsjenkins2
## 2. Setting up Jenkins
### 5 Installing
Here he uses mac.I use CentOS.  
step see [here](https://github.com/rengokantai/set/blob/master/README.md)


### 9 Running Jenkins with Docker
[jenkins docker image](hub.docker.com/r/library/jenkins)  
use this:
```
docker run -p 8090:8080 -p 50000:50000 jenkins
```
or
```
docker run -p 8080:8080 -p 50000:50000 -v ~/:var/jenkins_home jenkins
```
  
  
  
## 3. Creating Application Builds
### 2 Cloning the Sample Project
[test material](https://github.com/g0t4/jenkins2-course-spring-boot)
```
git clone https://github.com/g0t4/jenkins2-course-spring-boot
cd jenkins2-course-spring-boot/
cd spring-boot-sample
cd spring-boot-sample-atmosphere/
```
### 3 Manual Compilation with Maven
compile
centos(update 04/11/2017)
```
yum install -y java tree mvn
mvn compile
```
```
apt install maven openjdk-8-jdk
mvn compile
```
### 4 Manually Testing, Packaging, and Running the App
```
mvn test
```
### 6 Compiling in Jenkins
Build->Git repo URL
```
https://github.com/g0t4/jenkins2-course-spring-boot.git
```
in build->Invoke top-level Maven targets->Goals (compile)  
POM
```
spring-boot-samples/spring-boot-sample-atmosphere/pom.xml
```

### 7 Peeking into the Jenkins Workspace
For centos, project at
```
/var/lib/jenkins/workspace
```
### 9 App Packaging in Jenkins
change Build from `compile` to `package`  

### 10 Archiving Artifacts
#### 02:19
Post-build Actions
```
spring-boot-samples/spring-boot-sample-atmosphere/target/*.jar
```
### 11 Cleaning up Past Builds
in build->Invoke top-level Maven targets->Goals (clean package)  
```
clean package
```
POM
```
spring-boot-samples/spring-boot-sample-atmosphere/pom.xml
```
### 15 Challenging
```
https://git.io/vKSVZ
```
config.xml
```
<?xml version='1.0' encoding='UTF-8'?>
<project>
  <actions/>
  <description></description>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <scm class="hudson.plugins.git.GitSCM" plugin="git@2.5.2">
    <configVersion>2</configVersion>
    <userRemoteConfigs>
      <hudson.plugins.git.UserRemoteConfig>
        <url>ttps://github.com/g0t4/jenkins2-course-spring-boot.git</url>
      </hudson.plugins.git.UserRemoteConfig>
    </userRemoteConfigs>
    <branches>
      <hudson.plugins.git.BranchSpec>
        <name>*/master</name>
      </hudson.plugins.git.BranchSpec>
    </branches>
    <doGenerateSubmoduleConfigurations>false</doGenerateSubmoduleConfigurations>
    <submoduleCfg class="list"/>
    <extensions/>
  </scm>
  <canRoam>true</canRoam>
  <disabled>false</disabled>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <triggers/>
  <concurrentBuild>false</concurrentBuild>
  <builders>
    <hudson.tasks.Maven>
      <targets>clean compile</targets>
      <pom>spring-boot-samples/spring-boot-sample-atmosphere/pom.xml</pom>
      <usePrivateRepository>false</usePrivateRepository>
      <settings class="jenkins.mvn.DefaultSettingsProvider"/>
      <globalSettings class="jenkins.mvn.DefaultGlobalSettingsProvider"/>
    </hudson.tasks.Maven>
  </builders>
  <publishers>
    <hudson.tasks.ArtifactArchiver>
      <artifacts>spring-boot-samples/spring-boot-sample-atmosphere/target/*.jar</artifacts>
      <allowEmptyArchive>false</allowEmptyArchive>
      <onlyIfSuccessful>false</onlyIfSuccessful>
      <fingerprint>false</fingerprint>
      <defaultExcludes>true</defaultExcludes>
      <caseSensitive>true</caseSensitive>
    </hudson.tasks.ArtifactArchiver>
  </publishers>
  <buildWrappers/>
</project>
```

for centos,
```
cd /var/lib/jenkins/jobs
mkdir b && cd b && vi config.xml
```
copy the content.Then manage Jenkins->Reload configuration from Disk  

## 4. Testing and Continuous Integration
### 3 Creating a Pipeline Job to Execute Maven
Pipeline type.  
check Execute concurrent builds if necessary  
```
step([$class: 'ArtifaceArchiver', artifacts:'sring-boot-samples/spring-boot-sample-stmosphere/target/*.jar,excludes:null])
```
### 5 Checking out a Git Repository in a Pipeline
version1: groovy pipeline code
```
git 'https://github.com/g0t4/jenkins2-course-spring-boot.git'

sh 'mvn -f spring-boot-samples/spring-boot-sample-atmosphere/pom.xml clean package'

archiveArtifacts artifacts: 'spring-boot-samples/spring-boot-sample-atmosphere/target/*.jar', excludes: null
```

### 6 Changing Directories in a Pipeline
### 7 The Master Agent Model
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
#####5 Finding and Managing Plugins
######Integrating Code Coverage
plugin->jacoco/HTML publisher
######Publishing HTML Reports
pipeline->syntax->publishHTML->HTML dict(target/site/jacoco/), index(index.html),put generated gradle code before junit test
