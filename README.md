# About
This repository contains code for my portfolio web pplication.

This projects builds a Jenkins CI pipeline. Every time I commit changes to this repository. Jenkins will build the docker image of my web application, push it to a docker repository that is being hosted on Sonatype Nexus, and update the version of the web application in the GitHub.

Read this [blog](https://bhairavisanskriti.hashnode.dev/publish-docker-image-to-nexus-using-jenkins) for a better understanding of this project.

## OS
- use any Ubuntu version for the host server machine.

## Firewall config of nexus-server droplet:
- port `22`  : ssh - add only your device's ip address
- port `8081`: nexus - this is where your nexus application will run
- port `8082`: docker repo - this is the IP add of the docker(hosted by Nexus) where we will store images

## SSH into the server
- `ssh root@<ipAddressServer>`

## Beautify your prompt
- `echo 'PS1="\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "' >> .bashrc && source .bashrc`

# Nexus Setup 
## Nexus Host Server 
- Install docker on Nexus server:  `apt update -y && apt install docker.io -y`

## Run Nexus container:
- `docker volume create --name nexus-data`
- `docker run -d -p 8081:8081 -p 8082:8082 --name nexus -v nexus-data:/nexus-data sonatype/nexus3`

## Firewall config of jenkins-server droplet:
- port `22`  : ssh - add only your device's ip address
- port `8080`: jenkins - this is where your jenkins application will run

## Create Blob Store
Create a storage where all the components of your docker repository will be stored.

## Created a docker hosted repo
Create a docker(hosted) repository and assign port *8082* to it. 

## Create Role
Create a role. Provide this role access to the docker repo.

## Create User
Create a user. Assign this user the role that you just created.

# Jenkins setup
## Jenkins Host Server 
- Install docker on Jenkins server:  `apt update -y && apt install docker.io -y`
	
## Run Jenkins container:
- `docker run -d -p 8080:8080 -p 5000:5000 -v jenkins_home:/var/jenkins_home -v /usr/bin/docker:/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock --name jenkins jenkins/jenkins:lts-jdk11`

## Add docker repo hosted by Nexus to list of insecure regitries
- ssh into the *jenkins-server* as a root user.
- `vi /etc/docker/daemon.json`
  - Add `{"insecure-registries": ["64.227.128.95:8082"]}` to the *daemon.json* file. Create this file if it doesn't exist already. The IP address here is the address of the docker repo that you hosted on Nexus.
- Resart docker to reflect the changes:
  - `systemctl daemon-reload`
  - `systemctl docker restart docker`
  - Check whether the *jenkins* user in the jenkins container still has the permission to read `/var/run/docker.sock` file or not. If not, give the permission again.

## Give read permission to the *jenkins* user for docker.sock file:
- `docker exec -it -u 0 jenkins bash`
- `chmod 666 /var/run/docker.sock`
- `exit`

## Passwword to login into Jenkins as a admin 
- This file can be accessed by root user as well
- `docker exec -it jenkins bash`
- `cat /var/lib/jenkins/secrets/initialAdminPassword`

## Pipeline
- Create below creadentials IDs: 
	1. github (git repository)
	2. dockerhub (artifact repository)
	
## Trigger Jenkins pipeline
### Github Webhooks
- Ad webhook in github
- `<ipAddressOfHostServer>:<portNoOfJenkins>/github-webhook/` 
	- E.g.-> `143.110.184.204:8080/github-webhook/` 
			
### Jenkins
- Add GitHub SCM trigger
- Add 'jenkins' user to the list of *Polling ignores commits from certain users*!

