---
marp: true
theme: default
---

# Docker crash course (Docker and Compose)

---

## prerequisites

- JDK
- IDE
- git projects
    - [Docker project](https://github.com/PieterHuybrechts/docker-course-project) 
    - [Docker compose project](https://github.com/bartvoet/docker-springboot-demo)
- [docker](https://docs.docker.com/engine/install/)
  - check if docker is running: `docker run hello-world`
 
---

# Agenda

- Project
- Docker
  - Create image
  - Running container
  - Networking
- Docker compose
  - Create compose-file
  - Add database
  - Externalize configuration
  - Split over multiple composes
  

---

# What is Docker

- Containerization platform
- Package software to be run on any system
- Env with all dependencies to run our code

---

# Create your first Docker container

![image](./resources/dockerfile-image-container.png)

---

# Create your first Docker container

## Dockerfile

- create a file called `Dockerfile`

```docker
FROM debian

ENTRYPOINT ["echo", "Hello world"]
```

- build this file through the following command

```
docker build . -t my-hello-world
```

- run the image

```
docker run my-hello-world
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

## What?

- Define **multi container** applications
- **Centralize** configuration in 1 yaml file
- Is **not K8s**, it **lacks** **production**-features like
  - load balancing
  - service discovery
  - Auto scaling
- But it's
  - relatively **easy** to set up
  - **automates** a lot for **devs**
    - middleware, databases without installation
    - closest **simple** thing to production... 

---

## Docker Compose "in practice"

Next follow a couple of simple demo's you can **participate** during the sessions.  

Ideally you have cloned the markdown-version of this presentation making it easy to copy/paste some code-snippets at
https://github.com/PieterHuybrechts/docker-course-slides

> In case you just want to see the final result you can just checkout
> https://github.com/bartvoet/docker-springboot-demo

---

## Part 1: create a simple application

* So let's **get started** with 
  * A **simple Spring Boot** applciation
  * Only **1** image/container/**service**
  * Just to **demonstrate** the **mechanics** behind Docker **Compose**

---

### Step 1: Create a Spring Boot application

* As a first step let's create a **simple Spring Boot-app** itself
* As a starting-point use the following link to create a simple Spring Boot-app through **Spring Initializr**

https://start.spring.io/#!type=gradle-project&language=java&platformVersion=3.4.0&packaging=jar&jvmVersion=21&groupId=com.example&artifactId=demo&name=demo&description=Demo%20project%20for%20Spring%20Boot&packageName=be.demo.docker.hello&dependencies=web

* In Spring Initialzr Click **"Generate"** and **download**  
* Next **import** the application into your favorite IDE or editor

---

### Step 2: Add a dockerfile

Add the following **Dockerfile** to the **root** of your **project**

~~~docker
FROM eclipse-temurin:21-jdk-alpine
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
ARG JAR_FILE=./build/libs/*-SNAPSHOT.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
~~~

> To enhance the security we've added a spring-group and -user to avoid
> root-access

---

### Step 3: Add a docker-compose-file

Next we add a **compose-file**, this is yaml-file describing **1 or more services**  
Beside these services you can describe **volumes**, **network**, **startup-dependencies** as we will see later

```yaml
services:
  learning-service:
    container_name: demo_application
    build:
      dockerfile: Dockerfile
    image: demo_application:latest
    ports:
      - "8080:8080"
```

Store this file as **docker-compose.yml** in the **root** of your **project**

---

### Step 4: Add a simple controller

Once we've set up the Docker-configuration we add some **simple code** in order to test...  
For this **copy** the following snippet in your **source-folder** 

~~~java
package be.demo.docker.hello;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DemoController {

    @GetMapping("/")
    public String helloWorld() {
        return "Hello World";
    }
}
~~~

---

### Step 5: Perform a build

**Before** to be **able** to create the image/container/**service** we perform a **local** **gradle-build** through the command `./gradlew clean build`

> For this part you need to start a command-line at the root of your project (for this and next steps)

~~~bash
$ ./gradlew clean build
OpenJDK 64-Bit Server VM warning: Sharing is only supported for boot loader classes because bootstrap classpath has been appended

BUILD SUCCESSFUL in 4s
8 actionable tasks: 8 executed
$ 
~~~

---

### Step 5: Build the image

Next we let Docker Compose **prepare** and **build** the **image**.  
This you can launch through the command  `docker compose build`

~~~bash
$ docker compose build
[+] Building 0.7s (9/9) FINISHED docker:default
 => [learning-service internal] load build definition from Dockerfile
 => => transferring dockerfile: 250B
 => [learning-service internal] load metadata for docker.io/library/eclipse-temurin:21-jdk-alpine
 => [learning-service internal] load .dockerignore
 => => transferring context: 2B
 => [learning-service 1/3] FROM docker.io/library/eclipse-temurin:21-jdk-alpine@sha256:49...
 => [learning-service internal] load build context
 => => transferring context: 113B
 => CACHED [learning-service 2/3] RUN addgroup -S spring && adduser -S spring -G spring
 => CACHED [learning-service 3/3] COPY ./build/libs/*-SNAPSHOT.jar app.jar
 => [learning-service] exporting to image
 => => exporting layers
 => => writing image sha256:c27890b580ba722b75812245285797ed42ab267c052362c6be46f9d62db57a22
 => => naming to docker.io/library/demo_application:latest
 => [learning-service] resolving provenance for metadata file
$ 
~~~

---

### Step 6: Build and run the container

To start the service you just defined you perform the command `docker compose build`

~~~bash
$ docker compose up
[+] Running 1/0
 ✔ Container demo_application  Created                                                                                                                                            0.1s 
Attaching to demo_application
demo_application  | 
demo_application  |   .   ____          _            __ _ _
demo_application  |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
demo_application  | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
demo_application  |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
demo_application  |   '  |____| .__|_| |_|_| |_\__, | / / / /
demo_application  |  =========|_|==============|___/=/_/_/_/
...
demo_application  | 2024-11-25T22:34:19.265Z  INFO 1 --- [demo] [           main] be.demo.docker.hello.DemoApplication     : Started DemoApplication in 1.784 seconds (process running for 2.237)
~~~

This will start up services you defined in the compose-file

----

### Step 7: Stopping the services

In order to stop the service you just perform a **ctrl-c**, this will stop the service(s) defined in you compose-file.

~~~bash
Aborting on container exit...
[+] Stopping 1/1
 ✔ Container demo_application  Stopped  0.3s 
canceled
$
~~~

---

### Important tip: build and run

So far we need **3 steps**:

* Building your application with `./gradlew clean build`
* Building the container through `docker compose build`
* Starting the services through `docker compose up`

You can however also **combine** the 2 latest steps (build and run) in one command.  
The command `docker compose up --build` ensures that you will always rebuild your application.

----

### Important tip: IDE-support for docker-compose

* Needless to say is that most IDE's have direct support (or through plugins) for Docker and Docker Compose

![](intellij-plugin.png)

![](vscode-plugin.png)

---

## Part 2: create a "composed" application

In the next part we will extend this to a **composed** application containing

* A Spring Boot-application  
  (similar but with JPA)
* Database (MySql)

Let's get started

---

### Step 1: Create a (new) Spring Boot application

* Use the following link to create a simple Spring Boot-app through **Spring Initializr** (same as before but with jpa-starter + mysql-driver)

https://start.spring.io/#!type=gradle-project&language=java&platformVersion=3.4.0&packaging=jar&jvmVersion=21&groupId=be.demo.docker&artifactId=demodb&name=demodb&description=Demo%20project%20for%20Spring%20Boot&packageName=be.demo.docker.demodb&dependencies=web,data-jpa,mysql

* Click **"Generate"** and download
* Import the application into your favorite IDE or editor

---

### Step 2: Same Dockerfile as before

Add the following **Dockerfile** to the root of your project

~~~docker
FROM eclipse-temurin:21-jdk-alpine
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
ARG JAR_FILE=./build/libs/*-SNAPSHOT.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
~~~

> This should be the same as before

---

### Step 3: Add the (extended) docker-compose-file

Add the following **docker-compose.yml**  

```yaml
services:
  learning-service:
    container_name: demodb_application
    depends_on:
      db:
        condition: service_started
    build:
      dockerfile: Dockerfile
    image: demodb_application:latest
    environment:
      - DB_HOST=mysql_learning
      - DB_NAME=learning_db
      - DB_USERNAME=learning
      - DB_PASSWORD=learning
      - DB_PORT=3306
    ports:
      - "8080:8080"
    expose:
      - "8080"
    volumes:
      - ./.infra/logs/:/logs/
  db:
    image: mysql:latest
    container_name: mysql_learning
    restart: always
    environment:
      MYSQL_DATABASE: 'learning_db'
      MYSQL_USER: 'learning'
      MYSQL_PASSWORD: 'learning'
      MYSQL_ROOT_PASSWORD: 'learning'
    ports:
      - "3306:3306"
    expose:
      - "3306"
    volumes:
      - ./.infra/mysql/storage/_data:/var/lib/mysql
```

---

#### Some explanation on step 3

The big difference with the previous example:

* A **second service** (db) has been **added**
* A **startup** **dependency** has been **added** from the application to the db
* **Environment variables** has been added to configure both services
  * In Spring Boot this will **bind** to values in the **property-file**
* **Volumes** are used **mounting** some mutable disk space for the database and the logs

> This compose will create a local folder *.infra* when starting up the db.  
> In the demo project I've added *.infra* into your *.gitignore*

---

### Step 4: Let's add some code and configuration

We need add some more code compared to last time:

* HelloMessage-entity
* HelloRepo for fetching and storing HelloMessages
* HelloWorldController defining 2 entrypoints
  * "sayhello" using a query-parameter to pass a message
  * "listhello" for listing the query-parameters
* application.properties

> The controller is only using gets just to make it easy to test in the demo

---

#### Step 4a: Make an entity and repo

~~~java
package be.demo.docker.hellodb;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity(name="hello")
public class HelloMessage {
    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    private Long id;

    private String message;

    protected HelloMessage() {

    }

    public HelloMessage(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
~~~

----

#### Step 4b: Add a repo

~~~java
package be.demo.docker.hellodb;

import org.springframework.data.jpa.repository.JpaRepository;

public interface HelloRepo extends JpaRepository<HelloMessage, Long> {
}
~~~

-----

#### Step 4c: Add the controller

~~~java
package be.demo.docker.hellodb;

import jakarta.transaction.Transactional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.stream.Collectors;

@RestController
public class HelloWorldController {

    @Autowired
    private HelloRepo helloRepo;

    @GetMapping("sayhello")
    @Transactional
    public String sayhello(@RequestParam String hello) {
        helloRepo.save(new HelloMessage(hello));
        return "You said " + hello;
    }

    @GetMapping("listhellos")
    @Transactional
    public String listhellos() {
        return helloRepo.findAll()
                .stream().map(HelloMessage::getMessage)
                .collect(Collectors.joining("\n"));
    }
}
~~~

---

#### Step 4d: Override the application-properties

* Override the application.properties
  * Bind to the environment-variables
  * Configure a logging-directory (using the mounted volume)

```properties
spring.application.name=demodb

spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://${DB_HOST}:${DB_PORT}/${DB_NAME}?useSSL=false&allowPublicKeyRetrieval=true
spring.datasource.password=${DB_USERNAME}
spring.datasource.username=${DB_PASSWORD}
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

logging.file.path=/logs/measurements.log
```

---

### Step 5: Perform a gradle-build

* Before removing the build remove the test

~~~bash
$ ./gradlew  clean build -x test

BUILD SUCCESSFUL in 1s
6 actionable tasks: 6 executed
$
~~~

> For now we skip tests because the database is not available and required when you load the Spring Context

---

### Step 6: Run and build the compose

We perform the build and startup in 1 command: `docker compose up --build`

```bash
$ docker compose up --build
...
mysql_learning       | 2024-11-25T23:28:10.576494Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.2.0) starting as process 1
mysql_learning       | 2024-11-25T23:28:10.584372Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
mysql_learning       | 2024-11-25T23:28:10.719351Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
db_demo_application  | 
db_demo_application  |   .   ____          _            __ _ _
db_demo_application  |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
db_demo_application  | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
db_demo_application  |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
db_demo_application  |   '  |____| .__|_| |_|_| |_\__, | / / / /
db_demo_application  |  =========|_|==============|___/=/_/_/_/
...
$ 
```

---

### Step 7: test

If the previous (boot-step) went well, we perform a test:

> The demo is using curl but you can just use the browser if that's more convenient

~~~bash
$ curl http://localhost:8080/sayhello?hello=first_message
You said first_message
$ curl http://localhost:8080/listhellos
first_message
$ curl http://localhost:8080/sayhello?hello=second_message
You said second_message
$ curl http://localhost:8080/listhellos
first_message
second_message
$
~~~

---

## Part 3: split the compose file to allow for native development

What if you want to run only one part:

* Run and debug your application in your favourite IDE
* Use only DB-part

Compose-files allow you to modularize the services into separte files.  
In this example will separate the database-service-definition, so let's get started...

---

### Step 1: Create a separate compose-file for database

* Copy the database-part into a separate docker-compose-db.yml

```yaml
services:
  db:
    image: mysql:latest
    container_name: mysql_learning
    restart: always
    environment:
      MYSQL_DATABASE: 'learning_db'
      MYSQL_USER: 'learning'
      MYSQL_PASSWORD: 'learning'
      MYSQL_ROOT_PASSWORD: 'learning'
    ports:
      - "3306:3306"
    expose:
      - "3306"
    volumes:
      - ./infra_/mysql/storage/_data:/var/lib/mysql
```

---

### Step 2: Split compose

* Remove the db-part from the original yml
* But provide an include-directive pointing to the new file

```yaml
include:
  - docker-compose-db.yml
services:
  learning-service:
    container_name: demodb_application
   ...
```

---

### Step 3: Test for regression

Ensure that your application is still running by **booting** it once again

```bash
$ docker compose up --build
```

And stop it through **ctrl-c**

---

### Step 4: Test by running only the db-part

* Run the dedicated docker-compose only
* For this you have an **-f**-argument to indicate the specific compose-file

```bash
bartvoe@CI00327265 demo % docker compose -f docker-compose-db.yml up --build
[+] Building 0.0s (0/ docker:desktop-linux
[+] Running 2/2
 ✔ Network demo_default      Created  0.1s 
 ✔ Container mysql_learning  Created  0.1s 
Attaching to mysql_learning
mysql_learning  | 2024-11-26 11:50:35+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.1.0-1.el9 started.
mysql_learning  | 2024-11-26 11:50:35+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
mysql_learning  | 2024-11-26 11:50:35+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 9.1.0-1.el9 started.
...
```

* Only the database should start

---

### Step 5: Run the application

* To complete you could start directly the application natively onto your machine
* e.g. you can start up by issuing the following cmd-line

```bash
 java -DDB_PORT=3306 -DDB_USERNAME=learning -DDB_PASSWORD=learning -DDB_NAME=learning_db -DDB_HOST=localhost  -jar build/libs/*-SNAPSHOT.jar 
```

> You can naturally do the same thing through your IDE by configuring the environment variables...

---

## Next parts

You can **still** start **refining** al lot from here onwards.  

By e.g. using Spring profiles you can configure a standard application.properties for local usage and specialized versions for the docker-variations... 

The aim of this talk was however to give a basic introduction and to enable you:

* To set up basic simple images/containers
* Automate it for local development through compose

If you want to go further and see how you can put in a production-environment don't forget to subscribe to the next sessions...