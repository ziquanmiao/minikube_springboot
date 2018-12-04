SpringBoot Minikube example

## Pre-requisites

* MacOS (with [brew](https://brew.sh/)) _or_ a common Linux distribution (64-bit)
* [VirtualBox](https://www.virtualbox.org/) preferred; other virtualisation techniques are possible but are **unsupported**.
* [Docker](https://www.docker.com/install/) (CE is fine)
* `git`
* [java](https://www.java.com/en/download/) 
* [maven]() - optional


## Workshop

### Run springboot locally


Using Java and Maven, it is easy to write an application, package it, and run it locally

If you have maven already, you can head into the springboot folder of part 0 and build an app referencing pom.xml using
`mvn package` to produce the springboot.jar found in part 0

Then run:
`java -jar path_to_file/springboot.jar`
From there you can head to your browser and see a response like Hello World!

`localhost:8080`


### Dockerize and run again
Lets make this a bit more scalable by creating a docker image out of this

Build Dockerfile

`Dockerfile`
```
FROM java:8
WORKDIR /
ADD SpringBoot.jar /app/SpringBoot.jar
ADD dd-java-agent.jar /app/dd-java-agent.jar

CMD java -javaagent:/app/dd-java-agent.jar \
    -jar /app/SpringBoot.jar
```
`docker build -t springboot_app:001 .`

use docker cli's build command to use the current directory's contents to build a docker image tagged springboot_app version 1

Run Docker Image
```
docker run -d -p 8080:8080 springboot_app:001
```
use docker cli's run command to run a container daemon (-d) based off the image tagged springboot_app:001 and map the host 8080 port to the containers 8080 port

### Turn this into a Kubernetes Deployment and run in minikube

Prerequisites:
 - Minikube
 - Kubectl

Please see [here](https://github.com/phrawzty/datadog_k8s_workshop/blob/master/README.md#install-minikube-and-kubectl) for installation tips

Lets start minikube first
```
minikube start --bootstrapper=localkube
```
we need to scope kubectl cli to the 1 node kubernetes cluster in minikube
```
eval $(minikube docker-env)
```

Next, lets deploy the springboot app that has 2 pods using the specifications outlined in springboot.yaml in part 1

```
kubectl create -f path_to/springboot.yaml
```

You can confirm the pods have spun up using
```
kubectl get pods
```

Kubernetes has created a service to automatically loadbalance the pods that are servicing our app, you can get the IP address of the pod using

```
kubectl get services
```

then in minikube, you can curl the IP's 8080 to get the Hello World

```
minikube ssh
curl IP_OF_SPRINGBOOT:8080
```

### Run an app that talks to Postgres with Datadog set up

now lets run a prebuilt application that naively talks to postgres and is instrumented with Datadog

cd into part 2 directory and run

```
kubectl create secret generic datadog-api --from-literal=token=DDOG_API_KEY

docker build -t sample_postgres:007 ./postgres/
docker build -t sample_springboot:007 ./SpringBoot_app/

kubectl create -f postgres_deployment.yaml 
kubectl create -f springboot_deployment.yaml
kubectl create -f agent_daemon.yaml 
```

This will build the necessary files.
You can explore them by running:
```
kubectl get services
```
to get service IP of springboot app

```
minikube ssh
curl springboot_service_ip:8080
curl springboot_service_ip:8080/query
curl springboot_service_ip:8080/bad
```

to see the various endpoint behaviors


This has also set you up with a specific Datadog account in terms of having data start flowing in. Please check it out!
