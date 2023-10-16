FROM eclipse-temurin:17-jdk-alpine
VOLUME /tmp

# Connect repository majguo/gs-rest-service to container image pushed to GitHub Container Registry
LABEL org.opencontainers.image.source https://github.com/majguo/gs-rest-service

# Copy the artifact and run as non-root user
COPY --chown=1001:0 target/*.jar app.jar
USER 1001

ENTRYPOINT ["java","-jar","/app.jar"]