# =============================================================================
# Taller Evaluativo 2 - Fase 2: Empaquetamiento Inmutable Multi-Stage
# Frontend: studytrack-frontend
# =============================================================================

# ---------- Stage 1: Builder ----------
FROM node:20-alpine AS builder

WORKDIR /app

# Copiar package.json y descargar dependencias
COPY package*.json ./
RUN npm ci --silent

# Copiar el resto del código y compilar
COPY . .
RUN npm run build

# Cambiar usuario
USER node

# ---------- Stage 2: Runtime ----------
FROM nginx:alpine

# Copiar configuración de nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copiar los archivos estáticos generados por el builder
COPY --from=builder /app/dist /usr/share/nginx/html

# Exponer el puerto
EXPOSE 80

# Comando de ejecución
CMD ["nginx", "-g", "daemon off;"]
