ARG APPLICATION_NAME="fulfillment"
ARG VERSION="0.0.1-SNAPSHOT"

# Stage 1 : Setup the build environment
FROM maven:3.6-jdk-8 as buildenv
ARG GRADLE_VERSION=5.5.1
ENV GRADLE_HOME /opt/gradle
# create source folder
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
# Get Gradle.
ADD https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip /tmp/gradle.zip
RUN echo 'Installing Gradle' && \
    unzip /tmp/gradle.zip && \
    rm /tmp/gradle.zip && \
    mv "gradle-${GRADLE_VERSION}" "${GRADLE_HOME}/" && \
    ln --symbolic "${GRADLE_HOME}/bin/gradle" /usr/bin/gradle && \
    echo "Testing Gradle installation" && \
    gradle --version
COPY settings.gradle /usr/src/app
RUN gradle --refresh-dependencies
# copy buildscript and cache all dependencies
COPY build.gradle /usr/src/app
COPY gradle.properties /usr/src/app
RUN gradle --refresh-dependencies

# Stage 2 : Build the application
FROM buildenv as appbuild
# Copy the source code.
# This layer is recreated only when there are actual source chnages
COPY src /usr/src/app/src
# build an executable fat jar
RUN gradle clean assemble

# Stage 3 : Build a native image using GraalVM
FROM oracle/graalvm-ce:19.3.1-java8 as graalvm
COPY --from=appbuild /usr/src/app/build /home/app/fulfillment
WORKDIR /home/app/fulfillment/
RUN gu install native-image
RUN native-image --no-server  -cp libs/fulfillment-*-all.jar

# Step 4 : Build the final application image
FROM ubuntu:19.10
EXPOSE 80
COPY --from=graalvm /home/app/fulfillment/fulfillment /app/fulfillment
ENTRYPOINT ["/app/fulfillment", "-Djava.library.path=/app"]

