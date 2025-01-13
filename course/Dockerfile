FROM openjdk:21-oracle AS builder
WORKDIR /app
COPY target/course-v0.0.1.jar course-v0.0.1.jar
RUN java -Djarmode=layertools -jar course-v0.0.1.jar extract

FROM openjdk:21-oracle
WORKDIR /app
COPY --from=builder /app/dependencies/ ./
COPY --from=builder /app/spring-boot-loader/ ./
COPY --from=builder /app/snapshot-dependencies/ ./
COPY --from=builder /app/application/ ./
ENTRYPOINT ["java", "-Dspring.profiles.active=docker", "org.springframework.boot.loader.launch.JarLauncher"]