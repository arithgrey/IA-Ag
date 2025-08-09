# Arquitectura de Microservicios

## 🏗️ Estructura General

### Stack Tecnológico
- **Django**: Framework web
- **PostgreSQL**: Base de datos principal
- **Docker**: Containerización
- **Docker-compose**: Orquestación local

### Estructura de Microservicios
```
microservice/
├── app/
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── tests/
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```

## 🐳 Docker y Containerización

### Comandos Principales
```bash
# Construir y ejecutar
docker-compose up --build

# Ver logs
docker-compose logs -f [service_name]

# Ejecutar pruebas dentro del contenedor
docker-compose exec [service_name] python manage.py test

# Acceder al shell
docker-compose exec [service_name] python manage.py shell
```

### Docker-compose.yml Ejemplo
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/dbname
  
  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=dbname
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
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

## 📊 Base de Datos

### PostgreSQL
- **Migraciones** automáticas
- **Backups** regulares
- **Índices** optimizados
- **Transacciones** atómicas

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