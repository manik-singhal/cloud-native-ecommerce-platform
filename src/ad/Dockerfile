FROM eclipse-temurin:21-jdk AS builder

WORKDIR /usr/src/app/

COPY gradlew* settings.gradle* build.gradle .
COPY gradle ./gradle

RUN chmod +x ./gradlew
RUN ./gradlew Downloadrepos

COPY . .
COPY ./pb ./proto

RUN ./gradlew installDist -PprotoSourceDir=./proto

FROM eclipse-temurin:21-jre AS release

WORKDIR /usr/src/app/

COPY --from=builder /usr/src/app/ .

ENV AD_PORT 8080

ENTRYPOINT ["./build/install/opentelemetry-demo-ad/bin/Ad"]
