FROM maven:3.8.6-openjdk-11 as builder
WORKDIR /app
COPY . .
RUN mvn -B -DskipTests clean package

FROM openjdk:8u342-slim
WORKDIR /app
COPY --from=builder /app/target/my-app-1.0-SNAPSHOT.jar .
CMD java -jar my-app-1.0-SNAPSHOT.jar