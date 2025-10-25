# Plan de Proyecto - App Tienda Tech

## Arq. Programación Móvil - 2025

---

## 1. Resumen del Proyecto

### Objetivo
Desarrollar una aplicación móvil de e-commerce para productos tecnológicos utilizando React Native + Expo como frontend y Strapi como backend headless CMS.

### Stack Tecnológico
- **Frontend**: React Native + Expo + TypeScript
- **Backend**: Strapi v4 (Headless CMS)
- **Base de Datos**: PostgreSQL
- **API**: REST (Strapi)
- **Estado**: Context API (simple, sin Redux)

### Alcance MVP
- Browse de productos por categoría/marca con filtros y paginación
- Detalle de producto con imágenes, specs, precio, descuento
- Carrito de compras (agregar, quitar, actualizar cantidades)
- Checkout simulado
- Autenticación email/contraseña
- Favoritos de usuario
- Backoffice en Strapi para gestión

---

## 2. Requisitos Previos

### Software Necesario
- **Node.js**: v18 o superior
- **npm** o **yarn**
- **PostgreSQL**: v14 o superior
- **pgAdmin 4**: para gestión de base de datos
- **Git**: para control de versiones
- **Expo CLI**: `npm install -g expo-cli`
- **Editor**: VS Code (recomendado)

### Verificar Instalaciones
```bash
node --version  # v18+
npm --version
psql --version  # PostgreSQL 14+
```

---

## 3. Configuración de PostgreSQL

### Paso 1: Crear Base de Datos con pgAdmin

1. Abrir **pgAdmin 4**
2. Conectarse al servidor local (localhost)
3. Click derecho en "Databases" → Create → Database
4. Configurar:
   - **Database name**: `tienda_tech_db`
   - **Owner**: postgres (o tu usuario)
   - **Encoding**: UTF8
5. Click en "Save"

### Paso 2: Crear Usuario (Opcional)
```sql
CREATE USER strapi_user WITH PASSWORD 'strapi_password';
GRANT ALL PRIVILEGES ON DATABASE tienda_tech_db TO strapi_user;
```

### Credenciales para Strapi
Anota estos datos para la configuración de Strapi:
- **Host**: localhost
- **Port**: 5432
- **Database**: tienda_tech_db
- **Username**: strapi_user
- **Password**: strapi_password

---

## 4. Estructura del Proyecto

```
tp5/
├── backend/              # Strapi (Headless CMS)
│   ├── config/
│   ├── src/
│   │   ├── api/         # Content types y controllers
│   │   ├── extensions/
│   │   └── middlewares/
│   ├── public/
│   ├── .env
│   └── package.json
│
├── mobile/               # React Native + Expo
│   ├── src/
│   │   ├── components/  # Componentes reutilizables
│   │   ├── screens/     # Pantallas de la app
│   │   ├── navigation/  # Configuración de navegación
│   │   ├── context/     # Estado global (Context API)
│   │   ├── services/    # Llamadas a API
│   │   ├── types/       # TypeScript types
│   │   └── utils/       # Helpers y constantes
│   ├── assets/
│   ├── App.tsx
│   ├── app.json
│   └── package.json
│
├── TP5.MD               # Requerimientos originales
└── PLAN_PROYECTO.md     # Este archivo
```

---

## 5. Instalación Backend (Strapi)

### Paso 1: Crear Proyecto Strapi

Desde el directorio `tp5/`:

```bash
npx create-strapi-app@latest backend --quickstart --no-run
```

**Notas**:
- `--quickstart`: configuración rápida
- `--no-run`: no ejecutar automáticamente

### Paso 2: Configurar PostgreSQL

Navegar al directorio backend:
```bash
cd backend
```

Instalar el driver de PostgreSQL:
```bash
npm install pg
```

Editar el archivo `config/database.js`:

```javascript
module.exports = ({ env }) => ({
  connection: {
    client: 'postgres',
    connection: {
      host: env('DATABASE_HOST', 'localhost'),
      port: env.int('DATABASE_PORT', 5432),
      database: env('DATABASE_NAME', 'tienda_tech_db'),
      user: env('DATABASE_USERNAME', 'postgres'),
      password: env('DATABASE_PASSWORD', ''),
      ssl: env.bool('DATABASE_SSL', false),
    },
  },
});
```

Crear archivo `.env` en la raíz de `backend/`:

```env
HOST=0.0.0.0
PORT=1337

# Database
DATABASE_CLIENT=postgres
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=tienda_tech_db
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=tu_password_aqui
DATABASE_SSL=false

# Secrets
APP_KEYS=generado_automaticamente
API_TOKEN_SALT=generado_automaticamente
ADMIN_JWT_SECRET=generado_automaticamente
TRANSFER_TOKEN_SALT=generado_automaticamente
JWT_SECRET=generado_automaticamente
```

**Importante**: Strapi genera automáticamente los secretos. Si no existen, los creará en el primer inicio.

### Paso 3: Iniciar Strapi

```bash
npm run develop
```

- Se abrirá el navegador en `http://localhost:1337/admin`
- Crear cuenta de administrador (email + password)
- Guardar credenciales de admin

---

## 6. Modelo de Dominio en Strapi

### Content Types a Crear

Desde el panel de Strapi: **Content-Type Builder**

#### 6.1 Categoría (categoria)

| Campo | Tipo | Configuración |
|-------|------|---------------|
| nombre | Text | Required, Unique |
| descripcion | Text (Long text) | Optional |
| imagen | Media (Single) | Optional |

#### 6.2 Marca (marca)

| Campo | Tipo | Configuración |
|-------|------|---------------|
| nombre | Text | Required, Unique |
| logo | Media (Single) | Optional |

#### 6.3 Producto (producto)

| Campo | Tipo | Configuración |
|-------|------|---------------|
| nombre | Text | Required |
| descripcion | Text (Long text) | Required |
| precio | Decimal | Required, Min: 0 |
| descuento | Number | Min: 0, Max: 100, Default: 0 |
| stock | Number | Required, Min: 0, Default: 0 |
| activo | Boolean | Default: true |
| imagenes | Media (Multiple) | Optional |
| especificaciones | JSON | Optional |
| categoria | Relation | Many-to-One con Categoria |
| marca | Relation | Many-to-One con Marca |

**Campo especificaciones** (ejemplo JSON):
```json
{
  "procesador": "Intel i7",
  "ram": "16GB",
  "almacenamiento": "512GB SSD",
  "pantalla": "15.6 pulgadas"
}
```

#### 6.4 Orden (orden)

| Campo | Tipo | Configuración |
|-------|------|---------------|
| numero_orden | UID | Auto-generated |
| total | Decimal | Required, Min: 0 |
| estado | Enumeration | pendiente, procesando, completada, cancelada |
| items | JSON | Required |
| usuario | Relation | Many-to-One con User |

**Campo items** (ejemplo JSON):
```json
[
  {
    "producto_id": 1,
    "nombre": "Laptop HP",
    "cantidad": 2,
    "precio": 500,
    "subtotal": 1000
  }
]
```

#### 6.5 Favorito (favorito)

| Campo | Tipo | Configuración |
|-------|------|---------------|
| usuario | Relation | Many-to-One con User |
| producto | Relation | Many-to-One con Producto |

**Nota**: Configurar relación única para evitar duplicados (usuario + producto).

---

## 7. Configuración de Permisos en Strapi

### Settings → Users & Permissions Plugin → Roles

#### Public (usuarios no autenticados)
- **Categoria**: find, findOne
- **Marca**: find, findOne
- **Producto**: find, findOne

#### Authenticated (usuarios autenticados)
- **Categoria**: find, findOne
- **Marca**: find, findOne
- **Producto**: find, findOne
- **Orden**: create, find (solo sus órdenes)
- **Favorito**: create, delete, find (solo sus favoritos)

### Crear Custom Policy (Opcional)

Para que usuarios solo vean sus propios favoritos/órdenes, Strapi tiene políticas por defecto. Para reforzar:

Archivo: `backend/src/policies/isOwner.js`
```javascript
module.exports = async (policyContext, config, { strapi }) => {
  const user = policyContext.state.user;

  if (!user) {
    return false;
  }

  const { id } = policyContext.params;
  const entry = await strapi.entityService.findOne(
    config.contentType,
    id,
    { populate: ['usuario'] }
  );

  return entry && entry.usuario.id === user.id;
};
```

---

## 8. Instalación Frontend (Mobile)

### Paso 1: Crear Proyecto Expo

Desde el directorio `tp5/`:

```bash
npx create-expo-app mobile --template expo-template-blank-typescript
```

### Paso 2: Navegar e Instalar Dependencias

```bash
cd mobile
npm install @react-navigation/native @react-navigation/stack @react-navigation/bottom-tabs
npm install react-native-screens react-native-safe-area-context
npm install expo-secure-store
npx expo install expo-image
```

**Dependencias**:
- `@react-navigation/*`: navegación simple
- `expo-secure-store`: almacenamiento seguro de tokens
- `expo-image`: optimización de imágenes

### Paso 3: Estructura de Carpetas

Crear estructura en `mobile/src/`:

```bash
mkdir -p src/{components,screens,navigation,context,services,types,utils}
```

---

## 9. Configuración de TypeScript

### mobile/src/types/index.ts

```typescript
// Tipos principales
export interface Categoria {
  id: number;
  attributes: {
    nombre: string;
    descripcion?: string;
    imagen?: {
      data?: {
        attributes: {
          url: string;
        };
      };
    };
  };
}

export interface Marca {
  id: number;
  attributes: {
    nombre: string;
    logo?: {
      data?: {
        attributes: {
          url: string;
        };
      };
    };
  };
}

export interface Producto {
  id: number;
  attributes: {
    nombre: string;
    descripcion: string;
    precio: number;
    descuento: number;
    stock: number;
    activo: boolean;
    especificaciones?: any;
    imagenes?: {
      data?: Array<{
        attributes: {
          url: string;
        };
      }>;
    };
    categoria?: {
      data?: Categoria;
    };
    marca?: {
      data?: Marca;
    };
  };
}

export interface CarritoItem {
  producto: Producto;
  cantidad: number;
}

export interface Usuario {
  id: number;
  username: string;
  email: string;
  jwt: string;
}

export interface Orden {
  id: number;
  attributes: {
    numero_orden: string;
    total: number;
    estado: 'pendiente' | 'procesando' | 'completada' | 'cancelada';
    items: Array<{
      producto_id: number;
      nombre: string;
      cantidad: number;
      precio: number;
      subtotal: number;
    }>;
    createdAt: string;
  };
}
```

---

## 10. Configuración de API Service

### mobile/src/utils/constants.ts

```typescript
export const API_URL = 'http://localhost:1337/api';
export const BASE_URL = 'http://localhost:1337';
```

**Nota**: Para testing en dispositivo físico, cambiar `localhost` por la IP local (ej: `192.168.1.100`).

### mobile/src/services/api.ts

```typescript
import { API_URL } from '../utils/constants';

interface FetchOptions extends RequestInit {
  token?: string;
}

export const fetchApi = async (endpoint: string, options: FetchOptions = {}) => {
  const { token, ...fetchOptions } = options;

  const headers: HeadersInit = {
    'Content-Type': 'application/json',
    ...(fetchOptions.headers || {}),
  };

  if (token) {
    headers['Authorization'] = `Bearer ${token}`;
  }

  const response = await fetch(`${API_URL}${endpoint}`, {
    ...fetchOptions,
    headers,
  });

  if (!response.ok) {
    const error = await response.json().catch(() => ({ message: 'Error desconocido' }));
    throw new Error(error.message || 'Error en la petición');
  }

  return response.json();
};

// Productos
export const getProductos = async (filters?: string) => {
  const query = filters ? `?${filters}` : '?populate=*';
  return fetchApi(`/productos${query}`);
};

export const getProducto = async (id: number) => {
  return fetchApi(`/productos/${id}?populate=*`);
};

// Categorías
export const getCategorias = async () => {
  return fetchApi('/categorias?populate=*');
};

// Marcas
export const getMarcas = async () => {
  return fetchApi('/marcas?populate=*');
};

// Auth
export const login = async (email: string, password: string) => {
  return fetchApi('/auth/local', {
    method: 'POST',
    body: JSON.stringify({ identifier: email, password }),
  });
};

export const registro = async (username: string, email: string, password: string) => {
  return fetchApi('/auth/local/register', {
    method: 'POST',
    body: JSON.stringify({ username, email, password }),
  });
};

// Favoritos
export const getFavoritos = async (token: string, userId: number) => {
  return fetchApi(`/favoritos?filters[usuario][id][$eq]=${userId}&populate=producto.imagenes,producto.categoria,producto.marca`, { token });
};

export const addFavorito = async (token: string, userId: number, productoId: number) => {
  return fetchApi('/favoritos', {
    method: 'POST',
    token,
    body: JSON.stringify({
      data: {
        usuario: userId,
        producto: productoId,
      },
    }),
  });
};

export const removeFavorito = async (token: string, favoritoId: number) => {
  return fetchApi(`/favoritos/${favoritoId}`, {
    method: 'DELETE',
    token,
  });
};

// Órdenes
export const createOrden = async (token: string, userId: number, items: any[], total: number) => {
  return fetchApi('/ordens', {
    method: 'POST',
    token,
    body: JSON.stringify({
      data: {
        usuario: userId,
        items,
        total,
        estado: 'pendiente',
      },
    }),
  });
};

export const getOrdenes = async (token: string, userId: number) => {
  return fetchApi(`/ordens?filters[usuario][id][$eq]=${userId}&sort=createdAt:desc`, { token });
};
```

---

## 11. Context API para Estado Global

### mobile/src/context/AuthContext.tsx

```typescript
import React, { createContext, useState, useContext, useEffect } from 'react';
import * as SecureStore from 'expo-secure-store';
import { Usuario } from '../types';

interface AuthContextType {
  user: Usuario | null;
  isLoading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  register: (username: string, email: string, password: string) => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<Usuario | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    loadUser();
  }, []);

  const loadUser = async () => {
    try {
      const userData = await SecureStore.getItemAsync('user');
      if (userData) {
        setUser(JSON.parse(userData));
      }
    } catch (error) {
      console.error('Error cargando usuario:', error);
    } finally {
      setIsLoading(false);
    }
  };

  const login = async (email: string, password: string) => {
    const { login: loginService } = await import('../services/api');
    const data = await loginService(email, password);

    const usuario: Usuario = {
      id: data.user.id,
      username: data.user.username,
      email: data.user.email,
      jwt: data.jwt,
    };

    await SecureStore.setItemAsync('user', JSON.stringify(usuario));
    setUser(usuario);
  };

  const logout = async () => {
    await SecureStore.deleteItemAsync('user');
    setUser(null);
  };

  const register = async (username: string, email: string, password: string) => {
    const { registro } = await import('../services/api');
    const data = await registro(username, email, password);

    const usuario: Usuario = {
      id: data.user.id,
      username: data.user.username,
      email: data.user.email,
      jwt: data.jwt,
    };

    await SecureStore.setItemAsync('user', JSON.stringify(usuario));
    setUser(usuario);
  };

  return (
    <AuthContext.Provider value={{ user, isLoading, login, logout, register }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth debe usarse dentro de AuthProvider');
  }
  return context;
};
```

### mobile/src/context/CarritoContext.tsx

```typescript
import React, { createContext, useState, useContext } from 'react';
import { CarritoItem, Producto } from '../types';

interface CarritoContextType {
  items: CarritoItem[];
  agregarProducto: (producto: Producto, cantidad: number) => void;
  eliminarProducto: (productoId: number) => void;
  actualizarCantidad: (productoId: number, cantidad: number) => void;
  limpiarCarrito: () => void;
  calcularTotal: () => number;
  calcularTotalItems: () => number;
}

const CarritoContext = createContext<CarritoContextType | undefined>(undefined);

export const CarritoProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [items, setItems] = useState<CarritoItem[]>([]);

  const agregarProducto = (producto: Producto, cantidad: number) => {
    setItems(prev => {
      const existe = prev.find(item => item.producto.id === producto.id);

      if (existe) {
        return prev.map(item =>
          item.producto.id === producto.id
            ? { ...item, cantidad: item.cantidad + cantidad }
            : item
        );
      }

      return [...prev, { producto, cantidad }];
    });
  };

  const eliminarProducto = (productoId: number) => {
    setItems(prev => prev.filter(item => item.producto.id !== productoId));
  };

  const actualizarCantidad = (productoId: number, cantidad: number) => {
    if (cantidad <= 0) {
      eliminarProducto(productoId);
      return;
    }

    setItems(prev =>
      prev.map(item =>
        item.producto.id === productoId ? { ...item, cantidad } : item
      )
    );
  };

  const limpiarCarrito = () => {
    setItems([]);
  };

  const calcularTotal = () => {
    return items.reduce((total, item) => {
      const precio = item.producto.attributes.precio;
      const descuento = item.producto.attributes.descuento || 0;
      const precioFinal = precio - (precio * descuento / 100);
      return total + (precioFinal * item.cantidad);
    }, 0);
  };

  const calcularTotalItems = () => {
    return items.reduce((total, item) => total + item.cantidad, 0);
  };

  return (
    <CarritoContext.Provider
      value={{
        items,
        agregarProducto,
        eliminarProducto,
        actualizarCantidad,
        limpiarCarrito,
        calcularTotal,
        calcularTotalItems,
      }}
    >
      {children}
    </CarritoContext.Provider>
  );
};

export const useCarrito = () => {
  const context = useContext(CarritoContext);
  if (!context) {
    throw new Error('useCarrito debe usarse dentro de CarritoProvider');
  }
  return context;
};
```

---

## 12. Navegación

### mobile/src/navigation/AppNavigator.tsx

```typescript
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { useAuth } from '../context/AuthContext';

// Screens (a crear)
import HomeScreen from '../screens/HomeScreen';
import ProductoDetalleScreen from '../screens/ProductoDetalleScreen';
import CarritoScreen from '../screens/CarritoScreen';
import FavoritosScreen from '../screens/FavoritosScreen';
import PerfilScreen from '../screens/PerfilScreen';
import LoginScreen from '../screens/LoginScreen';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

const MainTabs = () => (
  <Tab.Navigator>
    <Tab.Screen name="Inicio" component={HomeScreen} />
    <Tab.Screen name="Favoritos" component={FavoritosScreen} />
    <Tab.Screen name="Carrito" component={CarritoScreen} />
    <Tab.Screen name="Perfil" component={PerfilScreen} />
  </Tab.Navigator>
);

const AppNavigator = () => {
  const { user, isLoading } = useAuth();

  if (isLoading) {
    return null; // O un Loading screen
  }

  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{ headerShown: false }}>
        {user ? (
          <>
            <Stack.Screen name="Main" component={MainTabs} />
            <Stack.Screen
              name="ProductoDetalle"
              component={ProductoDetalleScreen}
              options={{ headerShown: true, title: 'Detalle del Producto' }}
            />
          </>
        ) : (
          <Stack.Screen name="Login" component={LoginScreen} />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default AppNavigator;
```

---

## 13. Pantallas Principales (Estructura Simple)

### mobile/src/screens/HomeScreen.tsx

```typescript
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, StyleSheet, TouchableOpacity } from 'react-native';
import { Image } from 'expo-image';
import { getProductos } from '../services/api';
import { Producto } from '../types';
import { BASE_URL } from '../utils/constants';

const HomeScreen = ({ navigation }: any) => {
  const [productos, setProductos] = useState<Producto[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    cargarProductos();
  }, []);

  const cargarProductos = async () => {
    try {
      const data = await getProductos('populate=*');
      setProductos(data.data);
    } catch (error) {
      console.error('Error cargando productos:', error);
    } finally {
      setLoading(false);
    }
  };

  const renderProducto = ({ item }: { item: Producto }) => {
    const imagen = item.attributes.imagenes?.data?.[0]?.attributes.url;
    const precio = item.attributes.precio;
    const descuento = item.attributes.descuento;
    const precioFinal = descuento > 0 ? precio - (precio * descuento / 100) : precio;

    return (
      <TouchableOpacity
        style={styles.card}
        onPress={() => navigation.navigate('ProductoDetalle', { id: item.id })}
      >
        <Image
          source={{ uri: imagen ? `${BASE_URL}${imagen}` : 'https://via.placeholder.com/150' }}
          style={styles.imagen}
        />
        <Text style={styles.nombre}>{item.attributes.nombre}</Text>
        <View style={styles.precioContainer}>
          {descuento > 0 && (
            <Text style={styles.precioOriginal}>${precio.toFixed(2)}</Text>
          )}
          <Text style={styles.precio}>${precioFinal.toFixed(2)}</Text>
        </View>
        {descuento > 0 && (
          <Text style={styles.descuento}>{descuento}% OFF</Text>
        )}
      </TouchableOpacity>
    );
  };

  if (loading) {
    return (
      <View style={styles.container}>
        <Text>Cargando productos...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Productos Tech</Text>
      <FlatList
        data={productos}
        renderItem={renderProducto}
        keyExtractor={(item) => item.id.toString()}
        numColumns={2}
        contentContainerStyle={styles.lista}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  titulo: {
    fontSize: 24,
    fontWeight: 'bold',
    padding: 16,
  },
  lista: {
    padding: 8,
  },
  card: {
    flex: 1,
    backgroundColor: 'white',
    margin: 8,
    padding: 12,
    borderRadius: 8,
    elevation: 2,
  },
  imagen: {
    width: '100%',
    height: 150,
    borderRadius: 8,
  },
  nombre: {
    fontSize: 16,
    fontWeight: '600',
    marginTop: 8,
  },
  precioContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginTop: 4,
  },
  precio: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#2ecc71',
  },
  precioOriginal: {
    fontSize: 14,
    textDecorationLine: 'line-through',
    color: '#999',
    marginRight: 8,
  },
  descuento: {
    fontSize: 12,
    color: '#e74c3c',
    fontWeight: 'bold',
    marginTop: 4,
  },
});

export default HomeScreen;
```

### mobile/src/screens/ProductoDetalleScreen.tsx

```typescript
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, StyleSheet, TouchableOpacity, Alert } from 'react-native';
import { Image } from 'expo-image';
import { getProducto } from '../services/api';
import { useCarrito } from '../context/CarritoContext';
import { Producto } from '../types';
import { BASE_URL } from '../utils/constants';

const ProductoDetalleScreen = ({ route }: any) => {
  const { id } = route.params;
  const [producto, setProducto] = useState<Producto | null>(null);
  const [cantidad, setCantidad] = useState(1);
  const { agregarProducto } = useCarrito();

  useEffect(() => {
    cargarProducto();
  }, []);

  const cargarProducto = async () => {
    try {
      const data = await getProducto(id);
      setProducto(data.data);
    } catch (error) {
      console.error('Error cargando producto:', error);
    }
  };

  const handleAgregarCarrito = () => {
    if (!producto) return;

    const stock = producto.attributes.stock;

    if (cantidad > stock) {
      Alert.alert('Error', `Stock insuficiente. Disponible: ${stock}`);
      return;
    }

    agregarProducto(producto, cantidad);
    Alert.alert('Éxito', 'Producto agregado al carrito');
  };

  if (!producto) {
    return (
      <View style={styles.container}>
        <Text>Cargando...</Text>
      </View>
    );
  }

  const imagen = producto.attributes.imagenes?.data?.[0]?.attributes.url;
  const precio = producto.attributes.precio;
  const descuento = producto.attributes.descuento;
  const precioFinal = descuento > 0 ? precio - (precio * descuento / 100) : precio;

  return (
    <ScrollView style={styles.container}>
      <Image
        source={{ uri: imagen ? `${BASE_URL}${imagen}` : 'https://via.placeholder.com/400' }}
        style={styles.imagen}
      />

      <View style={styles.contenido}>
        <Text style={styles.nombre}>{producto.attributes.nombre}</Text>

        <View style={styles.precioContainer}>
          {descuento > 0 && (
            <>
              <Text style={styles.precioOriginal}>${precio.toFixed(2)}</Text>
              <Text style={styles.descuento}>{descuento}% OFF</Text>
            </>
          )}
        </View>
        <Text style={styles.precio}>${precioFinal.toFixed(2)}</Text>

        <Text style={styles.stock}>Stock: {producto.attributes.stock} unidades</Text>

        <Text style={styles.titulo}>Descripción</Text>
        <Text style={styles.descripcion}>{producto.attributes.descripcion}</Text>

        {producto.attributes.especificaciones && (
          <>
            <Text style={styles.titulo}>Especificaciones</Text>
            {Object.entries(producto.attributes.especificaciones).map(([key, value]) => (
              <Text key={key} style={styles.spec}>
                • {key}: {value as string}
              </Text>
            ))}
          </>
        )}

        <View style={styles.cantidadContainer}>
          <TouchableOpacity
            onPress={() => setCantidad(Math.max(1, cantidad - 1))}
            style={styles.botonCantidad}
          >
            <Text style={styles.botonCantidadTexto}>-</Text>
          </TouchableOpacity>

          <Text style={styles.cantidad}>{cantidad}</Text>

          <TouchableOpacity
            onPress={() => setCantidad(Math.min(producto.attributes.stock, cantidad + 1))}
            style={styles.botonCantidad}
          >
            <Text style={styles.botonCantidadTexto}>+</Text>
          </TouchableOpacity>
        </View>

        <TouchableOpacity
          style={styles.botonAgregar}
          onPress={handleAgregarCarrito}
          disabled={producto.attributes.stock === 0}
        >
          <Text style={styles.botonAgregarTexto}>
            {producto.attributes.stock === 0 ? 'Sin Stock' : 'Agregar al Carrito'}
          </Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: 'white',
  },
  imagen: {
    width: '100%',
    height: 300,
  },
  contenido: {
    padding: 16,
  },
  nombre: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  precioContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 4,
  },
  precio: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#2ecc71',
    marginBottom: 8,
  },
  precioOriginal: {
    fontSize: 18,
    textDecorationLine: 'line-through',
    color: '#999',
    marginRight: 8,
  },
  descuento: {
    fontSize: 14,
    color: '#e74c3c',
    fontWeight: 'bold',
  },
  stock: {
    fontSize: 14,
    color: '#555',
    marginBottom: 16,
  },
  titulo: {
    fontSize: 18,
    fontWeight: 'bold',
    marginTop: 16,
    marginBottom: 8,
  },
  descripcion: {
    fontSize: 16,
    lineHeight: 24,
    color: '#333',
  },
  spec: {
    fontSize: 14,
    marginVertical: 4,
    color: '#555',
  },
  cantidadContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    marginVertical: 16,
  },
  botonCantidad: {
    width: 40,
    height: 40,
    backgroundColor: '#3498db',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 8,
  },
  botonCantidadTexto: {
    fontSize: 24,
    color: 'white',
    fontWeight: 'bold',
  },
  cantidad: {
    fontSize: 20,
    fontWeight: 'bold',
    marginHorizontal: 24,
  },
  botonAgregar: {
    backgroundColor: '#2ecc71',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  botonAgregarTexto: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default ProductoDetalleScreen;
```

### mobile/src/screens/CarritoScreen.tsx

```typescript
import React from 'react';
import { View, Text, FlatList, StyleSheet, TouchableOpacity, Alert } from 'react-native';
import { Image } from 'expo-image';
import { useCarrito } from '../context/CarritoContext';
import { useAuth } from '../context/AuthContext';
import { createOrden } from '../services/api';
import { CarritoItem } from '../types';
import { BASE_URL } from '../utils/constants';

const CarritoScreen = ({ navigation }: any) => {
  const { items, eliminarProducto, actualizarCantidad, limpiarCarrito, calcularTotal } = useCarrito();
  const { user } = useAuth();

  const handleCheckout = async () => {
    if (items.length === 0) {
      Alert.alert('Error', 'El carrito está vacío');
      return;
    }

    if (!user) {
      Alert.alert('Error', 'Debes iniciar sesión');
      return;
    }

    try {
      const ordenItems = items.map(item => {
        const precio = item.producto.attributes.precio;
        const descuento = item.producto.attributes.descuento || 0;
        const precioFinal = precio - (precio * descuento / 100);

        return {
          producto_id: item.producto.id,
          nombre: item.producto.attributes.nombre,
          cantidad: item.cantidad,
          precio: precioFinal,
          subtotal: precioFinal * item.cantidad,
        };
      });

      await createOrden(user.jwt, user.id, ordenItems, calcularTotal());

      limpiarCarrito();
      Alert.alert('Éxito', 'Orden creada correctamente');
    } catch (error) {
      console.error('Error creando orden:', error);
      Alert.alert('Error', 'No se pudo crear la orden');
    }
  };

  const renderItem = ({ item }: { item: CarritoItem }) => {
    const imagen = item.producto.attributes.imagenes?.data?.[0]?.attributes.url;
    const precio = item.producto.attributes.precio;
    const descuento = item.producto.attributes.descuento || 0;
    const precioFinal = precio - (precio * descuento / 100);

    return (
      <View style={styles.item}>
        <Image
          source={{ uri: imagen ? `${BASE_URL}${imagen}` : 'https://via.placeholder.com/80' }}
          style={styles.imagen}
        />

        <View style={styles.info}>
          <Text style={styles.nombre}>{item.producto.attributes.nombre}</Text>
          <Text style={styles.precio}>${precioFinal.toFixed(2)}</Text>

          <View style={styles.cantidadContainer}>
            <TouchableOpacity
              onPress={() => actualizarCantidad(item.producto.id, item.cantidad - 1)}
              style={styles.botonCantidad}
            >
              <Text>-</Text>
            </TouchableOpacity>

            <Text style={styles.cantidad}>{item.cantidad}</Text>

            <TouchableOpacity
              onPress={() => actualizarCantidad(item.producto.id, item.cantidad + 1)}
              style={styles.botonCantidad}
            >
              <Text>+</Text>
            </TouchableOpacity>
          </View>
        </View>

        <TouchableOpacity
          onPress={() => eliminarProducto(item.producto.id)}
          style={styles.botonEliminar}
        >
          <Text style={styles.textoEliminar}>✕</Text>
        </TouchableOpacity>
      </View>
    );
  };

  if (items.length === 0) {
    return (
      <View style={styles.containerVacio}>
        <Text style={styles.textoVacio}>El carrito está vacío</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={items}
        renderItem={renderItem}
        keyExtractor={(item) => item.producto.id.toString()}
      />

      <View style={styles.footer}>
        <View style={styles.totalContainer}>
          <Text style={styles.totalTexto}>Total:</Text>
          <Text style={styles.totalValor}>${calcularTotal().toFixed(2)}</Text>
        </View>

        <TouchableOpacity style={styles.botonCheckout} onPress={handleCheckout}>
          <Text style={styles.botonCheckoutTexto}>Finalizar Compra</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  containerVacio: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  textoVacio: {
    fontSize: 18,
    color: '#999',
  },
  item: {
    flexDirection: 'row',
    backgroundColor: 'white',
    padding: 12,
    marginVertical: 4,
    marginHorizontal: 8,
    borderRadius: 8,
  },
  imagen: {
    width: 80,
    height: 80,
    borderRadius: 8,
  },
  info: {
    flex: 1,
    marginLeft: 12,
  },
  nombre: {
    fontSize: 16,
    fontWeight: '600',
  },
  precio: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#2ecc71',
    marginTop: 4,
  },
  cantidadContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginTop: 8,
  },
  botonCantidad: {
    width: 28,
    height: 28,
    backgroundColor: '#ecf0f1',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 4,
  },
  cantidad: {
    marginHorizontal: 12,
    fontSize: 16,
  },
  botonEliminar: {
    padding: 8,
  },
  textoEliminar: {
    fontSize: 20,
    color: '#e74c3c',
  },
  footer: {
    backgroundColor: 'white',
    padding: 16,
    borderTopWidth: 1,
    borderTopColor: '#ddd',
  },
  totalContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 16,
  },
  totalTexto: {
    fontSize: 20,
    fontWeight: 'bold',
  },
  totalValor: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#2ecc71',
  },
  botonCheckout: {
    backgroundColor: '#3498db',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  botonCheckoutTexto: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default CarritoScreen;
```

### mobile/src/screens/LoginScreen.tsx

```typescript
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { useAuth } from '../context/AuthContext';

const LoginScreen = () => {
  const [isLogin, setIsLogin] = useState(true);
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [username, setUsername] = useState('');
  const { login, register } = useAuth();

  const handleSubmit = async () => {
    try {
      if (isLogin) {
        await login(email, password);
      } else {
        await register(username, email, password);
      }
    } catch (error: any) {
      Alert.alert('Error', error.message);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>{isLogin ? 'Iniciar Sesión' : 'Registrarse'}</Text>

      {!isLogin && (
        <TextInput
          style={styles.input}
          placeholder="Usuario"
          value={username}
          onChangeText={setUsername}
        />
      )}

      <TextInput
        style={styles.input}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      />

      <TextInput
        style={styles.input}
        placeholder="Contraseña"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />

      <TouchableOpacity style={styles.boton} onPress={handleSubmit}>
        <Text style={styles.botonTexto}>{isLogin ? 'Entrar' : 'Crear Cuenta'}</Text>
      </TouchableOpacity>

      <TouchableOpacity onPress={() => setIsLogin(!isLogin)}>
        <Text style={styles.link}>
          {isLogin ? '¿No tienes cuenta? Regístrate' : '¿Ya tienes cuenta? Inicia sesión'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    padding: 16,
    backgroundColor: 'white',
  },
  titulo: {
    fontSize: 28,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 32,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    padding: 12,
    borderRadius: 8,
    marginBottom: 16,
    fontSize: 16,
  },
  boton: {
    backgroundColor: '#3498db',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
  },
  botonTexto: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  link: {
    textAlign: 'center',
    marginTop: 16,
    color: '#3498db',
  },
});

export default LoginScreen;
```

### mobile/src/screens/FavoritosScreen.tsx

```typescript
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, StyleSheet, TouchableOpacity } from 'react-native';
import { Image } from 'expo-image';
import { useAuth } from '../context/AuthContext';
import { getFavoritos, removeFavorito } from '../services/api';
import { BASE_URL } from '../utils/constants';

const FavoritosScreen = ({ navigation }: any) => {
  const { user } = useAuth();
  const [favoritos, setFavoritos] = useState<any[]>([]);

  useEffect(() => {
    if (user) {
      cargarFavoritos();
    }
  }, [user]);

  const cargarFavoritos = async () => {
    if (!user) return;

    try {
      const data = await getFavoritos(user.jwt, user.id);
      setFavoritos(data.data);
    } catch (error) {
      console.error('Error cargando favoritos:', error);
    }
  };

  const handleRemover = async (favoritoId: number) => {
    if (!user) return;

    try {
      await removeFavorito(user.jwt, favoritoId);
      setFavoritos(prev => prev.filter(f => f.id !== favoritoId));
    } catch (error) {
      console.error('Error removiendo favorito:', error);
    }
  };

  const renderItem = ({ item }: any) => {
    const producto = item.attributes.producto.data;
    const imagen = producto.attributes.imagenes?.data?.[0]?.attributes.url;

    return (
      <View style={styles.item}>
        <TouchableOpacity
          style={styles.itemContent}
          onPress={() => navigation.navigate('ProductoDetalle', { id: producto.id })}
        >
          <Image
            source={{ uri: imagen ? `${BASE_URL}${imagen}` : 'https://via.placeholder.com/80' }}
            style={styles.imagen}
          />
          <View style={styles.info}>
            <Text style={styles.nombre}>{producto.attributes.nombre}</Text>
            <Text style={styles.precio}>${producto.attributes.precio.toFixed(2)}</Text>
          </View>
        </TouchableOpacity>

        <TouchableOpacity onPress={() => handleRemover(item.id)}>
          <Text style={styles.eliminar}>✕</Text>
        </TouchableOpacity>
      </View>
    );
  };

  if (favoritos.length === 0) {
    return (
      <View style={styles.containerVacio}>
        <Text style={styles.textoVacio}>No tienes favoritos</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={favoritos}
        renderItem={renderItem}
        keyExtractor={(item) => item.id.toString()}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  containerVacio: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  textoVacio: {
    fontSize: 18,
    color: '#999',
  },
  item: {
    flexDirection: 'row',
    backgroundColor: 'white',
    padding: 12,
    marginVertical: 4,
    marginHorizontal: 8,
    borderRadius: 8,
  },
  itemContent: {
    flex: 1,
    flexDirection: 'row',
  },
  imagen: {
    width: 80,
    height: 80,
    borderRadius: 8,
  },
  info: {
    flex: 1,
    marginLeft: 12,
    justifyContent: 'center',
  },
  nombre: {
    fontSize: 16,
    fontWeight: '600',
  },
  precio: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#2ecc71',
    marginTop: 4,
  },
  eliminar: {
    fontSize: 20,
    color: '#e74c3c',
    padding: 8,
  },
});

export default FavoritosScreen;
```

### mobile/src/screens/PerfilScreen.tsx

```typescript
import React from 'react';
import { View, Text, StyleSheet, TouchableOpacity } from 'react-native';
import { useAuth } from '../context/AuthContext';

const PerfilScreen = () => {
  const { user, logout } = useAuth();

  return (
    <View style={styles.container}>
      <Text style={styles.titulo}>Perfil</Text>

      <View style={styles.infoContainer}>
        <Text style={styles.label}>Usuario:</Text>
        <Text style={styles.valor}>{user?.username}</Text>
      </View>

      <View style={styles.infoContainer}>
        <Text style={styles.label}>Email:</Text>
        <Text style={styles.valor}>{user?.email}</Text>
      </View>

      <TouchableOpacity style={styles.boton} onPress={logout}>
        <Text style={styles.botonTexto}>Cerrar Sesión</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
    backgroundColor: 'white',
  },
  titulo: {
    fontSize: 28,
    fontWeight: 'bold',
    marginBottom: 32,
  },
  infoContainer: {
    marginBottom: 16,
  },
  label: {
    fontSize: 14,
    color: '#999',
  },
  valor: {
    fontSize: 18,
    fontWeight: '600',
    marginTop: 4,
  },
  boton: {
    backgroundColor: '#e74c3c',
    padding: 16,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 32,
  },
  botonTexto: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default PerfilScreen;
```

---

## 14. Configurar App.tsx

### mobile/App.tsx

```typescript
import React from 'react';
import { AuthProvider } from './src/context/AuthContext';
import { CarritoProvider } from './src/context/CarritoContext';
import AppNavigator from './src/navigation/AppNavigator';

export default function App() {
  return (
    <AuthProvider>
      <CarritoProvider>
        <AppNavigator />
      </CarritoProvider>
    </AuthProvider>
  );
}
```

---

## 15. Reglas de Negocio

### En el Backend (Strapi)

#### Validación de Stock antes de Crear Orden

Crear un lifecycle hook en `backend/src/api/orden/content-types/orden/lifecycles.js`:

```javascript
module.exports = {
  async beforeCreate(event) {
    const { data } = event.params;
    const items = data.items;

    // Validar stock para cada producto
    for (const item of items) {
      const producto = await strapi.entityService.findOne(
        'api::producto.producto',
        item.producto_id
      );

      if (!producto || !producto.activo) {
        throw new Error(`Producto ${item.producto_id} no disponible`);
      }

      if (producto.stock < item.cantidad) {
        throw new Error(
          `Stock insuficiente para ${producto.nombre}. Disponible: ${producto.stock}`
        );
      }
    }

    // Opcional: Reducir stock automáticamente
    for (const item of items) {
      await strapi.entityService.update(
        'api::producto.producto',
        item.producto_id,
        {
          data: {
            stock: { $decrement: item.cantidad },
          },
        }
      );
    }
  },
};
```

#### Soft Delete de Productos

En lugar de eliminar, cambiar el campo `activo` a `false`. Configurar en el panel de Strapi o crear un controller personalizado.

#### Aplicar Descuentos Automáticos

Los descuentos se aplican en el frontend al calcular el precio final. En el backend, el campo `descuento` se almacena como porcentaje (0-100).

---

## 16. Comandos de Desarrollo

### Backend (Strapi)

```bash
cd backend

# Desarrollo
npm run develop

# Producción
npm run build
npm run start

# Ver base de datos
# Abrir pgAdmin y conectarse a tienda_tech_db
```

### Mobile (React Native + Expo)

```bash
cd mobile

# Iniciar en desarrollo
npx expo start

# Android
npx expo start --android

# iOS
npx expo start --ios

# Web
npx expo start --web

# Limpiar cache
npx expo start --clear
```

---

## 17. Checklist de Funcionalidades MVP

### Backend (Strapi)
- [ ] Base de datos PostgreSQL configurada
- [ ] Content Types creados (Categoria, Marca, Producto, Orden, Favorito)
- [ ] Permisos configurados (Public, Authenticated)
- [ ] Datos de prueba cargados (categorías, marcas, productos)
- [ ] Lifecycle hooks implementados (validación de stock)

### Mobile (App)
- [ ] Autenticación (login, registro, logout)
- [ ] Listado de productos con imágenes
- [ ] Filtros por categoría/marca
- [ ] Detalle de producto con specs
- [ ] Agregar al carrito con validación de stock
- [ ] Ver carrito y modificar cantidades
- [ ] Checkout (crear orden)
- [ ] Favoritos (agregar, eliminar, listar)
- [ ] Cálculo de descuentos
- [ ] Navegación funcional

---

## 18. Testing Manual

### Casos de Prueba

1. **Registro de usuario**
   - Registrar nuevo usuario
   - Verificar sesión iniciada automáticamente

2. **Login**
   - Login con credenciales válidas
   - Verificar persistencia de sesión

3. **Productos**
   - Visualizar listado de productos
   - Ver detalle de producto
   - Verificar imágenes y precios

4. **Carrito**
   - Agregar producto al carrito
   - Modificar cantidades
   - Eliminar producto
   - Validar stock insuficiente

5. **Checkout**
   - Crear orden con productos en carrito
   - Verificar orden en backoffice de Strapi

6. **Favoritos**
   - Agregar producto a favoritos
   - Eliminar favorito
   - Navegar desde favoritos a detalle

7. **Descuentos**
   - Verificar cálculo de precio con descuento
   - Validar precio final en carrito

---

## 19. Datos de Prueba

### Cargar en Strapi (Content Manager)

#### Categorías
1. **Laptops** - Computadoras portátiles
2. **Smartphones** - Teléfonos móviles
3. **Tablets** - Tablets y iPads
4. **Accesorios** - Accesorios tecnológicos

#### Marcas
1. **Apple**
2. **Samsung**
3. **HP**
4. **Lenovo**
5. **Sony**

#### Productos (Ejemplos)
1. **Laptop HP Pavilion 15**
   - Categoría: Laptops
   - Marca: HP
   - Precio: 750
   - Descuento: 10
   - Stock: 15
   - Specs: `{"procesador": "Intel i5", "ram": "8GB", "almacenamiento": "256GB SSD"}`

2. **iPhone 14 Pro**
   - Categoría: Smartphones
   - Marca: Apple
   - Precio: 999
   - Descuento: 0
   - Stock: 20
   - Specs: `{"pantalla": "6.1 pulgadas", "camara": "48MP", "bateria": "3200mAh"}`

3. **Samsung Galaxy Tab S8**
   - Categoría: Tablets
   - Marca: Samsung
   - Precio: 599
   - Descuento: 15
   - Stock: 10
   - Specs: `{"pantalla": "11 pulgadas", "procesador": "Snapdragon 8", "ram": "8GB"}`

---

## 20. Troubleshooting

### Errores Comunes

#### Backend no conecta a PostgreSQL
- Verificar que PostgreSQL esté corriendo
- Revisar credenciales en `.env`
- Comprobar que la base de datos existe en pgAdmin

#### Error de CORS en mobile
- Asegurarse de que Strapi esté en `http://localhost:1337`
- Si usas dispositivo físico, cambiar `localhost` por IP local (ej: `192.168.1.100`)
- Verificar configuración en `backend/config/middlewares.js`

#### Imágenes no se muestran
- Verificar que la URL base sea correcta (`http://localhost:1337`)
- Comprobar que los productos tengan imágenes subidas
- Revisar la estructura de populate en las queries

#### Token expirado
- Los tokens JWT de Strapi expiran. Implementar refresh token o re-login

#### App no carga en Expo
- Limpiar cache: `npx expo start --clear`
- Reinstalar dependencias: `rm -rf node_modules && npm install`
- Verificar versión de Node (18+)

---

## 21. Próximos Pasos (Post-MVP)

### Mejoras Sugeridas
1. **Paginación**: Implementar scroll infinito en listado de productos
2. **Búsqueda**: Agregar barra de búsqueda con filtros avanzados
3. **Notificaciones**: Push notifications para estado de órdenes
4. **Pasarela de Pago**: Integrar Stripe/MercadoPago
5. **Historial de Órdenes**: Pantalla para ver órdenes anteriores
6. **Reseñas**: Sistema de reviews y ratings
7. **Wishlist vs Carrito**: Separar favoritos de wishlist
8. **Dark Mode**: Tema oscuro
9. **Testing**: Unit tests con Jest
10. **CI/CD**: Deploy automático

---

## 22. Recursos y Documentación

### Documentación Oficial
- **Strapi**: https://docs.strapi.io
- **Expo**: https://docs.expo.dev
- **React Navigation**: https://reactnavigation.org/docs
- **TypeScript**: https://www.typescriptlang.org/docs
- **PostgreSQL**: https://www.postgresql.org/docs

### Tutoriales Recomendados
- Strapi + PostgreSQL: https://strapi.io/blog/postgre-sql
- React Native Best Practices: https://reactnative.dev/docs/getting-started

---

## Notas Finales

### Principios de Código SIMPLE

Este proyecto sigue el principio de **KISS (Keep It Simple, Stupid)**:

- **Sin Redux**: Context API es suficiente para este alcance
- **Sin librerías UI complejas**: StyleSheet nativo
- **Fetch nativo**: Sin Axios ni SWR
- **Componentes funcionales**: Hooks básicos (useState, useEffect, useContext)
- **Navegación básica**: Stack + Tabs, sin navegación anidada compleja
- **Sin over-engineering**: Código directo y mantenible

### Estructura Modular
- Cada pantalla es autocontenida
- Servicios de API centralizados
- Context para estado global mínimo
- Tipos TypeScript para seguridad

### Mantenibilidad
- Código legible y comentado
- Separación de concerns
- Fácil de extender y modificar
- Sin dependencias innecesarias

---

**Proyecto creado para Licenciatura en Sistemas de Información - Arq. Programación Móvil 2025**
