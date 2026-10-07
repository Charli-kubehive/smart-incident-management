# ---- build stage ----
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn -q -B clean package -DskipTests

# ---- runtime stage ----
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8088
# Asia/Kolkata avoids the PostgreSQL "invalid value for parameter TimeZone: Asia/Calcutta" crash
ENTRYPOINT ["java","-Duser.timezone=Asia/Kolkata","-jar","app.jar"]
