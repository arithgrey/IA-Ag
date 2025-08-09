# Principios de Desarrollo Backend

## 🎯 Principios Fundamentales

### 1. Principios SOLID
Todos los desarrollos **DEBEN** seguir los principios SOLID:

- **S** - Single Responsibility Principle: Cada clase tiene una sola responsabilidad
- **O** - Open/Closed Principle: Abierto para extensión, cerrado para modificación
- **L** - Liskov Substitution Principle: Las subclases pueden sustituir a sus clases base
- **I** - Interface Segregation Principle: Interfaces específicas en lugar de interfaces generales
- **D** - Dependency Inversion Principle: Depender de abstracciones, no de implementaciones

### 2. Pensamiento Cuidadoso y Soluciones Concisas
- **Analiza cuidadosamente** el problema antes de implementar
- **Implementa únicamente** la tarea específica solicitada
- **Busca la solución más elegante** y concisa
- **Minimiza los cambios** de código existente
- **Prioriza la simplicidad** sobre la complejidad

### 3. DRY (Don't Repeat Yourself)
- **NUNCA** repitas código
- **Reutiliza** componentes existentes
- **Crea abstracciones** cuando sea necesario
- **Usa herencia y composición** apropiadamente
- **Mantén la consistencia** en patrones similares

### 4. TDD (Test-Driven Development)
- **Desarrolla con TDD** en todos los proyectos
- **NO uses mocks** a nivel de pruebas unitarias
- **Escribe pruebas primero**, luego implementa
- **Refactoriza** constantemente
- **Mantén cobertura alta** de pruebas

### 5. Pruebas de Integración Obligatorias
- **Cada desarrollo DEBE** incluir pruebas de integración
- **Prueba flujos completos** de funcionalidad
- **Verifica interacciones** entre componentes
- **Usa datos reales** en pruebas de integración
- **Documenta casos de prueba** importantes

### 6. Faker y Comandos Django
- **Usa Faker** para generar datos de prueba
- **Aprovecha los comandos** de Django para crear datos
- **Mantén datos de prueba** consistentes
- **Usa factories** para crear objetos de prueba
- **Separa datos de prueba** por contexto

### 7. Documentación Limitada
- **Documenta solo lo esencial**
- **Evita archivos .md** excesivos
- **Mantén documentación** en `docs/`
- **Usa comentarios en código** cuando sea necesario
- **Prioriza código autodocumentado**

### 8. Stack Tecnológico
- **Django**: Framework principal
- **PostgreSQL**: Base de datos
- **Docker**: Containerización
- **Docker-compose**: Orquestación
- **Microservicios**: Arquitectura distribuida

### 9. Evitar Repositories
- **NO uses patrones Repository**
- **Aprovecha Serializers** de Django
- **Reutiliza código** existente
- **Mantén simplicidad** en la arquitectura
- **Usa ORM de Django** directamente

### 10. Documentación de APIs con Swagger/OpenAPI
- **TODAS las APIs DEBEN** estar documentadas con Swagger
- **Usa drf-yasg** para generar documentación automática
- **Incluye ejemplos** de request/response
- **Documenta parámetros** y códigos de respuesta
- **Mantén documentación actualizada** con cada cambio

## 🚀 Implementación Práctica

### Ejemplo de Aplicación TDD:
```python
# 1. Escribir prueba primero
def test_user_creation():
    user_data = {
        'email': fake.email(),
        'name': fake.name()
    }
    user = User.objects.create(**user_data)
    assert user.email == user_data['email']

# 2. Implementar funcionalidad
# 3. Refactorizar
# 4. Repetir
```

### Ejemplo de DRY:
```python
# ❌ Mal - Repetir código
def create_user_v1():
    return User.objects.create(email="test@test.com")

def create_user_v2():
    return User.objects.create(email="test@test.com")

# ✅ Bien - Reutilizar
def create_test_user(email="test@test.com"):
    return User.objects.create(email=email)
```

### Ejemplo de Serializers (sin Repository):
```python
# ✅ Usar Serializers de Django
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'email', 'name']
    
    def create(self, validated_data):
        return User.objects.create(**validated_data)

# Uso en ViewSet
class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

### Ejemplo de Documentación Swagger:
```python
from drf_yasg import openapi
from drf_yasg.utils import swagger_auto_schema
from rest_framework import status

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    
    @swagger_auto_schema(
        operation_description="Crear un nuevo usuario",
        request_body=UserSerializer,
        responses={
            201: openapi.Response(
                description="Usuario creado exitosamente",
                schema=UserSerializer
            ),
            400: openapi.Response(
                description="Datos inválidos",
                schema=openapi.Schema(
                    type=openapi.TYPE_OBJECT,
                    properties={
                        'error': openapi.Schema(type=openapi.TYPE_STRING)
                    }
                )
            )
        },
        examples=[
            openapi.Example(
                "Ejemplo de creación",
                value={
                    "email": "usuario@ejemplo.com",
                    "name": "Juan Pérez"
                }
            )
        ]
    )
    def create(self, request, *args, **kwargs):
        return super().create(request, *args, **kwargs)
```

### Configuración Swagger en settings.py:
```python
INSTALLED_APPS = [
    # ... otras apps
    'drf_yasg',
]

# Configuración de Swagger
SWAGGER_SETTINGS = {
    'SECURITY_DEFINITIONS': {
        'Bearer': {
            'type': 'apiKey',
            'name': 'Authorization',
            'in': 'header'
        }
    },
    'USE_SESSION_AUTH': False,
    'JSON_EDITOR': True,
}
```

### URLs para Swagger:
```python
from django.urls import path, re_path
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from rest_framework import permissions

schema_view = get_schema_view(
    openapi.Info(
        title="API de Microservicio",
        default_version='v1',
        description="Documentación de la API",
        terms_of_service="https://www.google.com/policies/terms/",
        contact=openapi.Contact(email="contact@ejemplo.com"),
        license=openapi.License(name="BSD License"),
    ),
    public=True,
    permission_classes=(permissions.AllowAny,),
)

urlpatterns = [
    # ... otras URLs
    re_path(r'^swagger(?P<format>\.json|\.yaml)$', 
            schema_view.without_ui(cache_timeout=0), name='schema-json'),
    path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), 
         name='schema-swagger-ui'),
    path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), 
         name='schema-redoc'),
]
``` 