**Containerize It**

Docker has a simple *Dockerfile* file format that it uses to specify the *layers* of an image. Create the following Dockerfile in your Spring Boot project:

Example 1. Dockerfile

    FROM openjdk:8-jdk-alpine
    ARG JAR_FILE=target/*.jar
    COPY ${JAR_FILE} app.jar
    ENTRYPOINT ["java","-jar","/app.jar"]

If you use Gradle, you can run it with the following command:

    docker build --build-arg JAR_FILE=build/libs/\*.jar -t springio/gs-spring-boot-docker .

If you use Maven, you can run it with the following command:

    docker build -t springio/gs-spring-boot-docker .

This command builds an image and tags it as springio/gs-spring-boot-docker.

This Dockerfile is very simple, but it is all you need to run a Spring Boot app with no frills: just Java and a JAR file. The build creates a spring user and a spring group to run the application. It is then copied (by the COPY command) the project JAR file into the container as app.jar, which is run in the ENTRYPOINT. The array form of the Dockerfile ENTRYPOINT is used so that there is no shell wrapping the Java process. The Topical Guide on Docker goes into this topic in more detail.

--------------------------
Running applications with user privileges helps to mitigate some risks (see, for example, a thread on StackExchange). So, an important improvement to the Dockerfile is to run the application as a non-root user:

Example 2. Dockerfile

    FROM openjdk:8-jdk-alpine
    RUN addgroup -S spring && adduser -S spring -G spring
    USER spring:spring
    ARG JAR_FILE=target/*.jar
    COPY ${JAR_FILE} app.jar
    ENTRYPOINT ["java","-jar","/app.jar"]

You can see the username in the application startup logs when you build and run the application:

    docker build -t springio/gs-spring-boot-docker .
    docker run -p 8080:8080 springio/gs-spring-boot-docker

Note the *started by* in the first INFO log entry:

     :: Spring Boot ::        (v2.2.1.RELEASE)
    
    2020-04-23 07:29:41.729  INFO 1 --- [           main] hello.Application                        : Starting Application on b94c86e91cf9 with PID 1 (/app started by spring in /)
    ...

-----
**Build a Docker Image with Gradle**
You can build a tagged docker image with Gradle in one command:

    ./gradlew bootBuildImage --imageName=springio/gs-spring-boot-docker

**Build a Docker Image with Maven**
To get started quickly, you can run the Spring Boot image generator without even changing your *pom.xml* (remember that the Dockerfile, if it is still, there is *ignored*):

    ./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=springio/gs-spring-boot-docker
