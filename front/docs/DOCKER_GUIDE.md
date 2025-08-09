# Docker Guide - Frontend

## 🐳 Configuración de Docker para Frontend

### **Dockerfile de Desarrollo (Basado en el Proyecto Real)**

```dockerfile
# ✅ CORRECTO: Dockerfile de desarrollo del proyecto service-store
FROM node:21.6.0-alpine

# Establecer directorio de trabajo
WORKDIR /app

# Copiar archivos de dependencias
COPY package*.json ./

# Instalar dependencias con flag legacy para compatibilidad
RUN npm install --legacy-peer-deps

# Copiar código fuente
COPY . .

# Exponer puerto de desarrollo
EXPOSE 5173

# Comando de desarrollo con hot reload
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

### **Dockerfile de Producción (Multi-stage Build)**

```dockerfile
# ✅ CORRECTO: Dockerfile de producción optimizado
# Stage 1: Build
FROM node:21.6.0-alpine AS builder

WORKDIR /app

# Copiar archivos de dependencias
COPY package*.json ./

# Instalar dependencias
RUN npm ci --only=production --legacy-peer-deps

# Copiar código fuente
COPY . .

# Build de producción
RUN npm run build

# Stage 2: Production
FROM nginx:alpine

# Copiar archivos buildados
COPY --from=builder /app/dist /usr/share/nginx/html

# Copiar configuración de nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Exponer puerto 80
EXPOSE 80

# Comando de inicio
CMD ["nginx", "-g", "daemon off;"]
```

## 🐙 Docker Compose

### **docker-compose.yml de Desarrollo (Basado en el Proyecto Real)**

```yaml
# ✅ CORRECTO: Docker Compose del proyecto service-store
version: '3'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    image: ${SERVICENAME}_service
    ports:
      - "5173:5173"
    volumes:
      - .:/app
      - /app/node_modules
    networks:
      - enid_service_network
    environment:
      - NODE_ENV=development
      - VITE_API_BASE_URL=${VITE_API_BASE_URL}
      - VITE_STORE_ID=${VITE_STORE_ID}

networks:
  enid_service_network:
    external: true
```

### **docker-compose.prod.yml para Producción**

```yaml
# ✅ CORRECTO: Docker Compose de producción
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    image: ${SERVICENAME}_frontend:${TAG:-latest}
    ports:
      - "80:80"
    networks:
      - enid_service_network
    environment:
      - NODE_ENV=production
    restart: unless-stopped

networks:
  enid_service_network:
    external: true
```

## 🌐 Configuración de Nginx

### **nginx.conf para Producción**

```nginx
# ✅ CORRECTO: Configuración de Nginx para SPA Vue
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Configuración de logs
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Configuración de compresión
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    # Configuración de caché para assets estáticos
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Configuración principal para SPA
    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        # Configuración de seguridad
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;

        # Configuración de CORS para desarrollo
        add_header Access-Control-Allow-Origin "*" always;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
        add_header Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept, Authorization" always;

        # Manejo de rutas de SPA (Vue Router)
        location / {
            try_files $uri $uri/ /index.html;
        }

        # Configuración para API (reverse proxy)
        location /api/ {
            proxy_pass ${BACKEND_URL};
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Configuración para health check
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

## 🔧 Variables de Entorno

### **.env.example (Basado en el Proyecto Real)**

```bash
# ✅ CORRECTO: Variables de entorno del proyecto service-store
# Configuración del servicio
SERVICENAME=service-store

# Configuración de la API
VITE_API_BASE_URL=http://localhost:8000/api
VITE_STORE_ID=1

# Configuración de desarrollo
NODE_ENV=development
VITE_DEV_SERVER_PORT=5173

# Configuración de producción
VITE_APP_TITLE=Service Store
VITE_APP_VERSION=1.0.0

# Configuración de analytics
VITE_ANALYTICS_ID=GA_TRACKING_ID
VITE_ANALYTICS_ENABLED=true

# Configuración de features
VITE_FEATURE_FLAGS_ENABLED=true
VITE_DEBUG_MODE=false
```

### **.env.production**

```bash
# ✅ CORRECTO: Variables de entorno para producción
NODE_ENV=production
VITE_API_BASE_URL=https://api.production.com/api
VITE_STORE_ID=1
VITE_APP_TITLE=Service Store
VITE_APP_VERSION=1.0.0
VITE_ANALYTICS_ID=PROD_GA_TRACKING_ID
VITE_ANALYTICS_ENABLED=true
VITE_FEATURE_FLAGS_ENABLED=false
VITE_DEBUG_MODE=false
```

## 🚀 Comandos de Docker

### **Comandos de Desarrollo**

```bash
# ✅ CORRECTO: Comandos para desarrollo

# Construir imagen de desarrollo
docker build -t service-store-frontend:dev .

# Ejecutar contenedor de desarrollo
docker run -d \
  --name service-store-frontend-dev \
  -p 5173:5173 \
  -v $(pwd):/app \
  -v /app/node_modules \
  --network enid_service_network \
  service-store-frontend:dev

# Ejecutar con Docker Compose (desarrollo)
docker-compose up -d

# Ver logs en tiempo real
docker-compose logs -f frontend

# Ejecutar comandos dentro del contenedor
docker-compose exec frontend npm run test
docker-compose exec frontend npm run lint
```

### **Comandos de Producción**

```bash
# ✅ CORRECTO: Comandos para producción

# Construir imagen de producción
docker build --target production -t service-store-frontend:prod .

# Ejecutar contenedor de producción
docker run -d \
  --name service-store-frontend-prod \
  -p 80:80 \
  --network enid_service_network \
  service-store-frontend:prod

# Ejecutar con Docker Compose (producción)
docker-compose -f docker-compose.prod.yml up -d

# Verificar estado del contenedor
docker ps
docker logs service-store-frontend-prod

# Health check
curl http://localhost/health
```

## 🔍 Debugging y Troubleshooting

### **Comandos de Debugging**

```bash
# ✅ CORRECTO: Comandos para debugging

# Entrar al contenedor en modo interactivo
docker-compose exec frontend sh

# Ver logs del contenedor
docker-compose logs frontend

# Ver logs con timestamps
docker-compose logs -t frontend

# Ver logs de los últimos N minutos
docker-compose logs --since="10m" frontend

# Ver uso de recursos
docker stats service-store-frontend-dev

# Ver información del contenedor
docker inspect service-store-frontend-dev
```

### **Solución de Problemas Comunes**

#### **1. Problema: Contenedor no inicia**
```bash
# Verificar logs de error
docker-compose logs frontend

# Verificar si el puerto está ocupado
netstat -tulpn | grep 5173

# Verificar permisos de archivos
ls -la
```

#### **2. Problema: Hot reload no funciona**
```bash
# Verificar montaje de volúmenes
docker inspect service-store-frontend-dev | grep Mounts

# Verificar permisos de archivos
docker-compose exec frontend ls -la /app

# Reiniciar contenedor
docker-compose restart frontend
```

#### **3. Problema: Dependencias no se instalan**
```bash
# Limpiar imagen y reconstruir
docker-compose down
docker system prune -f
docker-compose build --no-cache

# Verificar package.json
docker-compose exec frontend cat package.json
```

## 📊 Monitoreo y Logs

### **Configuración de Logs**

```yaml
# ✅ CORRECTO: Configuración de logs en docker-compose
version: '3'

services:
  frontend:
    # ... otras configuraciones ...
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    environment:
      - NODE_ENV=development
      - LOG_LEVEL=debug
```

### **Comandos de Monitoreo**

```bash
# ✅ CORRECTO: Comandos para monitoreo

# Ver logs en tiempo real
docker-compose logs -f --tail=100 frontend

# Ver logs de múltiples servicios
docker-compose logs -f frontend backend

# Exportar logs a archivo
docker-compose logs frontend > frontend-logs.txt

# Ver logs con filtro
docker-compose logs frontend | grep "ERROR"

# Ver logs de contenedor específico
docker logs service-store-frontend-dev
```

## 🔒 Seguridad

### **Configuraciones de Seguridad**

```dockerfile
# ✅ CORRECTO: Dockerfile con configuraciones de seguridad
FROM node:21.6.0-alpine

# Usar usuario no-root
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

WORKDIR /app

# Copiar archivos de dependencias
COPY package*.json ./

# Instalar dependencias
RUN npm ci --only=production --legacy-peer-deps

# Copiar código fuente
COPY . .

# Cambiar propietario de archivos
RUN chown -R nextjs:nodejs /app
USER nextjs

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

### **Configuración de Nginx con Seguridad**

```nginx
# ✅ CORRECTO: Nginx con configuraciones de seguridad
server {
    # ... configuración básica ...

    # Headers de seguridad
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https:;" always;

    # Limitar métodos HTTP
    if ($request_method !~ ^(GET|POST|HEAD|OPTIONS)$) {
        return 405;
    }

    # Limitar tamaño de archivos
    client_max_body_size 10M;

    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
}
```

## 🚀 CI/CD Integration

### **GitHub Actions Workflow**

```yaml
# ✅ CORRECTO: Workflow de CI/CD para Docker
name: Docker CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
      
    - name: Build and test Docker image
      run: |
        docker build -t service-store-frontend:test .
        docker run --rm service-store-frontend:test npm run test
        docker run --rm service-store-frontend:test npm run lint
        
    - name: Build production image
      run: |
        docker build --target production -t service-store-frontend:prod .
        
    - name: Push to registry
      if: github.ref == 'refs/heads/main'
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker tag service-store-frontend:prod ${{ secrets.REGISTRY }}/service-store-frontend:latest
        docker push ${{ secrets.REGISTRY }}/service-store-frontend:latest
```

### **Docker Hub Integration**

```bash
# ✅ CORRECTO: Comandos para Docker Hub

# Login a Docker Hub
docker login

# Tag de imagen
docker tag service-store-frontend:latest username/service-store-frontend:latest

# Push a Docker Hub
docker push username/service-store-frontend:latest

# Pull desde Docker Hub
docker pull username/service-store-frontend:latest
```

## 📋 Checklist de Docker

### ✅ **Antes de Construir**
- [ ] ¿El Dockerfile está optimizado para el target?
- [ ] ¿Las dependencias están correctamente especificadas?
- [ ] ¿Los volúmenes están configurados correctamente?
- [ ] ¿Las variables de entorno están definidas?

### ✅ **Durante el Build**
- [ ] ¿La imagen se construye sin errores?
- [ ] ¿El tamaño de la imagen es razonable?
- [ ] ¿Las dependencias se instalan correctamente?
- [ ] ¿Los archivos se copian en el orden correcto?

### ✅ **Después del Build**
- [ ] ¿El contenedor inicia correctamente?
- [ ] ¿La aplicación es accesible desde el host?
- [ ] ¿Los logs se muestran correctamente?
- [ ] ¿El hot reload funciona en desarrollo?

### ✅ **En Producción**
- [ ] ¿La imagen de producción es ligera?
- [ ] ¿Nginx está configurado correctamente?
- [ ] ¿Los headers de seguridad están configurados?
- [ ] ¿El reverse proxy funciona correctamente?

## 🔗 Recursos Adicionales

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Nginx Configuration](https://nginx.org/en/docs/)
- [Vue.js Deployment Guide](https://vuejs.org/guide/best-practices/production-deployment.html)
- [Docker Security Best Practices](https://docs.docker.com/develop/dev-best-practices/) 