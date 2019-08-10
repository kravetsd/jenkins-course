# 1.Vm Setup
### 1.1 started VM
### 1.2 Installed jenkin using script from the course. 
### 1.3 created user, installed all the sujested plugins

# 2.First build
### 2.1  build manually (without docker)
index.js from the course repo (deocker demo repo) https://github.com/wardviaene/docker-demo.git
we need to install node js, available plugins -> nodejs plugin 
Next we are going to checkout repo with nindex.js and install the app using npm.install.
    - nodejs example app freestyle project.
    - from git repo http
    - build step with shell comnmand "npm intsall"
    - provide with path to nodejs binary (we need to install node to jenkins via mange jenkins)
    -save your project and build it using ui. Shluld exit with success.

###### Start and test your app:
    - root@ip-10-0-1-199:/var/jenkins_home/workspace/nodejs exmple app# pwd
        /var/jenkins_home/workspace/nodejs exmple app
    - NOW you can see that you are really can not start and teest your app locally becaues actually you have no installed NODEJS npm app on your local machine.  so 'npm start' will fail.

### 2.2 package into docker and build/start/test it.
Add plugin CloudBees Docker Build and Publish
To build/start/test your apps using jenkins and docekr plugin you need to make sure your Jenkins docker container can reach docekr socket api. You can create your own docker image. Here you can refere to git repo proivide in course and clone it https://github.com/wardviaene/jenkins-docker.git . 

###### Here the repo where we get our new Dockerfile:
```
root@ip-10-0-1-199:~# git clone https://github.com/wardviaene/jenkins-docker.git
Cloning into 'jenkins-docker'...
remote: Enumerating objects: 19, done.
remote: Total 19 (delta 0), reused 0 (delta 0), pack-reused 19
Unpacking objects: 100% (19/19), done.
root@ip-10-0-1-199:~# cd jenkins-docker/
root@ip-10-0-1-199:~/jenkins-docker# ll
total 20
drwxr-xr-x 3 root root 4096 Jul 27 06:58 ./
drwx------ 9 root root 4096 Jul 27 06:58 ../
drwxr-xr-x 8 root root 4096 Jul 27 06:58 .git/
-rw-r--r-- 1 root root  382 Jul 27 06:58 Dockerfile
-rw-r--r-- 1 root root   49 Jul 27 06:58 README.md
```


###### Here is a new Docker file to build our custom image:
```
root@ip-10-0-1-199:~/jenkins-docker# cat Dockerfile
FROM jenkins/jenkins
USER root

RUN mkdir -p /tmp/download && \
 curl -L https://download.docker.com/linux/static/stable/x86_64/docker-18.03.1-ce.tgz | tar -xz -C /tmp/download && \
 rm -rf /tmp/download/docker/dockerd && \
 mv /tmp/download/docker/docker* /usr/local/bin/ && \
 rm -rf /tmp/download && \
 groupadd -g 999 docker && \
 usermod -aG staff,docker jenkins

user jenkins
```

###### building our custom image from the docker file provided in course:
```
root@ip-10-0-1-199:~/jenkins-docker# docker build -t jenkins-docker .
Sending build context to Docker daemon  80.38kB
Step 1/4 : FROM jenkins/jenkins
latest: Pulling from jenkins/jenkins
a4d8138d0f6b: Already exists
dbdc36973392: Already exists
f59d6d019dd5: Already exists
aaef3e026258: Already exists
5e86b04a4500: Already exists
1a6643a2873a: Already exists
2ad1e30fc17c: Pull complete
bd17e030bbd0: Pull complete
5db7a628e8cf: Pull complete
67b15c5354cf: Pull complete
3c49d20b1808: Pull complete
dec5af58d9cf: Pull complete
37c5b18aeae8: Pull complete
83780060ef63: Pull complete
ebe4867cb928: Pull complete
e6d3b4b723b3: Pull complete
3f518cd88861: Pull complete
0284eee7e4ca: Pull complete
0212d148be1d: Pull complete
Digest: sha256:3b426cff2ed61e2408ad43724d4d1c55fa819cf63afb932dcc91cc28bd1fb785
Status: Downloaded newer image for jenkins/jenkins:latest
 ---> 224eb89e812c
Step 2/4 : USER root
 ---> Running in cc6b85c7ca45
 ---> 0b5d819a419a
Removing intermediate container cc6b85c7ca45
Step 3/4 : RUN mkdir -p /tmp/download &&  curl -L https://download.docker.com/linux/static/stable/x86_64/docker-18.03.1-ce.tgz | tar -xz -C /tmp/download &&  rm -rf /tmp/download/docker/dockerd &&  mv /tmp/download/docker/docker* /usr/local/bin/ &&  rm -rf /tmp/download &&  groupadd -g 999 docker &&  usermod -aG staff,docker jenkins
 ---> Running in 8e854ac6ff50
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 36.9M  100 36.9M    0     0  22.0M      0  0:00:01  0:00:01 --:--:-- 22.0M
 ---> 626004635b4e
Removing intermediate container 8e854ac6ff50
Step 4/4 : USER jenkins
 ---> Running in 788fecfaddfc
 ---> 1365820c25ea
Removing intermediate container 788fecfaddfc
Successfully built 1365820c25ea
Successfully tagged jenkins-docker:latest
```

###### Stoping current container:
```
root@ip-10-0-1-199:~/jenkins-docker# docker stop jenkins
jenkins
```
###### Removing old image:
```
root@ip-10-0-1-199:~/jenkins-docker# docker rm jenkins
jenkins
```

###### Here you can find a script to start new docker container:
```
root@ip-10-0-1-199:~/jenkins-docker# cat  ../jenkins-course/scripts/install_jenkins.sh | grep run
# run jenkins
docker run -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home -d --name jenkins jenkins/jenkins:lts
```

###### Strarting a new docker container with a newly builded image with mounted docker sockets:
```
root@ip-10-0-1-199:~/jenkins-docker# docker run -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock  --name jenkins -d jenkins-docker
5c939198eabadb29b57971c697bbd20adb136680f136d34dd6feaae7097b932b
root@ip-10-0-1-199:~/jenkins-docker#
```

###### Check if you have access to docekr command inside the kenins container
```
ubuntu@ip-10-0-1-199:~$ docker exec -it jenkins bash

jenkins@5c939198eaba:/$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                              NAMES
5c939198eaba        jenkins-docker      "/sbin/tini -- /usr/…"   5 hours ago         Up 5 hours          0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
```

###### Reconfigure previously created manual jenkins job.
Add a new build step "Docker build and publish" Add there your docker hub or another repo credentials and save and run your job.


###### Check a new pushed docker image in your docker hub repo. Then test it by pulling and runnin your up inside the docker. You can call curl and check if your app is up and running.
```
root@ip-10-0-1-199:~# docker pull kdykrg/docker-nodejs-demo
Using default tag: latest
latest: Pulling from kdykrg/docker-nodejs-demo
386a066cd84a: Already exists
75ea84187083: Already exists
88b459c9f665: Already exists
1e3ee139a577: Already exists
f78ff7d0315b: Already exists
f4ba677961ff: Already exists
21db8c3555aa: Already exists
b988de943095: Pull complete
caddc3eb3fe6: Pull complete
85d4f345cbcc: Pull complete
Digest: sha256:52347defe4f59fec4a071915dc9862037bdf678b6e1d5f074e03ab07356e4bf0
Status: Downloaded newer image for kdykrg/docker-nodejs-demo:latest
root@ip-10-0-1-199:~#
root@ip-10-0-1-199:~#
root@ip-10-0-1-199:~#
root@ip-10-0-1-199:~#
root@ip-10-0-1-199:~# docker run -p 3000:3000  --name my-nodejs-app -d kdykrg/docker-nodejs-demo
de1005ffd349045600918924b63eba0b51584d5cbec9817324d0e5f6f15c5b35
root@ip-10-0-1-199:~#
root@ip-10-0-1-199:~# curl localhost:3000
Hello World!root@ip-10-0-1-199:~#
```

# 3.DSL jobs.
### 3.1 Install plugin "Job DSL"
### 3.2 Create a parent job which is going to check your repos and create new jobs based on the code in those repos. Your parent [job code](Jenkins_module/jenkins-course-master/job-dsl/nodejs.groovy)
```
job('NodeJS example') {
    scm {
        git('git://github.com/wardviaene/docker-demo.git') {  node -> // is hudson.plugins.git.GitSCM
            node / gitConfigName('DSL User')
            node / gitConfigEmail('jenkins-dsl@newtech.academy')
        }
    }
    triggers {
        scm('H/5 * * * *')
    }
    wrappers {
        nodejs('nodejs') // this is the name of the NodeJS installation in 
                         // Manage Jenkins -> Configure Tools -> NodeJS Installations -> Name
    }
    steps {
        shell("npm install")
    }
}
```
