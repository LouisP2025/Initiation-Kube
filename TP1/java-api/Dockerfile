# build
# on utilise une image de base pour construire l'application Java
FROM maven:3.9-eclipse-temurin-21 AS build
# on choisi son répertoire de travail
WORKDIR /app
# on copie les fichiers de dépendances
COPY pom.xml .
# on installe les dépendances
RUN mvn dependency:go-offline -B
# on copie le code source de l'application
COPY src ./src
# on compile l'application
RUN mvn package -DskipTests -B

# run
# on utilise une image de base pour exécuter l'application Java
FROM eclipse-temurin:21-jre
# on choisi son répertoire de travail
WORKDIR /app
# on copie le fichier jar généré par l'étape de build
COPY --from=build /app/target/*.jar app.jar
# on expose le port 8080 pour l'application
EXPOSE 8080
# on indique les commandes à exécuter
ENTRYPOINT ["java", "-jar", "app.jar"]