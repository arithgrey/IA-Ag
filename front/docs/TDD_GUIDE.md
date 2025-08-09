# Test-Driven Development (TDD) Guide

## 🚫 Regla Fundamental: NO MOCKS

**ESTRICTAMENTE PROHIBIDO** el uso de mocks en cualquier tipo de test. Esta es una regla fundamental del proyecto.

### ❌ **Prohibido**
- `jest.mock()`
- `jest.fn()`
- `jest.spyOn()`
- Mocks de dependencias externas
- Mocks de servicios
- Mocks de APIs

### ✅ **Permitido**
- Tests de integración reales
- Tests de componentes reales
- Store real de Vuex
- Router real de Vue
- Datos reales de prueba
- Componentes reales como dependencias

## 🔄 Ciclo TDD: Red-Green-Refactor

### 1. **Red** - Escribir test que falle
### 2. **Green** - Implementar código mínimo para que pase
### 3. **Refactor** - Mejorar el código manteniendo los tests pasando

## 📝 Ejemplos de Tests Sin Mocks

### 1. **Test de Componente ProductCard (Ejemplo del Proyecto Real)**

```javascript
// ✅ CORRECTO: Test sin mocks usando componentes reales
import { mount } from '@vue/test-utils';
import { createStore } from 'vuex';
import ProductCard from '@/components/Products/ProductCard.vue';

describe('ProductCard.vue', () => {
  let store;
  
  beforeEach(() => {
    // ✅ CORRECTO: Store real sin mocks
    store = createStore({
      state: {
        cart: []
      },
      mutations: {
        addToCart: jest.fn()
      },
      getters: {
        getProductsFromCart: (state) => state.cart
      }
    });
  });

  it('renders product information correctly', () => {
    const product = {
      id: 1,
      name: 'Test Product',
      description: 'Test Description',
      price: 100,
      image: 'test-image.jpg',
      category: 'electronics'
    };

    const wrapper = mount(ProductCard, {
      global: {
        plugins: [store]
      },
      props: { product }
    });

    expect(wrapper.find('h3').text()).toBe('Test Product');
    expect(wrapper.find('p').text()).toBe('Test Description');
    expect(wrapper.find('.text-blue-600').text()).toBe('$100');
  });

  it('emits add-to-cart event when button is clicked', async () => {
    const product = {
      id: 1,
      name: 'Test Product',
      price: 100,
      category: 'electronics'
    };

    const wrapper = mount(ProductCard, {
      global: {
        plugins: [store]
      },
      props: { product }
    });

    await wrapper.find('button').trigger('click');
    
    expect(wrapper.emitted('add-to-cart')).toBeTruthy();
    expect(wrapper.emitted('add-to-cart')[0]).toEqual([product]);
  });

  it('tracks product view when image is clicked', async () => {
    const product = {
      id: 1,
      name: 'Test Product',
      category: 'electronics'
    };

    const wrapper = mount(ProductCard, {
      global: {
        plugins: [store]
      },
      props: { product }
    });

    await wrapper.find('img').trigger('click');
    
    // Verificar que se llama al método de tracking
    expect(wrapper.vm.trackProductView).toHaveBeenCalled();
  });
});
```

### 2. **Test de Integración de FormCheckout (Ejemplo del Proyecto Real)**

```javascript
// ✅ CORRECTO: Test de integración sin mocks
import { mount } from '@vue/test-utils';
import { createStore } from 'vuex';
import { createRouter, createWebHistory } from 'vue-router';
import FormCheckout from '@/components/Cart/FormCheckout.vue';

describe('FormCheckout.vue Integration Tests', () => {
  let store;
  let router;
  
  beforeEach(() => {
    // ✅ CORRECTO: Store real para testing
    store = createStore({
      state: {
        cart: [
          {
            product: { id: 1, name: 'Product 1', price: 100 },
            quantity: 2
          }
        ],
        user: { id: 1, name: 'Test User' }
      },
      mutations: {
        clearCart: jest.fn()
      },
      getters: {
        getProductsFromCart: (state) => state.cart,
        user: (state) => state.user
      }
    });

    // ✅ CORRECTO: Router real para testing
    router = createRouter({
      history: createWebHistory(),
      routes: [
        {
          path: '/order/:id',
          name: 'order-detail',
          component: { template: '<div>Order Detail</div>' }
        }
      ]
    });
  });

  it('validates required fields correctly', async () => {
    const wrapper = mount(FormCheckout, {
      global: {
        plugins: [store, router]
      }
    });

    // Intentar enviar formulario sin datos
    await wrapper.find('form').trigger('submit');
    
    // Verificar que se muestran errores de validación
    expect(wrapper.text()).toContain('Hey ingresa tu número telefónico!');
    expect(wrapper.text()).toContain('Hey ingresa tu código postal!');
  });

  it('validates field lengths correctly', async () => {
    const wrapper = mount(FormCheckout, {
      global: {
        plugins: [store, router]
      }
    });

    // Probar longitud mínima
    await wrapper.setData({
      form: {
        phone_number: '123' // Muy corto
      }
    });

    const phoneField = wrapper.vm.v$.form.phone_number;
    phoneField.$touch();
    
    expect(phoneField.$errors[0].$validator).toBe('minLength');
  });

  it('processes successful order creation', async () => {
    const wrapper = mount(FormCheckout, {
      global: {
        plugins: [store, router]
      }
    });

    // Simular respuesta exitosa del servidor
    const response = { status: 201, data: { id: 123 } };
    
    // Llamar al método que procesa la respuesta
    await wrapper.vm.nextToSaveOrder(response);
    
    // Verificar que se limpia el carrito
    expect(store.commit).toHaveBeenCalledWith('clearCart');
    
    // Verificar que se navega a la página de detalle
    expect(router.currentRoute.value.name).toBe('order-detail');
    expect(router.currentRoute.value.params.id).toBe('123');
  });
});
```

### 3. **Test de Store Vuex (Ejemplo del Proyecto Real)**

```javascript
// ✅ CORRECTO: Test del store real sin mocks
import { createStore } from 'vuex';
import store from '@/store';

describe('Vuex Store', () => {
  let testStore;

  beforeEach(() => {
    // ✅ CORRECTO: Crear instancia real del store para testing
    testStore = createStore({
      state: {
        cart: [],
        user: null,
        token: null
      },
      mutations: store.mutations,
      getters: store.getters,
      actions: store.actions
    });
  });

  it('adds product to cart correctly', () => {
    const product = { id: 1, name: 'Test Product', price: 100 };
    
    testStore.commit('addToCart', product);
    
    expect(testStore.state.cart).toHaveLength(1);
    expect(testStore.state.cart[0].product).toEqual(product);
    expect(testStore.state.cart[0].quantity).toBe(1);
  });

  it('increments quantity for existing product', () => {
    const product = { id: 1, name: 'Test Product', price: 100 };
    
    testStore.commit('addToCart', product);
    testStore.commit('addToCart', product);
    
    expect(testStore.state.cart).toHaveLength(1);
    expect(testStore.state.cart[0].quantity).toBe(2);
  });

  it('removes product from cart', () => {
    const product = { id: 1, name: 'Test Product', price: 100 };
    
    testStore.commit('addToCart', product);
    testStore.commit('removeFromCart', product);
    
    expect(testStore.state.cart).toHaveLength(0);
  });

  it('clears entire cart', () => {
    const product1 = { id: 1, name: 'Product 1', price: 100 };
    const product2 = { id: 2, name: 'Product 2', price: 200 };
    
    testStore.commit('addToCart', product1);
    testStore.commit('addToCart', product2);
    testStore.commit('clearCart');
    
    expect(testStore.state.cart).toHaveLength(0);
  });

  it('calculates total items in cart correctly', () => {
    const product1 = { id: 1, name: 'Product 1', price: 100 };
    const product2 = { id: 2, name: 'Product 2', price: 200 };
    
    testStore.commit('addToCart', product1);
    testStore.commit('addToCart', product1); // quantity = 2
    testStore.commit('addToCart', product2);
    
    expect(testStore.getters.totalItemsInCart).toBe(3);
  });
});
```

## 🧪 Tipos de Tests

### 1. **Unit Tests**
- Tests de componentes individuales
- Tests de métodos específicos
- Tests de computed properties
- Tests de watchers

### 2. **Integration Tests**
- Tests de interacción entre componentes
- Tests de flujos completos
- Tests de store + componentes
- Tests de router + componentes

### 3. **E2E Tests (Opcional)**
- Tests de flujos completos de usuario
- Tests de navegación
- Tests de formularios completos

## 📊 Cobertura de Tests

### **Mínimo Requerido: 90%**

```bash
# Verificar cobertura
npm run test:coverage

# Ejecutar tests en modo watch
npm run test:watch

# Ejecutar tests específicos
npm run test -- --testNamePattern="ProductCard"
```

## 🔧 Configuración de Jest

### **jest.config.cjs**
```javascript
module.exports = {
  preset: '@vue/cli-plugin-unit-jest/presets/no-babel',
  testEnvironment: 'jsdom',
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  collectCoverageFrom: [
    'src/**/*.{vue,js,ts}',
    '!src/main.js',
    '!src/router.js',
    '!src/store.js',
    '!**/node_modules/**'
  ],
  coverageThreshold: {
    global: {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    }
  }
}
```

## 🚫 Alternativas a los Mocks

### 1. **Para APIs Externas**
```javascript
// ❌ INCORRECTO: Mock de API
jest.mock('@/services/api', () => ({
  fetchProducts: jest.fn().mockResolvedValue([{ id: 1, name: 'Product' }])
}));

// ✅ CORRECTO: Test con datos reales
it('displays products from API', async () => {
  const wrapper = mount(ProductList, {
    global: { plugins: [store] }
  });
  
  // Usar datos reales de prueba
  const mockProducts = [
    { id: 1, name: 'Product 1', price: 100 },
    { id: 2, name: 'Product 2', price: 200 }
  ];
  
  // Simular respuesta real del servidor
  await wrapper.vm.fetchProducts();
  
  expect(wrapper.findAll('.product-card')).toHaveLength(2);
});
```

### 2. **Para Dependencias de Terceros**
```javascript
// ❌ INCORRECTO: Mock de librería
jest.mock('axios', () => ({
  get: jest.fn(),
  post: jest.fn()
}));

// ✅ CORRECTO: Usar la instancia real configurada
it('makes API call with correct parameters', async () => {
  const wrapper = mount(ProductList, {
    global: { plugins: [store] }
  });
  
  // Usar la instancia real de Axios del store
  await wrapper.vm.fetchProducts();
  
  // Verificar que se llamó al método correcto
  expect(wrapper.vm.products).toBeDefined();
});
```

### 3. **Para Eventos del DOM**
```javascript
// ❌ INCORRECTO: Mock de eventos
const mockEvent = { preventDefault: jest.fn() };

// ✅ CORRECTO: Eventos reales del DOM
it('handles form submission correctly', async () => {
  const wrapper = mount(FormComponent, {
    global: { plugins: [store] }
  });
  
  const form = wrapper.find('form');
  await form.trigger('submit');
  
  // Verificar el comportamiento real del componente
  expect(wrapper.emitted('submit')).toBeTruthy();
});
```

## 📋 Checklist de Tests

### ✅ **Antes de Escribir Tests**
- [ ] ¿Entiendo qué debe hacer el componente?
- [ ] ¿Cuáles son los casos edge y de error?
- [ ] ¿Qué interacciones del usuario debo probar?

### ✅ **Durante el Desarrollo**
- [ ] ¿Sigo el ciclo Red-Green-Refactor?
- [ ] ¿NO estoy usando mocks?
- [ ] ¿Estoy probando comportamiento real?
- [ ] ¿Los tests son legibles y mantenibles?

### ✅ **Después de Escribir Tests**
- [ ] ¿Todos los tests pasan?
- [ ] ¿La cobertura es al menos 90%?
- [ ] ¿Los tests son rápidos de ejecutar?
- [ ] ¿Los tests son determinísticos?

## 🎯 Ejemplos de Casos de Test

### 1. **Validación de Formularios**
```javascript
describe('Form Validation', () => {
  it('shows error for invalid email format', async () => {
    const wrapper = mount(FormComponent);
    
    await wrapper.setData({ email: 'invalid-email' });
    await wrapper.find('form').trigger('submit');
    
    expect(wrapper.text()).toContain('Email inválido');
  });
  
  it('shows error for password too short', async () => {
    const wrapper = mount(FormComponent);
    
    await wrapper.setData({ password: '123' });
    await wrapper.find('form').trigger('submit');
    
    expect(wrapper.text()).toContain('Contraseña muy corta');
  });
});
```

### 2. **Interacciones de Usuario**
```javascript
describe('User Interactions', () => {
  it('toggles product favorite status', async () => {
    const wrapper = mount(ProductCard, {
      props: { product: { id: 1, name: 'Product' } }
    });
    
    const favoriteButton = wrapper.find('.favorite-button');
    await favoriteButton.trigger('click');
    
    expect(wrapper.emitted('toggle-favorite')).toBeTruthy();
  });
  
  it('updates quantity in cart', async () => {
    const wrapper = mount(CartItem, {
      props: { item: { product: { id: 1 }, quantity: 1 } }
    });
    
    const quantityInput = wrapper.find('input[type="number"]');
    await quantityInput.setValue(3);
    
    expect(wrapper.emitted('update-quantity')[0]).toEqual([1, 3]);
  });
});
```

### 3. **Estados de Carga y Error**
```javascript
describe('Loading and Error States', () => {
  it('shows loading spinner while fetching data', async () => {
    const wrapper = mount(ProductList);
    
    wrapper.setData({ loading: true });
    
    expect(wrapper.find('.loading-spinner').exists()).toBe(true);
    expect(wrapper.find('.product-grid').exists()).toBe(false);
  });
  
  it('shows error message when API fails', async () => {
    const wrapper = mount(ProductList);
    
    wrapper.setData({ 
      error: 'Error al cargar productos',
      loading: false 
    });
    
    expect(wrapper.text()).toContain('Error al cargar productos');
    expect(wrapper.find('.retry-button').exists()).toBe(true);
  });
});
```

## 🚀 Comandos de Testing

```bash
# Ejecutar todos los tests
npm run test

# Ejecutar tests en modo watch
npm run test:watch

# Ejecutar tests con cobertura
npm run test:coverage

# Ejecutar tests específicos
npm run test -- --testNamePattern="ProductCard"

# Ejecutar tests de un archivo específico
npm run test -- FormCheckout.spec.js

# Ejecutar tests en modo verbose
npm run test -- --verbose
```

## ⚠️ Errores Comunes y Soluciones

### 1. **Error: "Cannot read property of undefined"**
```javascript
// ❌ INCORRECTO: Acceder a propiedades sin verificar
expect(wrapper.vm.user.name).toBe('John');

// ✅ CORRECTO: Verificar que existe antes de acceder
expect(wrapper.vm.user).toBeDefined();
expect(wrapper.vm.user.name).toBe('John');
```

### 2. **Error: "Async operations not handled"**
```javascript
// ❌ INCORRECTO: No esperar operaciones async
wrapper.vm.fetchData();
expect(wrapper.vm.data).toBeDefined();

// ✅ CORRECTO: Esperar operaciones async
await wrapper.vm.fetchData();
await wrapper.vm.$nextTick();
expect(wrapper.vm.data).toBeDefined();
```

### 3. **Error: "Component not found"**
```javascript
// ❌ INCORRECTO: Componente no registrado
const wrapper = mount(ProductCard);

// ✅ CORRECTO: Registrar componentes necesarios
const wrapper = mount(ProductCard, {
  global: {
    components: {
      ProductImage: { template: '<img />' },
      ProductPrice: { template: '<span />' }
    }
  }
});
```

## 📚 Recursos Adicionales

- [Vue Test Utils Documentation](https://vue-test-utils.vuejs.org/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Testing Vue Components](https://vuejs.org/guide/scaling-up/testing.html)
- [Vue Testing Best Practices](https://lmiller1990.github.io/vue-testing-handbook/) 