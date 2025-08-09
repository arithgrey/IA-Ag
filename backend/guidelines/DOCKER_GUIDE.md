# Guía Docker y Microservicios

## 🐳 Docker Best Practices

### Estructura de Dockerfile
```dockerfile
# Usar imagen oficial de Python
FROM python:3.11-slim

# Establecer directorio de trabajo
WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copiar requirements primero (para cache)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código de la aplicación
COPY . .

# Exponer puerto
EXPOSE 8000

# Comando por defecto
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### Docker-compose.yml Completo
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/dbname
      - REDIS_URL=redis://redis:6379/0
      - DEBUG=True
    volumes:
      - .:/app
    command: python manage.py runserver 0.0.0.0:8000

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=dbname
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - web

volumes:
  postgres_data:
```

## 🚀 Comandos Docker Esenciales

### Desarrollo Local
```bash
# Construir y ejecutar todos los servicios
docker-compose up --build

# Ejecutar en background
docker-compose up -d

# Ver logs de un servicio específico
docker-compose logs -f web

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
docker-compose exec web python manage.py test

# Ejecutar pruebas con coverage
docker-compose exec web python -m pytest --cov=.

# Ejecutar pruebas específicas
docker-compose exec web python manage.py test users.tests

# Ejecutar linting
docker-compose exec web flake8 .

# Ejecutar migraciones
docker-compose exec web python manage.py migrate
```

### Debugging y Desarrollo
```bash
# Acceder al shell de Django
docker-compose exec web python manage.py shell

# Crear superusuario
docker-compose exec web python manage.py createsuperuser

# Ejecutar comandos personalizados
docker-compose exec web python manage.py create_test_data

# Ver logs en tiempo real
docker-compose logs -f web | grep ERROR

# Reconstruir solo un servicio
docker-compose build web
docker-compose up web
```

## 🧪 Testing en Contenedores

### Configuración de Tests
```python
# settings/test.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'test_db',
        'USER': 'user',
        'PASSWORD': 'pass',
        'HOST': 'db',
        'PORT': '5432',
    }
}

# Usar Redis para cache en tests
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
docker-compose exec web python manage.py migrate

# Ejecutar tests
docker-compose exec web python manage.py test --verbosity=2

# Ejecutar coverage
docker-compose exec web coverage run --source='.' manage.py test
docker-compose exec web coverage report
docker-compose exec web coverage html
```

## 📊 Monitoreo y Logs

### Ver Logs por Servicio
```bash
# Logs del servicio web
docker-compose logs web

# Logs de la base de datos
docker-compose logs db

# Logs de Redis
docker-compose logs redis

# Logs con timestamps
docker-compose logs -t web

# Últimas 100 líneas
docker-compose logs --tail=100 web
```

### Debugging Avanzado
```bash
# Entrar al contenedor
docker-compose exec web bash

# Ver procesos dentro del contenedor
docker-compose exec web ps aux

# Ver uso de recursos
docker stats

# Inspeccionar contenedor
docker-compose exec web env
```

## 🔧 Configuración de Entorno

### Variables de Entorno
```bash
# .env
DEBUG=True
DATABASE_URL=postgresql://user:pass@db:5432/dbname
REDIS_URL=redis://redis:6379/0
SECRET_KEY=your-secret-key
ALLOWED_HOSTS=localhost,127.0.0.1
```

### Docker-compose con .env
```yaml
version: '3.8'
services:
  web:
    build: .
    environment:
      - DEBUG=${DEBUG}
      - DATABASE_URL=${DATABASE_URL}
      - SECRET_KEY=${SECRET_KEY}
    env_file:
      - .env
```

## 🚀 Comandos de Producción

### Construir para Producción
```bash
# Construir imagen de producción
docker build -t myapp:prod .

# Ejecutar en producción
docker run -d -p 8000:8000 myapp:prod

# Usar docker-compose para producción
docker-compose -f docker-compose.prod.yml up -d
```

### Backup y Restore
```bash
# Backup de base de datos
docker-compose exec db pg_dump -U user dbname > backup.sql

# Restore de base de datos
docker-compose exec -T db psql -U user dbname < backup.sql

# Backup de volúmenes
docker run --rm -v myapp_postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/postgres_backup.tar.gz -C /data .
```

## 📋 Checklist Docker

- [ ] ¿Usas imágenes oficiales?
- [ ] ¿Optimizaste el Dockerfile para cache?
- [ ] ¿Configuraste variables de entorno?
- [ ] ¿Incluiste health checks?
- [ ] ¿Configuraste logs apropiadamente?
- [ ] ¿Usas volúmenes para datos persistentes?
- [ ] ¿Ejecutas tests en contenedores?
- [ ] ¿Tienes backup strategy?
- [ ] ¿Documentaste comandos esenciales?
- [ ] ¿Configuraste nginx para producción? 