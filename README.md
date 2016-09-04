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
