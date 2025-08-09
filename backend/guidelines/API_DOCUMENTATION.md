# Guía de Documentación de APIs con Swagger/OpenAPI

## 🎯 Propósito
Esta guía establece los estándares para documentar todas las APIs del sistema Enid usando Swagger/OpenAPI para mantener consistencia y facilidad de uso.

## 📋 Principios de Documentación

### Reglas Fundamentales
- **TODAS las APIs DEBEN** estar documentadas con Swagger
- **Documentación automática** usando drf-yasg
- **Ejemplos claros** de request/response
- **Códigos de respuesta** documentados
- **Parámetros explicados** detalladamente
- **Mantener actualizada** con cada cambio

## 🚀 Configuración Inicial

### Instalación de Dependencias
```python
# requirements.txt
drf-yasg==1.21.7
```

### Configuración en settings.py
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
    'SUPPORTED_SUBMIT_METHODS': [
        'get',
        'post',
        'put',
        'delete',
        'patch'
    ],
}
```

### URLs para Swagger
```python
# urls.py
from django.urls import path, re_path
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from rest_framework import permissions

schema_view = get_schema_view(
    openapi.Info(
        title="API de Microservicio",
        default_version='v1',
        description="Documentación completa de la API",
        terms_of_service="https://www.ejemplo.com/terms/",
        contact=openapi.Contact(email="dev@ejemplo.com"),
        license=openapi.License(name="MIT License"),
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

## 📝 Documentación de ViewSets

### ViewSet Básico con Documentación
```python
from drf_yasg import openapi
from drf_yasg.utils import swagger_auto_schema
from rest_framework import viewsets, status
from rest_framework.decorators import action

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    
    @swagger_auto_schema(
        operation_description="Listar todos los usuarios",
        responses={
            200: openapi.Response(
                description="Lista de usuarios",
                schema=UserSerializer(many=True)
            ),
            401: openapi.Response(
                description="No autorizado"
            )
        },
        manual_parameters=[
            openapi.Parameter(
                'page',
                openapi.IN_QUERY,
                description="Número de página",
                type=openapi.TYPE_INTEGER
            ),
            openapi.Parameter(
                'page_size',
                openapi.IN_QUERY,
                description="Elementos por página",
                type=openapi.TYPE_INTEGER
            )
        ]
    )
    def list(self, request, *args, **kwargs):
        return super().list(request, *args, **kwargs)
    
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
                        'error': openapi.Schema(type=openapi.TYPE_STRING),
                        'field_errors': openapi.Schema(type=openapi.TYPE_OBJECT)
                    }
                )
            )
        },
        examples=[
            openapi.Example(
                "Ejemplo de creación",
                value={
                    "email": "usuario@ejemplo.com",
                    "name": "Juan Pérez",
                    "phone": "+1234567890"
                }
            )
        ]
    )
    def create(self, request, *args, **kwargs):
        return super().create(request, *args, **kwargs)
    
    @swagger_auto_schema(
        operation_description="Obtener un usuario específico",
        responses={
            200: openapi.Response(
                description="Usuario encontrado",
                schema=UserSerializer
            ),
            404: openapi.Response(
                description="Usuario no encontrado"
            )
        }
    )
    def retrieve(self, request, *args, **kwargs):
        return super().retrieve(request, *args, **kwargs)
```

### Acciones Personalizadas con Documentación
```python
class LeadViewSet(viewsets.ModelViewSet):
    queryset = Lead.objects.all()
    serializer_class = LeadSerializer
    
    @swagger_auto_schema(
        operation_description="Buscar leads por criterios",
        manual_parameters=[
            openapi.Parameter(
                'status',
                openapi.IN_QUERY,
                description="Estado del lead",
                type=openapi.TYPE_STRING,
                enum=['pending', 'contacted', 'converted', 'lost']
            ),
            openapi.Parameter(
                'date_from',
                openapi.IN_QUERY,
                description="Fecha desde (YYYY-MM-DD)",
                type=openapi.TYPE_STRING,
                format=openapi.FORMAT_DATE
            ),
            openapi.Parameter(
                'date_to',
                openapi.IN_QUERY,
                description="Fecha hasta (YYYY-MM-DD)",
                type=openapi.TYPE_STRING,
                format=openapi.FORMAT_DATE
            )
        ],
        responses={
            200: openapi.Response(
                description="Leads encontrados",
                schema=LeadSerializer(many=True)
            ),
            400: openapi.Response(
                description="Parámetros inválidos"
            )
        }
    )
    @action(detail=False, methods=['get'])
    def search(self, request):
        # Lógica de búsqueda
        pass
```

## 🔐 Documentación de Autenticación

### Configuración de Seguridad
```python
from drf_yasg import openapi

# En el ViewSet
@swagger_auto_schema(
    operation_description="Endpoint protegido",
    security=[{'Bearer': []}],
    responses={
        200: openapi.Response(description="Acceso autorizado"),
        401: openapi.Response(description="Token inválido"),
        403: openapi.Response(description="Sin permisos")
    }
)
def protected_endpoint(self, request):
    pass
```

### Ejemplo de Autenticación JWT
```python
class SecureViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    
    @swagger_auto_schema(
        operation_description="Datos sensibles del usuario",
        security=[{'Bearer': []}],
        responses={
            200: UserDataSerializer,
            401: openapi.Response(
                description="Token JWT requerido",
                schema=openapi.Schema(
                    type=openapi.TYPE_OBJECT,
                    properties={
                        'detail': openapi.Schema(
                            type=openapi.TYPE_STRING,
                            example="Las credenciales de autenticación no se proporcionaron."
                        )
                    }
                )
            )
        }
    )
    def list(self, request, *args, **kwargs):
        return super().list(request, *args, **kwargs)
```

## 📊 Documentación de Serializers

### Serializer con Documentación
```python
from drf_yasg import openapi

class LeadSerializer(serializers.ModelSerializer):
    class Meta:
        model = Lead
        fields = ['id', 'name', 'email', 'phone', 'status', 'created_at']
    
    def to_representation(self, instance):
        """Documentación del serializer"""
        return super().to_representation(instance)

# En el ViewSet
@swagger_auto_schema(
    operation_description="Crear nuevo lead",
    request_body=LeadSerializer,
    responses={
        201: LeadSerializer,
        400: openapi.Response(
            description="Datos inválidos",
            schema=openapi.Schema(
                type=openapi.TYPE_OBJECT,
                properties={
                    'name': openapi.Schema(
                        type=openapi.TYPE_ARRAY,
                        items=openapi.Schema(type=openapi.TYPE_STRING),
                        description="Errores de validación del nombre"
                    ),
                    'email': openapi.Schema(
                        type=openapi.TYPE_ARRAY,
                        items=openapi.Schema(type=openapi.TYPE_STRING),
                        description="Errores de validación del email"
                    )
                }
            )
        )
    }
)
def create(self, request, *args, **kwargs):
    return super().create(request, *args, **kwargs)
```

## 🧪 Testing de Documentación

### Tests para Verificar Documentación
```python
from django.test import TestCase
from django.urls import reverse
from rest_framework.test import APIClient

class SwaggerDocumentationTest(TestCase):
    def setUp(self):
        self.client = APIClient()
    
    def test_swagger_ui_accessible(self):
        """Verificar que Swagger UI es accesible"""
        response = self.client.get(reverse('schema-swagger-ui'))
        self.assertEqual(response.status_code, 200)
    
    def test_swagger_json_accessible(self):
        """Verificar que el JSON de Swagger es accesible"""
        response = self.client.get(reverse('schema-json'))
        self.assertEqual(response.status_code, 200)
        self.assertIn('openapi', response.json())
    
    def test_api_endpoints_documented(self):
        """Verificar que los endpoints están documentados"""
        response = self.client.get(reverse('schema-json'))
        swagger_data = response.json()
        
        # Verificar que los paths están documentados
        self.assertIn('/api/users/', swagger_data['paths'])
        self.assertIn('/api/leads/', swagger_data['paths'])
```

## 📋 Checklist de Documentación

### Antes de Implementar
- [ ] ¿Incluiste drf-yasg en requirements.txt?
- [ ] ¿Configuraste SWAGGER_SETTINGS en settings.py?
- [ ] ¿Agregaste las URLs de Swagger?
- [ ] ¿Planificaste la estructura de documentación?

### Durante el Desarrollo
- [ ] ¿Documentaste cada endpoint con @swagger_auto_schema?
- [ ] ¿Incluiste ejemplos de request/response?
- [ ] ¿Documentaste códigos de respuesta?
- [ ] ¿Explicaste parámetros de query?
- [ ] ¿Configuraste autenticación si es necesario?

### Antes de Finalizar
- [ ] ¿Verificaste que Swagger UI es accesible?
- [ ] ¿Probaste los ejemplos en la documentación?
- [ ] ¿Actualizaste documentación con cambios?
- [ ] ¿Incluiste tests para verificar documentación?

## 🚀 Comandos Útiles

### Verificar Documentación
```bash
# Acceder a Swagger UI
curl http://localhost:8086/swagger/

# Obtener JSON de Swagger
curl http://localhost:8086/swagger.json

# Verificar que el JSON es válido
python -m json.tool swagger.json
```

### Generar Documentación
```bash
# Verificar que drf-yasg está instalado
docker-compose exec microservice_enid pip list | grep drf-yasg

# Reiniciar servicio para aplicar cambios
docker-compose restart microservice_enid

# Ver logs para errores de documentación
docker-compose logs -f microservice_enid | grep swagger
```

## 💡 Mejores Prácticas

### Documentación Clara
- **Usa descripciones concisas** pero informativas
- **Incluye ejemplos reales** de uso
- **Documenta errores comunes** y sus soluciones
- **Mantén consistencia** en el formato

### Ejemplos Útiles
- **Proporciona ejemplos completos** de request/response
- **Incluye casos edge** y manejo de errores
- **Documenta parámetros opcionales** claramente
- **Explica códigos de respuesta** específicos

### Mantenimiento
- **Actualiza documentación** con cada cambio de API
- **Verifica que ejemplos funcionan** en producción
- **Revisa documentación** en code reviews
- **Prueba endpoints** desde Swagger UI 