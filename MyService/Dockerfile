# syntax=docker/dockerfile:1

# ---------- Stage 1 : build ----------
FROM gradle:8.5-jdk21 AS build
WORKDIR /app

# GRADLE_USER_HOME est vide dans l'image gradle:8.5-jdk21 (et on y tourne en
# root), donc on le fixe explicitement pour que le cache mount ci-dessous
# pointe vraiment sur le cache de dépendances de Gradle.
ENV GRADLE_USER_HOME=/home/gradle/.gradle

# Fichiers de build d'abord : cette couche ne change que si les dépendances
# changent. Pas besoin de gradlew ni de gradle/ : l'image fournit Gradle 8.5.
COPY build.gradle settings.gradle ./

# Puis les sources, qui changent souvent (couche séparée)
COPY src ./src

# 'bootJar' et non 'build' : 'build' produit AUSSI myservice-x.y.z-plain.jar,
# et deux jars dans build/libs/ font échouer le COPY *.jar du stage runtime.
RUN --mount=type=cache,target=/home/gradle/.gradle/caches \
    gradle bootJar --no-daemon

# Nom de jar déterministe pour le stage suivant
RUN cp build/libs/*.jar /app/app.jar

# ---------- Stage 2 : runtime ----------
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Ne pas tourner en root dans le conteneur
RUN addgroup -S spring && adduser -S -u 10001 -G spring spring
USER spring

COPY --from=build --chown=spring:spring /app/app.jar app.jar

EXPOSE 8080

# MaxRAMPercentage : la JVM respecte la limite mémoire du conteneur
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
