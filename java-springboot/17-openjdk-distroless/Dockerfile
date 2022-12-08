ARG  BUILDER_IMAGE=gradle:7.6.0-jdk17
ARG  PRODUCTION_IMAGE=gcr.io/distroless/java17-debian11

# STEP 1 Builder image
FROM ${BUILDER_IMAGE} as builder

WORKDIR /app

# Only copy dependency-related files and download dependencies
COPY build.gradle* gradle.properties* settings.gradle* /app/
RUN gradle clean build --no-daemon > /dev/null 2>&1 || true

COPY . .
RUN gradle --no-watch-fs --no-daemon clean build -x test && \
    mkdir -p build/libs/dependency && (cd build/libs/dependency; jar -xf ../application.jar)


# STEP 2 build a small image for production
FROM ${PRODUCTION_IMAGE} as production

ARG APP_PORT=8080
ARG APP_VERSION=0.0.0
ENV TZ=America/Bogota

WORKDIR /app
COPY --from=builder --chown=nonroot:nonroot /app/build/libs/dependency/BOOT-INF/lib /app/libs
COPY --from=builder --chown=nonroot:nonroot /app/build/libs/dependency/META-INF /app/META-INF
COPY --from=builder --chown=nonroot:nonroot /app/build/libs/dependency/BOOT-INF/classes /app

USER nonroot

EXPOSE ${APP_PORT}
ENTRYPOINT ["java","-XX:MinRAMPercentage=25.0","-XX:MaxRAMPercentage=75.0","-cp","/app:/app/libs/*","com.example.demo2.Demo2Application"]

