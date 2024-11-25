---
marp: true
theme: default
---

# K8s crash course

---

## prerequisites
- JDK
- IDE
- [git project](/workspace/personal/k8s-demo/) 
- [docker](https://docs.docker.com/engine/install/)
  - check if docker is running: `docker run hello-world`
<!-- - Install [kubectl](https://kubernetes.io/docs/tasks/tools/)
- Install [minikube](https://minikube.sigs.k8s.io/docs/start/)
    - run `minikube start` once (To prevent downloading during guild meeting) -->

<!-- ---

## Todo
- [/] java project
  - [x] Create spring web project
  - [ ] commit on git
- [x] docker
  - [x] Install docker
  - [x] Create image of java project
- [/] k8s
  - [x] pod definition
  - [/] deployment definition
  - [ ] short explanation about services and communication between services -->
 
---

# Agenda
- Project
- Docker
    - Create image
    - Running container
    - Networking
    - Docker compose (quick)
<!-- - K8s
    - Minikube
    - Pods
    - Deployments
    - Networking -->

---

# Create your first docker container
## dockerfile
- create a file called `Dockerfile`
```docker
FROM debian

ENTRYPOINT ["echo", "Hello world"]
```

```
docker build .
docker run xyz
```

---

# Docker CLI

- docker image ...
  - ls (Lists images)
  - rm (Removes image)
  - build (alias for docker build)

- docker container ...
  - ls (Lists containers. Add --all for stopped containers)
  - rm (Removes container)
  - run (alias for docker run)
  - stop (Stops running container)
  - ...

---

# Project
```java
@SpringBootApplication
@RestController
public class K8sDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(K8sDemoApplication.class, args);
    }

    @GetMapping("/")
    public String helloWorld() {
        return "Hello World";
    }

}
```
- Run the spring app
- `curl localhost:8080`

---



# Create a docker image
## dockerfile
- create a file called `Dockerfile` in the project root
- Soon it will look like this
```docker
FROM eclipse-temurin:21-jdk-alpine

ARG JAR_FILE
COPY ${JAR_FILE} app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```

---

# Create a docker image
## dockerfile
### FROM

```docker
FROM eclipse-temurin:21-jdk-alpine
```

- Base Image
    - Includes java 21 installation
- Build on top of it
- [Dockerhub](https://hub.docker.com/)

---

# Create a docker image
## dockerfile
### COPY
```docker
FROM eclipse-temurin:21-jdk-alpine

COPY build/libs/k8s-demo-0.0.1-SNAPSHOT.jar app.jar
```

- Copy
    - A local file to the image
    - rename file


---

# Create a docker image
## dockerfile
```docker
FROM eclipse-temurin:21-jdk-alpine

COPY build/libs/k8s-demo-0.0.1-SNAPSHOT.jar app.jar

ENTRYPOINT ["java","-jar","/app.jar"] 
# java -jar /app.jar
```

- Entry point 


---

# Create a docker image
## Build the image

```sh
./gradlew build
docker build -t demo:latest .
```

---

# Run a docker image
`docker run demo:latest`
&
`curl localhost:8080`

---

# Run a docker image
`docker run demo:latest`

### Problem
```
➜  ~ curl localhost:8080
curl: (7) Failed to connect to localhost port 8080 after 0 ms: Connection refused
```
### Solution
- Expose
- Publish

---

# Networking
## Expose

```docker
FROM eclipse-temurin:21-jdk-alpine

COPY build/libs/k8s-demo-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```
- Optional
- Documentation
- Ports available in container

---

# Networking
## Publish
- Bind a container port to a host port
- `-p, --publish`
- `docker run -p 8080:8080 demo:latest`

---

# Create a docker image
## ARG
```docker
FROM eclipse-temurin:21-jdk-alpine

ARG JAR_FILE
COPY ${JAR_FILE} app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```

```sh
docker build --build-arg JAR_FILE=build/libs/k8s-demo-0.0.1-SNAPSHOT.jar -t demo:latest .
```


- Argument 
    - Passed when we build the docker image

---

# Docker Compose
- Define multi container applications
- 1 yaml file
- Lacks
    - load balancing
    - service discovery
    - Auto scaling 

---

# Docker compose
```yaml
services:
  frontend:
    image: example/webapp
    ports:
      - "443:8043"
    networks:
      - front-tier
      - back-tier
    configs:
      - httpd-config
    secrets:
      - server-certificate

  backend:
    image: example/database
    volumes:
      - db-data:/etc/data
    networks:
      - back-tier

volumes:
  db-data:
    driver: flocker
    driver_opts:
      size: "10GiB"

configs:
  httpd-config:
    external: true

secrets:
  server-certificate:
    external: true

networks:
  # The presence of these objects is sufficient to define them
  front-tier: {}
  back-tier: {}
```

<!-- ---

# k8s

---

# Minikube
minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.

## start a cluster localy
`minikube start`

---

# Create a pod
## pod.yml
```yml
apiVersion: v1
kind: Pod
metadata:
  name: k8s-demo
spec:
  containers:
    - name: hello-world
      image: demo:latest
      ports:
        - name: apiport
          containerPort: 8080
```

---

# Create a pod
## pod.yml
```yml
kind: Pod
```

kind: type of definition
- namespace
- pod
- deployment
- configmap
- persistent volume
- ...

---

# Create a pod
## pod.yml
```yml
metadata:
  name: k8s-demo
```

Metadata:
- name
- annotations 

---

# Create a pod
## pod.yml
```yml
spec:
  containers:
    - name: hello-world
      image: demo:latest
      ports:
        - name: apiport 
          containerPort: 8080
```

specifications:
- containers []
  - name: container name
  - image: image we want to deploy
  - ports: ports in a container we want to expose

---

# Delete a pod
`kubectl -n default delete pod k8s-demo`

---

# Deployments

```yml
apiVersion: v1
kind: Deployment
metadata:
  name: k8s-demo
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: hello-world
          image: demo:latest
          ports:
            - name: apiport
              containerPort: 8080
```

---

# Deploy image to a k8s cluster
minikube
deployment -->
