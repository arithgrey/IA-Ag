# Índice de Contexto para Agentes de IA

## 🎯 Propósito
Este índice sirve como punto de entrada para que los agentes de IA encuentren rápidamente la información necesaria para implementar soluciones siguiendo nuestras mejores prácticas.

## 📚 Documentación Principal

### Principios Fundamentales
- **[backend/PRINCIPLES.md](backend/PRINCIPLES.md)** - Principios SOLID, DRY, TDD y stack tecnológico
- **[patterns/SOLID_PATTERNS.md](patterns/SOLID_PATTERNS.md)** - Ejemplos prácticos de SOLID en Django

### Guías Específicas
- **[guidelines/TDD_GUIDE.md](guidelines/TDD_GUIDE.md)** - Guía completa de TDD sin mocks
- **[guidelines/DOCKER_GUIDE.md](guidelines/DOCKER_GUIDE.md)** - Docker y microservicios
- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** - Arquitectura de microservicios

## 🔍 Búsqueda Rápida por Tema

### Para Nuevas Implementaciones
1. **Revisar principios** → `backend/PRINCIPLES.md`
2. **Aplicar SOLID** → `patterns/SOLID_PATTERNS.md`
3. **Implementar TDD** → `guidelines/TDD_GUIDE.md`
4. **Configurar Docker** → `guidelines/DOCKER_GUIDE.md`

### Para Testing
- **TDD sin mocks** → `guidelines/TDD_GUIDE.md`
- **Pruebas de integración** → `guidelines/TDD_GUIDE.md#pruebas-de-integración`
- **Faker y datos de prueba** → `guidelines/TDD_GUIDE.md#comandos-django-con-faker`

### Para Docker
- **Comandos esenciales** → `guidelines/DOCKER_GUIDE.md#comandos-docker-esenciales`
- **Testing en contenedores** → `guidelines/DOCKER_GUIDE.md#testing-en-contenedores`
- **Debugging** → `guidelines/DOCKER_GUIDE.md#debugging-y-desarrollo`

### Para Arquitectura
- **Estructura de microservicios** → `docs/ARCHITECTURE.md`
- **Stack tecnológico** → `docs/ARCHITECTURE.md#stack-tecnológico`
- **Base de datos** → `docs/ARCHITECTURE.md#base-de-datos`

## 🚀 Checklist de Implementación

### Antes de Empezar
- [ ] ¿Entendiste los principios SOLID?
- [ ] ¿Planeaste la solución más concisa?
- [ ] ¿Identificaste oportunidades para DRY?
- [ ] ¿Preparaste el entorno Docker?

### Durante el Desarrollo
- [ ] ¿Escribiste pruebas primero (TDD)?
- [ ] ¿Evitaste repetir código?
- [ ] ¿Usaste Serializers en lugar de Repositories?
- [ ] ¿Incluiste pruebas de integración?

### Antes de Finalizar
- [ ] ¿Ejecutaste tests en contenedores?
- [ ] ¿Verificaste que no hay mocks?
- [ ] ¿Documentaste solo lo esencial?
- [ ] ¿Revisaste logs del contenedor?

## 💡 Consejos Rápidos

### Código
- **DRY**: Reutiliza, no repitas
- **SOLID**: Cada clase una responsabilidad
- **TDD**: Prueba primero, implementa después
- **Serializers**: Usa Django Serializers, no Repositories

### Testing
- **Sin mocks**: Usa datos reales con Faker
- **Integración**: Prueba flujos completos
- **Contenedores**: Ejecuta tests dentro de Docker

### Docker
- **Logs**: `docker-compose logs -f [service]`
- **Tests**: `docker-compose exec web python manage.py test`
- **Shell**: `docker-compose exec web python manage.py shell`

## 🔗 Referencias Rápidas

### Comandos Docker Esenciales
```bash
# Desarrollo
docker-compose up --build
docker-compose logs -f web

# Testing
docker-compose exec web python manage.py test
docker-compose exec web python manage.py shell

# Debugging
docker-compose exec web python manage.py createsuperuser
docker-compose exec web python manage.py create_test_data
```

### Estructura de Archivos
```
microservice/
├── app/
│   ├── models.py      # Modelos Django
│   ├── serializers.py # Serializers (NO Repositories)
│   ├── views.py       # Views/ViewSets
│   └── tests/         # Pruebas TDD
├── docker-compose.yml # Orquestación
├── Dockerfile         # Containerización
└── requirements.txt   # Dependencias
```

## 📞 Recordatorios Importantes

1. **SIEMPRE** aplica DRY - nunca repitas código
2. **SIEMPRE** usa TDD - escribe pruebas primero
3. **SIEMPRE** incluye pruebas de integración
4. **NUNCA** uses mocks en pruebas unitarias
5. **NUNCA** uses patrones Repository - usa Serializers
6. **SIEMPRE** ejecuta tests en contenedores
7. **SIEMPRE** revisa logs del contenedor para debugging
8. **MANTÉN** documentación limitada y esencial 