# # ---------- Stage 1: Build ----------
# FROM eclipse-temurin:21-jdk AS builder

# # Set working directory inside the container
# WORKDIR /usr/src/app/

# # Copy only build scripts and wrapper first (enables caching of dependencies)
# COPY gradlew* settings.gradle* build.gradle .
# COPY ./gradle ./gradle

# # Make Gradle wrapper executable
# RUN chmod +x ./gradlew
# RUN ./gradlew


# # Optional: Pre-download dependencies
# RUN ./gradlew downloadRepos

# # Now copy the remaining source files
# COPY . .

# # If you have protobuf or proto files specifically in pb/, this line ensures they're available
# COPY ./pb ./proto

# RUN chmod +x ./gradlew
# # 

# # Build the distributable Java application
# RUN ./gradlew installDist -PprotoSourceDir=./proto


# ###################################################

# # ---------- Stage 2: Runtime ----------
# FROM eclipse-temurin:21-jre

# # Set working directory inside container
# WORKDIR /usr/src/app/

# # Copy only the built output from the builder stage
# COPY --from=builder /usr/src/app ./

# # Set environment variables (optional defaults)
# ENV AD_PORT=9099


# # Make sure the binary is executable (optional safety net)
# RUN chmod +x ./bin/Ad

# # Start the service
# ENTRYPOINT ["./build/install/opentelemetry-demo-ad/bin/Ad"]

FROM eclipse-temurin:21-jdk AS builder

WORKDIR /usr/src/app/

COPY gradlew* settings.gradle* build.gradle .
COPY ./gradle ./gradle

RUN chmod +x ./gradlew
RUN ./gradlew
RUN ./gradlew downloadRepos

COPY . .
COPY ./pb ./proto
RUN chmod +x ./gradlew
RUN ./gradlew installDist -PprotoSourceDir=./proto

#####################################################

FROM eclipse-temurin:21-jre

WORKDIR /usr/src/app/

COPY --from=builder /usr/src/app/ ./

ENV AD_PORT 9099

ENTRYPOINT ["./build/install/opentelemetry-demo-ad/bin/Ad"]
