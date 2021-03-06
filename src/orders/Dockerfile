# Application Metadata
ARG APPLICATION_NAME="orders"
ARG VERSION="0.0.1-SNAPSHOT"
ARG clientVersion=19.3

# ------------
# Stage 1 : Setup the build environment
FROM maven:3.6-jdk-8 as buildenv
ARG GRADLE_VERSION=5.5.1
ENV GRADLE_HOME /opt/gradle

# create source folder
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Get Gradle.
ADD https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip /tmp/gradle.zip
RUN echo "Installing Gradle" && \
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
RUN gradle --refresh-dependencies
#
# ------------

# ------------
# Stage 2 : Build the application
#
FROM buildenv as appbuild
ARG APPLICATION_NAME
ARG VERSION

# Copy the source code. 
# This layer is recreated only when there are actual source chnages 
COPY src /usr/src/app/src

# Install the application
RUN gradle clean bootJar 
RUN ls -ltr /usr/src/app/build/libs
# ------------

# ------------
# Stage 3 : Application container
#
FROM openjdk:8-jre-slim
ARG APPLICATION_NAME
ARG VERSION
ARG clientVersion


# copy the generated application distribution
COPY --from=appbuild /usr/src/app/build/libs/${APPLICATION_NAME}-${VERSION}.jar /app/${APPLICATION_NAME}-${VERSION}.jar

EXPOSE 80
WORKDIR /app
ENV APPLICATION_NAME=${APPLICATION_NAME}
ENV VERSION=${VERSION}
ENV TNS_ADMIN=/usr/lib/oracle/${clientVersion}/client64/lib/network/admin/
ENTRYPOINT java $JAVA_OPTS -jar /app/${APPLICATION_NAME}-${VERSION}.jar --port=80
#
# ------------

