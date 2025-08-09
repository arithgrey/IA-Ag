# Guía TDD sin Mocks

## 🎯 Principios TDD

### Ciclo Red-Green-Refactor
1. **Red**: Escribir prueba que falle
2. **Green**: Implementar código mínimo para pasar
3. **Refactor**: Mejorar código sin cambiar comportamiento

### Reglas Importantes
- **NO uses mocks** en pruebas unitarias
- **Usa datos reales** con Faker
- **Prueba comportamiento**, no implementación
- **Mantén pruebas simples** y legibles

## 🚀 Ejemplo Práctico: UserService

### Paso 1: Escribir Primera Prueba
```python
# tests/test_user_service.py
from faker import Faker
import pytest

fake = Faker()

class TestUserService:
    def test_create_user_with_valid_data(self):
        # Arrange
        service = UserService()
        user_data = {
            'email': fake.email(),
            'name': fake.name()
        }
        
        # Act
        user = service.create_user(user_data)
        
        # Assert
        assert user.email == user_data['email']
        assert user.name == user_data['name']
        assert user.id is not None
```

### Paso 2: Implementar Código Mínimo
```python
# services/user_service.py
class UserService:
    def create_user(self, data):
        return User.objects.create(**data)
```

### Paso 3: Agregar Más Pruebas
```python
def test_create_user_without_email_raises_error(self):
    # Arrange
    service = UserService()
    user_data = {'name': fake.name()}
    
    # Act & Assert
    with pytest.raises(ValueError, match="Email is required"):
        service.create_user(user_data)

def test_create_user_with_invalid_email_raises_error(self):
    # Arrange
    service = UserService()
    user_data = {
        'email': 'invalid-email',
        'name': fake.name()
    }
    
    # Act & Assert
    with pytest.raises(ValueError, match="Invalid email format"):
        service.create_user(user_data)
```

### Paso 4: Refactorizar Implementación
```python
import re
from django.core.exceptions import ValidationError

class UserService:
    def create_user(self, data):
        self._validate_user_data(data)
        return User.objects.create(**data)
    
    def _validate_user_data(self, data):
        if not data.get('email'):
            raise ValueError("Email is required")
        
        if not self._is_valid_email(data['email']):
            raise ValueError("Invalid email format")
    
    def _is_valid_email(self, email):
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        return re.match(pattern, email) is not None
```

## 🧪 Pruebas de Integración

### Ejemplo: API Endpoint
```python
# tests/test_user_api.py
from django.test import TestCase
from django.urls import reverse
from rest_framework.test import APIClient
from faker import Faker

fake = Faker()

class TestUserAPI(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.url = reverse('user-list')
    
    def test_create_user_via_api(self):
        # Arrange
        user_data = {
            'email': fake.email(),
            'name': fake.name()
        }
        
        # Act
        response = self.client.post(self.url, user_data, format='json')
        
        # Assert
        self.assertEqual(response.status_code, 201)
        self.assertEqual(response.data['email'], user_data['email'])
        self.assertEqual(response.data['name'], user_data['name'])
    
    def test_get_user_list(self):
        # Arrange
        user1 = User.objects.create(
            email=fake.email(),
            name=fake.name()
        )
        user2 = User.objects.create(
            email=fake.email(),
            name=fake.name()
        )
        
        # Act
        response = self.client.get(self.url)
        
        # Assert
        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(response.data), 2)
```

## 📊 Comandos Django con Faker

### Crear Comando Personalizado
```python
# management/commands/create_test_data.py
from django.core.management.base import BaseCommand
from faker import Faker
from users.models import User

fake = Faker()

class Command(BaseCommand):
    help = 'Create test data using Faker'
    
    def add_arguments(self, parser):
        parser.add_argument(
            '--count',
            type=int,
            default=10,
            help='Number of users to create'
        )
    
    def handle(self, *args, **options):
        count = options['count']
        
        for i in range(count):
            User.objects.create(
                email=fake.email(),
                name=fake.name(),
                created_at=fake.date_time_this_year()
            )
        
        self.stdout.write(
            self.style.SUCCESS(f'Successfully created {count} users')
        )
```

### Uso del Comando
```bash
# Crear 10 usuarios de prueba
python manage.py create_test_data

# Crear 50 usuarios de prueba
python manage.py create_test_data --count 50
```

## 🔄 Refactoring Ejemplos

### Antes (Código Duplicado)
```python
def test_user_creation():
    user = User.objects.create(email="test@test.com", name="Test")
    assert user.email == "test@test.com"

def test_user_update():
    user = User.objects.create(email="test@test.com", name="Test")
    user.name = "Updated"
    user.save()
    assert user.name == "Updated"
```

### Después (DRY)
```python
@pytest.fixture
def test_user():
    return User.objects.create(
        email=fake.email(),
        name=fake.name()
    )

def test_user_creation(test_user):
    assert test_user.email is not None
    assert test_user.name is not None

def test_user_update(test_user):
    test_user.name = "Updated"
    test_user.save()
    assert test_user.name == "Updated"
```

## 📋 Checklist TDD

- [ ] ¿Escribiste la prueba primero?
- [ ] ¿La prueba falla inicialmente?
- [ ] ¿Implementaste código mínimo para pasar?
- [ ] ¿Refactorizaste sin cambiar comportamiento?
- [ ] ¿No usaste mocks?
- [ ] ¿Usaste Faker para datos de prueba?
- [ ] ¿Incluiste pruebas de integración?
- [ ] ¿Las pruebas son legibles y mantenibles? 