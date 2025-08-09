# Guía Docker y Microservicios

## 🐳 Docker Best Practices

### Estructura de Dockerfile
```dockerfile
# Utiliza Python 3.8 Alpine como imagen base
ARG PYTHON_VERSION=python:3.8-alpine
FROM ${PYTHON_VERSION}

RUN apk add --no-cache build-base

WORKDIR /app

COPY requirements.txt /app/
RUN pip install --upgrade pip && pip install -r requirements.txt

COPY . .

# Copia el script de entrada al contenedor
COPY entrypoint.sh /app/entrypoint.sh

# Asegura que el script tenga permisos de ejecución
RUN chmod +x /app/entrypoint.sh

EXPOSE 8080

# Configura el entrypoint para ejecutar las migraciones y levantar el servidor
ENTRYPOINT ["/app/entrypoint.sh"]
```

### Entrypoint.sh
```bash
#!/bin/sh
# Ejecuta las migraciones si es necesario
echo "Microservice [NOMBRE_SERVICIO]"
echo "Running makemigrations and migrate..."

python manage.py makemigrations
python manage.py makemigrations [app1] [app2] [app3]
python manage.py migrate

# Inicia el servidor con watchmedo
echo "Starting the server with watchmedo..."
watchmedo auto-restart --directory=./ --pattern=*.py --recursive -- gunicorn -b 0.0.0.0:8080 app.wsgi:application
```

### Docker-compose.yml Completo
```yaml
version: '3'

services:
  microservice_enid:
    build:
      context: .
      dockerfile: Dockerfile
    image: ${SERVICENAME}_service
    container_name: ${SERVICENAME}_service
    ports:
      - "8086:8080"
    env_file:
      - .env
    environment:
      - ENVIRONMENT=${ENVIRONMENT}
    depends_on:
      - postgres
      - redis  # Solo si es necesario
    volumes:
      - .:/app
    networks:
      - backend
      - enid_service_network

  postgres:
    container_name: ${SERVICENAME}_postgres
    image: postgres:alpine
    ports:
      - "5436:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    env_file:
      - .env
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      retries: 5

  # Redis - Solo cuando sea necesario para mejorar tiempos de respuesta
  redis:
    container_name: ${SERVICENAME}_redis
    image: redis:6-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      retries: 5
    command: redis-server --appendonly yes

volumes:
  postgres_data:
  redis_data:  # Solo si usas Redis

networks:
  backend:
    driver: bridge
  enid_service_network:
    external: true
```

## 🚀 Levantar Todos los Microservicios

### Alias local_up
Para levantar todos los microservicios de una sola vez (incluyendo frontend) para pruebas de integración:

```bash
# Levantar todos los microservicios
local_up

# Verificar estado de todos los servicios
docker-compose ps

# Ver logs de todos los servicios
docker-compose logs -f
```

### Script de Integración Completa
```bash
#!/bin/bash
# Script para levantar todo el ecosistema para pruebas de integración

echo "🚀 Levantando todos los microservicios para pruebas de integración..."

# Crear red si no existe
docker network create enid_service_network 2>/dev/null || true

# Levantar servicios backend
echo "📦 Levantando microservicios backend..."
cd /home/arithgrey/enid_service/services/service_leads && docker-compose up -d
cd /home/arithgrey/enid_service/services/service-oauth && docker-compose up -d
cd /home/arithgrey/enid_service/services/service_stock && docker-compose up -d
cd /home/arithgrey/enid_service/services/service_asistence && docker-compose up -d
cd /home/arithgrey/enid_service/services/service-references && docker-compose up -d
cd /home/arithgrey/enid_service/services/service-faqs && docker-compose up -d

# Levantar frontend
echo "🌐 Levantando frontend..."
cd /home/arithgrey/enid_service/services/service-store && docker-compose up -d

# Levantar proxy
echo "🔗 Levantando proxy..."
cd /home/arithgrey/enid_service/services/service_web && docker-compose up -d

echo "✅ Todos los servicios levantados para pruebas de integración"
echo "📊 URLs disponibles:"
echo "  - Frontend: http://localhost:5173"
echo "  - service_leads: http://localhost:8086"
echo "  - service-oauth: http://localhost:8087"
echo "  - service_stock: http://localhost:8088"
echo "  - service_asistence: http://localhost:8089"
echo "  - service-references: http://localhost:8090"
echo "  - service-faqs: http://localhost:8091"
echo "  - Swagger UI: http://localhost:8086/swagger/"
```

### Comandos de Verificación
```bash
# Verificar que todos los servicios estén corriendo
docker ps

# Ver logs de un servicio específico
docker-compose logs -f [service_name]

# Ver logs de todos los servicios
docker logs $(docker ps -q)

# Verificar conectividad entre servicios
curl http://localhost:8086/health/
curl http://localhost:8087/health/
curl http://localhost:8088/health/
```

## 🔴 Redis - Criterios de Uso

### ¿Cuándo Usar Redis?

**✅ USAR Redis cuando:**
- **Cache de consultas frecuentes** que no cambian frecuentemente
- **Sesiones de usuario** que requieren alta disponibilidad
- **Rate limiting** para APIs
- **Colas de tareas** asíncronas
- **Datos temporales** que se acceden muy frecuentemente
- **Mejora de tiempos de respuesta** en endpoints críticos

**❌ NO usar Redis cuando:**
- Los datos cambian constantemente
- No hay ganancia real en performance
- Añade complejidad innecesaria
- Los datos son pequeños y simples
- No hay patrones de acceso frecuente

### Configuración de Redis en Django

#### Settings.py
```python
# Solo si Redis es necesario para el microservicio
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://redis:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Para sesiones (solo si es necesario)
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_CACHE_ALIAS = 'default'
```

#### Requirements.txt (solo si usas Redis)
```txt
# ... otras dependencias
django-redis==5.4.0
redis==5.0.1
```

### Ejemplos de Uso Elegante

#### Cache de Consultas Frecuentes
```python
from django.core.cache import cache
from django_redis import get_redis_connection

class UserService:
    def get_user_profile(self, user_id):
        # Cache key único
        cache_key = f"user_profile_{user_id}"
        
        # Intentar obtener del cache
        cached_profile = cache.get(cache_key)
        if cached_profile:
            return cached_profile
        
        # Si no está en cache, obtener de DB
        profile = UserProfile.objects.get(id=user_id)
        
        # Guardar en cache por 1 hora
        cache.set(cache_key, profile, 3600)
        
        return profile
```

#### Rate Limiting
```python
from django.core.cache import cache
import time

def rate_limit(key, max_requests=100, window=3600):
    """Rate limiting elegante con Redis"""
    current_time = int(time.time())
    window_key = f"{key}:{current_time // window}"
    
    # Incrementar contador
    requests = cache.get(window_key, 0) + 1
    cache.set(window_key, requests, window)
    
    return requests <= max_requests
```

## 🚀 Comandos Docker Esenciales

### Desarrollo Local
```bash
# Construir y ejecutar todos los servicios
docker-compose up --build

# Ejecutar en background
docker-compose up -d

# Ver logs de un servicio específico
docker-compose logs -f microservice_enid

# Ver logs de Redis (solo si lo usas)
docker-compose logs -f redis

# Ver logs de todos los servicios
docker-compose logs -f

# Detener todos los servicios
docker-compose down

# Detener y eliminar volúmenes
docker-compose down -v
```

### Testing en Contenedores
```bash
# Ejecutar pruebas dentro del contenedor
docker-compose exec microservice_enid python manage.py test

# Ejecutar pruebas con coverage
docker-compose exec microservice_enid python -m pytest --cov=.

# Ejecutar pruebas específicas
docker-compose exec microservice_enid python manage.py test [app_name].tests

# Ejecutar linting
docker-compose exec microservice_enid flake8 .

# Ejecutar migraciones
docker-compose exec microservice_enid python manage.py migrate
```

### Debugging y Desarrollo
```bash
# Acceder al shell de Django
docker-compose exec microservice_enid python manage.py shell

# Crear superusuario
docker-compose exec microservice_enid python manage.py createsuperuser

# Ejecutar comandos personalizados
docker-compose exec microservice_enid python manage.py create_test_data

# Ver logs en tiempo real
docker-compose logs -f microservice_enid | grep ERROR

# Reconstruir solo un servicio
docker-compose build microservice_enid
docker-compose up microservice_enid

# Acceder a Redis CLI (solo si lo usas)
docker-compose exec redis redis-cli
```

## 🧪 Testing en Contenedores

### Configuración de Tests
```python
# settings/test.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'test_db',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'postgres',
        'PORT': '5432',
    }
}

# Redis para cache en tests (solo si es necesario)
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://redis:6379/1',
    }
}
```

### Ejecutar Tests Completo
```bash
# Script para ejecutar todos los tests
#!/bin/bash
echo "Running tests in Docker containers..."

# Ejecutar migraciones
docker-compose exec microservice_enid python manage.py migrate

# Ejecutar tests
docker-compose exec microservice_enid python manage.py test --verbosity=2

# Ejecutar coverage
docker-compose exec microservice_enid coverage run --source='.' manage.py test
docker-compose exec microservice_enid coverage report
docker-compose exec microservice_enid coverage html
```

## 📊 Monitoreo y Logs

### Ver Logs por Servicio
```bash
# Logs del servicio web
docker-compose logs microservice_enid

# Logs de la base de datos
docker-compose logs postgres

# Logs de Redis (solo si lo usas)
docker-compose logs redis

# Logs con timestamps
docker-compose logs -t microservice_enid

# Últimas 100 líneas
docker-compose logs --tail=100 microservice_enid

# Ver logs de watchmedo
docker-compose logs -f microservice_enid | grep watchmedo
```

### Debugging Avanzado
```bash
# Entrar al contenedor
docker-compose exec microservice_enid sh

# Ver procesos dentro del contenedor
docker-compose exec microservice_enid ps aux

# Ver uso de recursos
docker stats

# Inspeccionar contenedor
docker-compose exec microservice_enid env

# Verificar Redis (solo si lo usas)
docker-compose exec redis redis-cli ping
```

## 🔧 Configuración de Entorno

### Variables de Entorno (.env)
```bash
# .env
ENVIRONMENT=development
SERVICENAME=service_leads
DEBUG=True
DATABASE_URL=postgresql://postgres:postgres@postgres:5432/dbname
REDIS_URL=redis://redis:6379/0  # Solo si usas Redis
SECRET_KEY=your-secret-key
ALLOWED_HOSTS=localhost,127.0.0.1
```

### Requirements.txt
```txt
asgiref==3.7.2
Django==4.2.7
djangorestframework==3.14.0
gunicorn==21.2.0
psycopg2-binary==2.9.9
Faker==20.1.0
watchdog==4.0.0
django-redis==5.4.0  # Solo si usas Redis
redis==5.0.1          # Solo si usas Redis
# ... otras dependencias
```

## 🚀 Comandos de Producción

### Construir para Producción
```bash
# Construir imagen de producción
docker build -t myapp:prod .

# Ejecutar en producción
docker run -d -p 8080:8080 myapp:prod

# Usar docker-compose para producción
docker-compose -f docker-compose.prod.yml up -d
```

### Backup y Restore
```bash
# Backup de base de datos
docker-compose exec postgres pg_dump -U postgres dbname > backup.sql

# Restore de base de datos
docker-compose exec -T postgres psql -U postgres dbname < backup.sql

# Backup de Redis (solo si lo usas)
docker-compose exec redis redis-cli BGSAVE

# Backup de volúmenes
docker run --rm -v [project_name]_postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/postgres_backup.tar.gz -C /data .
```

## 📋 Estructura de Microservicio

```
microservice/
├── app/                    # Aplicación Django principal
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── [app1]/                # Apps Django específicas
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── tests/
├── [app2]/
├── manage.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
├── entrypoint.sh
└── .env
```

## 📋 Checklist Docker

- [ ] ¿Usas Python 3.8-alpine como base?
- [ ] ¿Incluiste entrypoint.sh con watchmedo?
- [ ] ¿Configuraste variables de entorno con .env?
- [ ] ¿Incluiste health checks para postgres?
- [ ] ¿Configuraste networks correctamente?
- [ ] ¿Ejecutas tests en contenedores?
- [ ] ¿Tienes backup strategy?
- [ ] ¿Documentaste comandos esenciales?
- [ ] ¿Configuraste gunicorn con watchmedo?
- [ ] ¿Incluiste migraciones en entrypoint.sh?
- [ ] ¿Evaluaste si Redis es realmente necesario?
- [ ] ¿Configuraste Redis solo para casos específicos?
- [ ] ¿Configuraste el alias local_up para integración?
- [ ] ¿Verificaste conectividad entre microservicios? 