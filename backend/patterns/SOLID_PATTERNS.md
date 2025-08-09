# Patrones SOLID en Django

## 🎯 Aplicación Práctica de SOLID

### 1. Single Responsibility Principle (SRP)

#### ❌ Mal - Múltiples responsabilidades
```python
class UserService:
    def create_user(self, data):
        # Validación
        if not data.get('email'):
            raise ValueError("Email required")
        
        # Creación
        user = User.objects.create(**data)
        
        # Envío de email
        send_welcome_email(user.email)
        
        # Logging
        logger.info(f"User created: {user.id}")
        
        return user
```

#### ✅ Bien - Responsabilidad única
```python
class UserValidator:
    def validate(self, data):
        if not data.get('email'):
            raise ValueError("Email required")
        return data

class UserCreator:
    def create(self, data):
        return User.objects.create(**data)

class EmailService:
    def send_welcome(self, email):
        send_welcome_email(email)

class UserService:
    def __init__(self):
        self.validator = UserValidator()
        self.creator = UserCreator()
        self.email_service = EmailService()
    
    def create_user(self, data):
        validated_data = self.validator.validate(data)
        user = self.creator.create(validated_data)
        self.email_service.send_welcome(user.email)
        return user
```

### 2. Open/Closed Principle (OCP)

#### ✅ Extensible sin modificar código existente
```python
# Base class
class PaymentProcessor:
    def process_payment(self, amount):
        raise NotImplementedError

# Extensiones
class CreditCardProcessor(PaymentProcessor):
    def process_payment(self, amount):
        return f"Processing {amount} via credit card"

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount):
        return f"Processing {amount} via PayPal"

# Uso
def process_payment(processor: PaymentProcessor, amount):
    return processor.process_payment(amount)
```

### 3. Liskov Substitution Principle (LSP)

#### ✅ Subclases sustituibles
```python
class BaseSerializer:
    def serialize(self, obj):
        raise NotImplementedError

class UserSerializer(BaseSerializer):
    def serialize(self, user):
        return {
            'id': user.id,
            'email': user.email
        }

class ProductSerializer(BaseSerializer):
    def serialize(self, product):
        return {
            'id': product.id,
            'name': product.name,
            'price': product.price
        }

# Uso - cualquier serializer funciona
def serialize_object(serializer: BaseSerializer, obj):
    return serializer.serialize(obj)
```

### 4. Interface Segregation Principle (ISP)

#### ❌ Mal - Interface muy grande
```python
class UserOperations:
    def create(self, data): pass
    def update(self, id, data): pass
    def delete(self, id): pass
    def read(self, id): pass
    def list_all(self): pass
    def search(self, query): pass
    def export(self, format): pass
    def import_data(self, file): pass
```

#### ✅ Bien - Interfaces específicas
```python
class UserReader:
    def read(self, id): pass
    def list_all(self): pass
    def search(self, query): pass

class UserWriter:
    def create(self, data): pass
    def update(self, id, data): pass
    def delete(self, id): pass

class UserExporter:
    def export(self, format): pass
    def import_data(self, file): pass
```

### 5. Dependency Inversion Principle (DIP)

#### ✅ Depender de abstracciones
```python
# Abstracción
class UserRepository:
    def save(self, user): pass
    def find_by_id(self, id): pass

# Implementación concreta
class DjangoUserRepository(UserRepository):
    def save(self, user):
        return user.save()
    
    def find_by_id(self, id):
        return User.objects.get(id=id)

# Servicio que depende de abstracción
class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository
    
    def create_user(self, data):
        user = User(**data)
        return self.repository.save(user)
```

## 🚀 Implementación en Django

### Serializers como Abstracción
```python
# ✅ Usar Serializers en lugar de Repositories
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

### Testing con SOLID
```python
class TestUserService:
    def test_create_user_with_valid_data(self):
        # Arrange
        service = UserService()
        data = {'email': 'test@test.com', 'name': 'Test User'}
        
        # Act
        user = service.create_user(data)
        
        # Assert
        assert user.email == data['email']
        assert user.name == data['name']
``` 