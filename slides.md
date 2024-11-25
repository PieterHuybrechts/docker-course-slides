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
 
---

# Agenda
- Project
- Docker
    - Create image
    - Running container
    - Networking
    - Docker compose (quick)

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
