# =============================================================================
# Taller Evaluativo 2 - Fase 2: Empaquetamiento Inmutable Multi-Stage
# Backend: studytrack-api
#
# Complete los TODOs siguiendo las buenas prácticas vistas en el curso:
# reducción de peso, cache de dependencias, usuario no root y ausencia
# total del SDK/compilador en la imagen final de runtime.
# =============================================================================

# ---------- Stage 1: Builder ----------
FROM maven:3.9.6-alpine as builder

WORKDIR /app

# Copiar POM y descargar dependencias primero (cacheo)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copiar resto del código y compilar
COPY src ./src
RUN mvn package -DskipTests

# ---------- Stage 2: Runtime ----------
FROM eclipse-temurin:17-jre-alpine

# Crear usuario no root (en alpine)
RUN addgroup -g 1000 appuser && \
    adduser -u 1000 -G appuser -s /bin/sh -D appuser

WORKDIR /app

# Copiar el WAR desde el builder (Fase 2 pide JAR, pero en pom.xml configuramos WAR)
COPY --from=builder /app/target/*.war /app.war

# Exponer el puerto
EXPOSE 8080

# Cambiar usuario antes de ejecutar
USER appuser

# Comando de ejecución
CMD ["java", "-jar", "/app.war"]