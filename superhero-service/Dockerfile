
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} spring-mongo-app.jar
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-mongo-app.jar"]


#ADD ./target/*.jar /usr/src/spring-mongo-app.jar
#WORKDIR usr/src
#ENTRYPOINT ["java","-jar", "spring-mongo-app.jar"]