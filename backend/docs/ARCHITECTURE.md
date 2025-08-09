# Arquitectura de Microservicios

## 🏗️ Estructura General

### Stack Tecnológico
- **Django**: Framework web
- **PostgreSQL**: Base de datos principal
- **Redis**: Cache y sesiones (solo cuando sea necesario)
- **Gunicorn**: Servidor WSGI
- **Watchmedo**: Auto-reload en desarrollo
- **Docker**: Containerización
- **Docker-compose**: Orquestación local

### Estructura de Microservicios
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

## 🐳 Docker y Containerización

### Comandos Principales
```bash
# Construir y ejecutar
docker-compose up --build

# Ver logs
docker-compose logs -f microservice_enid

# Ejecutar pruebas dentro del contenedor
docker-compose exec microservice_enid python manage.py test

# Acceder al shell
docker-compose exec microservice_enid python manage.py shell
```

### Dockerfile Estándar
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

### Docker-compose.yml
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

## 🔴 Redis - Criterios de Uso Elegante

### ¿Cuándo Incluir Redis?

**✅ INCLUIR Redis cuando:**
- **Cache de consultas frecuentes** que no cambian frecuentemente
- **Sesiones de usuario** que requieren alta disponibilidad
- **Rate limiting** para APIs
- **Colas de tareas** asíncronas
- **Datos temporales** que se acceden muy frecuentemente
- **Mejora de tiempos de respuesta** en endpoints críticos

**❌ NO incluir Redis cuando:**
- Los datos cambian constantemente
- No hay ganancia real en performance
- Añade complejidad innecesaria
- Los datos son pequeños y simples
- No hay patrones de acceso frecuente

### Configuración Elegante de Redis

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
asgiref==3.7.2
Django==4.2.7
djangorestframework==3.14.0
gunicorn==21.2.0
psycopg2-binary==2.9.9
Faker==20.1.0
watchdog==4.0.0
django-redis==5.4.0  # Solo si usas Redis
redis==5.0.1          # Solo si usas Redis
```

## 🧪 Testing Strategy

### Pruebas de Integración
- **Obligatorias** para cada desarrollo
- **Sin mocks** en pruebas unitarias
- **Datos reales** con Faker
- **Flujos completos** de funcionalidad

### Estructura de Tests
```
tests/
├── test_models.py
├── test_views.py
├── test_serializers.py
└── test_integration.py
```

### Ejecutar Tests en Contenedores
```bash
# Ejecutar todas las pruebas
docker-compose exec microservice_enid python manage.py test

# Ejecutar pruebas específicas
docker-compose exec microservice_enid python manage.py test [app_name].tests

# Ejecutar con coverage
docker-compose exec microservice_enid coverage run --source='.' manage.py test
docker-compose exec microservice_enid coverage report
```

## 📊 Base de Datos

### PostgreSQL
- **Migraciones** automáticas en entrypoint.sh
- **Backups** regulares
- **Índices** optimizados
- **Transacciones** atómicas

### Redis (solo si es necesario)
- **Cache** para consultas frecuentes
- **Sesiones** de usuario
- **Rate limiting** para APIs
- **Datos temporales** de alta frecuencia

### Comandos Django
```bash
# Crear migración
python manage.py makemigrations

# Aplicar migraciones
python manage.py migrate

# Crear superusuario
python manage.py createsuperuser

# Comando personalizado con Faker
python manage.py create_test_data
```

## 🔧 Variables de Entorno

### .env
```bash
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

## 🚀 Desarrollo Local

### Comandos Esenciales
```bash
# Iniciar servicios
docker-compose up --build

# Ver logs
docker-compose logs -f microservice_enid

# Ejecutar migraciones
docker-compose exec microservice_enid python manage.py migrate

# Crear superusuario
docker-compose exec microservice_enid python manage.py createsuperuser

# Ejecutar tests
docker-compose exec microservice_enid python manage.py test

# Acceder al shell
docker-compose exec microservice_enid python manage.py shell

# Verificar Redis (solo si lo usas)
docker-compose exec redis redis-cli ping
```

### Debugging
```bash
# Ver logs en tiempo real
docker-compose logs -f microservice_enid | grep ERROR

# Entrar al contenedor
docker-compose exec microservice_enid sh

# Ver procesos
docker-compose exec microservice_enid ps aux

# Ver variables de entorno
docker-compose exec microservice_enid env

# Acceder a Redis CLI (solo si lo usas)
docker-compose exec redis redis-cli
```

## 📋 Checklist de Arquitectura

### Antes de Implementar Redis
- [ ] ¿Realmente necesitas mejorar los tiempos de respuesta?
- [ ] ¿Los datos se acceden frecuentemente?
- [ ] ¿Los datos no cambian constantemente?
- [ ] ¿La ganancia en performance justifica la complejidad?
- [ ] ¿Has evaluado alternativas más simples?

### Configuración de Servicios
- [ ] ¿Configuraste PostgreSQL con health checks?
- [ ] ¿Configuraste Redis solo si es necesario?
- [ ] ¿Configuraste networks correctamente?
- [ ] ¿Incluiste variables de entorno apropiadas?
- [ ] ¿Configuraste volúmenes para persistencia? 