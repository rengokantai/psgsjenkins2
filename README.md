#### psgsjenkins2
#####2
######
[jenkins docker image](hub.docker.com/r/library/jenkins)  
use this:
```
docker run -p 8080:8080 -p 50000:50000 jenkins
```
or
```
docker run -p 8080:8080 -p 50000:50000 -v ~/:var/jenkins_home jenkins
```
