ARG  BUILDER_IMAGE=gradle:7.6.0-jdk17
ARG  PRODUCTION_IMAGE=amazoncorretto:17.0.5-alpine

FROM ${BUILDER_IMAGE} as builder

WORKDIR /home/app

LABEL maintainer="paul.manjarres@gmail.com"
LABEL "org.label-schema.name"="java-springboot"
LABEL "org.label-schema.schema-version"="1.0"

# Only copy dependency-related files and download dependencies
COPY build.gradle* gradle.properties* settings.gradle* /home/app/
RUN gradle clean build --no-daemon > /dev/null 2>&1 || true

COPY . .
RUN gradle --no-watch-fs --no-daemon clean build -x test && \
    mkdir -p build/libs/dependency && (cd build/libs/dependency; jar -xf ../application.jar)


# STEP 2 build a small image for production
FROM ${PRODUCTION_IMAGE} as production
RUN apk -U upgrade && apk add bash

RUN addgroup --system javauser && adduser -S -s /bin/false -G javauser javauser
RUN mkdir /app
RUN chown -R javauser:javauser /app

USER javauser

ARG APP_PORT=8080
ARG APP_VERSION=0.0.0
ENV TZ=America/Bogota

COPY --from=builder --chown=javauser:javauser /home/app/build/libs/dependency/BOOT-INF/lib /app/libs
COPY --from=builder --chown=javauser:javauser /home/app/build/libs/dependency/META-INF /app/META-INF
COPY --from=builder --chown=javauser:javauser /home/app/build/libs/dependency/BOOT-INF/classes /app

WORKDIR /app

EXPOSE ${APP_PORT}
# ENTRYPOINT ["java","-XX:MinRAMPercentage=25.0","-XX:MaxRAMPercentage=75.0","-cp","/app:/app/libs/*","com.yagamipaul.Application"]

