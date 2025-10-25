# Guía de Ejecución y Testing - App Tienda Tech

## Estado del Proyecto

✅ **Backend (Strapi)**: Configurado con PostgreSQL
✅ **Mobile (React Native + Expo)**: Código implementado
✅ **Estructura**: Completa
⚠️ **Correcciones aplicadas**: ProductoDetalleScreen.jsx → ProductoDetalleScreen.tsx

---

## Correcciones Realizadas

### 1. Extensión de Archivo Corregida
- **Problema**: `ProductoDetalleScreen.jsx` usaba TypeScript pero tenía extensión `.jsx`
- **Solución**: Renombrado a `ProductoDetalleScreen.tsx`
- **Status**: ✅ Corregido

### 2. Estructura Verificada
Todos los archivos necesarios están presentes:
- ✅ `src/types/index.ts` - Tipos TypeScript
- ✅ `src/utils/constants.ts` - Constantes de API
- ✅ `src/context/AuthContext.tsx` - Autenticación
- ✅ `src/context/CarritoContext.tsx` - Carrito de compras
- ✅ `src/services/api.ts` - Servicios de API
- ✅ `src/navigation/AppNavigator.tsx` - Navegación
- ✅ `src/screens/*.tsx` - Todas las pantallas
- ✅ `App.tsx` - Punto de entrada

---

## Cómo Ejecutar el Proyecto

### **PASO 1: Iniciar el Backend (Strapi)**

Abre una **terminal** (CMD o PowerShell) y ejecuta:

```bash
cd backend
npm run develop
```

**Espera a que veas:**
```
[2025-10-17 XX:XX:XX] ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
[2025-10-17 XX:XX:XX] ┃ Strapi is running ✨                  ┃
[2025-10-17 XX:XX:XX] ┃ ──────────────────────────────────────  ┃
[2025-10-17 XX:XX:XX] ┃ Local:  http://localhost:1337/admin    ┃
[2025-10-17 XX:XX:XX] ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

✅ **Strapi está corriendo** cuando veas este mensaje.

**No cierres esta terminal**. Déjala abierta todo el tiempo.

---

### **PASO 2: Cargar Datos de Prueba en Strapi**

Antes de usar la app móvil, necesitas cargar productos de prueba.

#### Opción A: Cargar Manualmente desde el Panel

1. Abrir en navegador: `http://localhost:1337/admin`
2. Iniciar sesión con tu usuario admin
3. Ir a **Content Manager**

#### Cargar Categorías:
1. Click en **Categoria** → **Create new entry**
2. Crear al menos 2 categorías:
   - **Nombre**: Laptops, **Descripción**: Computadoras portátiles
   - **Nombre**: Smartphones, **Descripción**: Teléfonos móviles
3. Click **Save** y **Publish** cada una

#### Cargar Marcas:
1. Click en **Marca** → **Create new entry**
2. Crear al menos 2 marcas:
   - **Nombre**: HP
   - **Nombre**: Apple
3. Click **Save** y **Publish** cada una

#### Cargar Productos:
1. Click en **Producto** → **Create new entry**
2. Llenar los campos:
   - **Nombre**: Laptop HP Pavilion 15
   - **Descripción**: Laptop potente para trabajo y estudio
   - **Precio**: 750
   - **Descuento**: 10
   - **Stock**: 15
   - **Activo**: ✅ (checked)
   - **Categoría**: Seleccionar "Laptops"
   - **Marca**: Seleccionar "HP"
   - **Especificaciones** (JSON):
     ```json
     {
       "procesador": "Intel i5",
       "ram": "8GB",
       "almacenamiento": "256GB SSD"
     }
     ```
   - **Imágenes**: Opcional (puedes subir o dejarlo vacío por ahora)
3. Click **Save** y **Publish**

**Crear al menos 3-5 productos** para tener contenido en la app.

---

### **PASO 3: Verificar que la API Funcione**

Antes de ejecutar la app móvil, verifica que Strapi responda correctamente.

Abre en tu navegador:
```
http://localhost:1337/api/productos?populate=*
```

**Deberías ver un JSON** con tus productos. Ejemplo:
```json
{
  "data": [
    {
      "id": 1,
      "attributes": {
        "nombre": "Laptop HP Pavilion 15",
        "precio": 750,
        "stock": 15,
        ...
      }
    }
  ],
  "meta": { ... }
}
```

✅ Si ves esto, la API está funcionando correctamente.

❌ Si ves error 403 (Forbidden), revisa los **permisos en Settings → Roles → Public**.

---

### **PASO 4: Iniciar la App Móvil (Expo)**

Abre una **segunda terminal** (nueva ventana de CMD o PowerShell):

```bash
cd mobile
npx expo start
```

**Espera a que veas:**
```
› Metro waiting on exp://192.168.x.x:8081
› Scan the QR code above with Expo Go (Android) or the Camera app (iOS)

› Press a | open Android
› Press i | open iOS simulator
› Press w | open web
```

---

### **PASO 5: Ejecutar en Dispositivo/Emulador**

Tienes 3 opciones:

#### **Opción A: En el Navegador Web** (más fácil para testing rápido)
1. Presiona **`w`** en la terminal de Expo
2. Se abrirá en tu navegador: `http://localhost:8081`
3. ✅ Listo! Deberías ver la pantalla de Login

#### **Opción B: En tu Celular** (Android/iOS)
1. Instalar **Expo Go** desde Play Store o App Store
2. Abrir Expo Go en tu celular
3. Escanear el código QR que aparece en la terminal
4. ⚠️ **IMPORTANTE**: Tu celular y PC deben estar en la misma red WiFi

**Problema común**: Si no carga, edita [mobile/src/utils/constants.ts](mobile/src/utils/constants.ts):
```typescript
// Cambiar localhost por la IP de tu PC
export const API_URL = 'http://192.168.1.X:1337/api';  // Reemplaza X con tu IP local
export const BASE_URL = 'http://192.168.1.X:1337';
```

Para saber tu IP local:
```bash
ipconfig  # En Windows
ifconfig  # En Mac/Linux
```
Busca la línea "IPv4 Address" (ej: 192.168.1.105)

#### **Opción C: En Emulador Android** (requiere Android Studio)
1. Presiona **`a`** en la terminal de Expo
2. Abrirá automáticamente el emulador

---

## Testing de Funcionalidades

### **Test 1: Registro de Usuario**

1. Al abrir la app, verás la pantalla **Login**
2. Click en "¿No tienes cuenta? Regístrate"
3. Llenar:
   - **Usuario**: testuser
   - **Email**: test@example.com
   - **Password**: password123
4. Click en "Crear Cuenta"

✅ **Resultado esperado**: Sesión iniciada automáticamente, redirige a pantalla de productos

❌ **Si falla**: Verifica que en Strapi → Settings → Users & Permissions → Advanced Settings esté habilitado "Enable sign-ups"

---

### **Test 2: Ver Productos**

1. En la pantalla **Inicio**, deberías ver un grid de productos
2. Cada producto muestra:
   - Imagen (o placeholder si no tiene)
   - Nombre
   - Precio
   - Descuento (si aplica)

✅ **Resultado esperado**: Lista de productos cargados desde Strapi

❌ **Si aparece vacío**: Verifica que tengas productos publicados en Strapi

❌ **Si aparece error**:
- Revisa la consola de la terminal de Expo
- Verifica que Strapi esté corriendo
- Revisa la URL en [constants.ts](mobile/src/utils/constants.ts)

---

### **Test 3: Detalle de Producto**

1. Click en cualquier producto del grid
2. Deberías ver:
   - Imagen grande
   - Nombre, descripción
   - Precio con descuento aplicado
   - Stock disponible
   - Especificaciones (si las tiene)
   - Controles de cantidad (+/-)
   - Botón "Agregar al Carrito"

✅ **Resultado esperado**: Pantalla de detalle completa

3. Ajustar cantidad con +/-
4. Click en "Agregar al Carrito"

✅ **Resultado esperado**: Alert "Producto agregado al carrito"

---

### **Test 4: Carrito de Compras**

1. Ir al tab **Carrito** (ícono inferior)
2. Deberías ver los productos agregados
3. Probar:
   - **Modificar cantidad** con +/-
   - **Eliminar producto** con el ícono ✕
4. Verificar que el **Total** se actualice correctamente

✅ **Resultado esperado**: Carrito funcional con cálculos correctos

---

### **Test 5: Checkout (Crear Orden)**

1. Con productos en el carrito, click en **"Finalizar Compra"**
2. Esperar respuesta del servidor

✅ **Resultado esperado**:
- Alert "Orden creada correctamente"
- Carrito se vacía automáticamente

**Verificar en Strapi:**
1. Ir a `http://localhost:1337/admin`
2. Content Manager → **Orden**
3. Deberías ver la orden creada con los productos

---

### **Test 6: Favoritos**

1. En detalle de producto, agregar a favoritos (⭐ ícono, si lo implementaste)
2. Ir al tab **Favoritos**
3. Verificar que aparezca el producto

✅ **Resultado esperado**: Lista de favoritos

4. Click en ✕ para eliminar

✅ **Resultado esperado**: Favorito eliminado

---

### **Test 7: Perfil y Logout**

1. Ir al tab **Perfil**
2. Verificar que muestre:
   - Usuario: testuser
   - Email: test@example.com
3. Click en **"Cerrar Sesión"**

✅ **Resultado esperado**: Redirige a pantalla de Login

---

## Problemas Comunes y Soluciones

### Error: "Network request failed"

**Causa**: La app no puede conectarse al backend Strapi

**Soluciones**:
1. Verificar que Strapi esté corriendo en `http://localhost:1337`
2. Si usas celular físico, cambiar `localhost` por tu IP local en [constants.ts](mobile/src/utils/constants.ts)
3. Verificar que no haya firewall bloqueando el puerto 1337

---

### Error: "Cannot connect to Metro"

**Causa**: Expo no puede iniciar el bundler

**Soluciones**:
1. Cerrar Expo y volver a ejecutar:
   ```bash
   npx expo start --clear
   ```
2. Verificar que no haya otro proceso usando el puerto 8081

---

### Error: TypeScript - "Cannot find module"

**Causa**: VSCode no reconoce los tipos o imports

**Soluciones**:
1. Reiniciar VSCode: Ctrl+Shift+P → "Reload Window"
2. Verificar que exista [mobile/tsconfig.json](mobile/tsconfig.json)
3. Ejecutar:
   ```bash
   cd mobile
   npm install
   ```

---

### Productos no aparecen en la app

**Causa**: No hay productos publicados o permisos incorrectos

**Soluciones**:
1. Verificar en Strapi Content Manager que los productos estén **Published** (no Draft)
2. Verificar permisos en Settings → Roles → Public:
   - ✅ Producto: find, findOne
3. Probar la API directamente en navegador:
   ```
   http://localhost:1337/api/productos?populate=*
   ```

---

### Imágenes no se muestran

**Causa**: URLs incorrectas o imágenes no subidas

**Soluciones**:
1. Verificar que [BASE_URL](mobile/src/utils/constants.ts) sea correcta: `http://localhost:1337`
2. Si no subiste imágenes, aparecerá un placeholder (normal)
3. Subir imágenes en Strapi Content Manager → Producto → Campo "imagenes"

---

### Error 403 (Forbidden) en la API

**Causa**: Permisos no configurados correctamente

**Soluciones**:
1. Ir a Strapi: Settings → Users & Permissions Plugin → Roles
2. Configurar **Public**:
   - ✅ Categoria: find, findOne
   - ✅ Marca: find, findOne
   - ✅ Producto: find, findOne
3. Configurar **Authenticated**:
   - ✅ Todo lo de Public
   - ✅ Orden: create, find, findOne
   - ✅ Favorito: create, delete, find, findOne
4. Hacer click en **Save** (arriba a la derecha)

---

### App se cierra o crashea al iniciar

**Causa**: Error en algún archivo o dependencia faltante

**Soluciones**:
1. Ver logs en la terminal de Expo
2. Verificar que todas las dependencias estén instaladas:
   ```bash
   cd mobile
   npm install
   ```
3. Limpiar cache:
   ```bash
   npx expo start --clear
   ```

---

## Checklist de Verificación

Antes de probar la app, asegúrate de:

### Backend (Strapi)
- [ ] PostgreSQL corriendo
- [ ] Strapi corriendo en `http://localhost:1337`
- [ ] Content Types creados (Categoria, Marca, Producto, Orden, Favorito)
- [ ] Permisos configurados (Public y Authenticated)
- [ ] Al menos 3-5 productos publicados
- [ ] API responde en `http://localhost:1337/api/productos?populate=*`

### Mobile (App)
- [ ] Dependencias instaladas (`npm install` en mobile/)
- [ ] ProductoDetalleScreen.tsx (no .jsx)
- [ ] constants.ts con URL correcta
- [ ] Expo corriendo sin errores
- [ ] Sin errores de TypeScript en VSCode

---

## Comandos Útiles

### Backend (Strapi)
```bash
cd backend
npm run develop          # Modo desarrollo (auto-reload)
npm run build            # Compilar para producción
npm run start            # Producción (después de build)
```

### Mobile (Expo)
```bash
cd mobile
npx expo start           # Iniciar Expo
npx expo start --clear   # Limpiar cache y iniciar
npx expo start --web     # Solo web
npx expo start --android # Solo Android

npm install              # Reinstalar dependencias
```

### Ver logs en detalle
```bash
# Terminal de Expo, presiona:
j  # Abrir debugger
r  # Reload app
c  # Clear console
```

---

## Estructura de Testing Recomendada

### Flujo completo de usuario:
1. **Registro** → Crear cuenta nueva
2. **Browse** → Ver productos en Home
3. **Detalle** → Click en producto
4. **Agregar al carrito** → Con cantidad personalizada
5. **Ver carrito** → Verificar productos y total
6. **Checkout** → Crear orden
7. **Favoritos** → Agregar/eliminar favoritos
8. **Perfil** → Ver datos de usuario
9. **Logout** → Cerrar sesión
10. **Login** → Iniciar sesión con credenciales existentes

---

## Testing Avanzado (Opcional)

### Probar validaciones:
- **Stock insuficiente**: Intentar agregar más cantidad que el stock disponible
- **Carrito vacío**: Intentar hacer checkout sin productos
- **Logout y acceso**: Verificar que al hacer logout no se pueda acceder a rutas protegidas

### Probar edge cases:
- **Sin productos**: Eliminar todos los productos de Strapi y ver cómo responde la app
- **Sin conexión**: Desconectar WiFi y ver los errores
- **Productos sin imagen**: Verificar placeholders

---

## Próximos Pasos Después del Testing

Una vez que todo funcione correctamente:

### 1. Mejorar UI/UX
- Agregar iconos en tabs (usar `@expo/vector-icons`)
- Mejorar estilos de botones
- Agregar animaciones suaves

### 2. Funcionalidades Extra
- Filtros por categoría/marca en HomeScreen
- Búsqueda de productos
- Historial de órdenes
- Pantalla de "Mis Órdenes"

### 3. Optimizaciones
- Implementar pull-to-refresh
- Paginación de productos
- Caché de imágenes
- Loading states mejorados

### 4. Testing Profesional
- Unit tests con Jest
- E2E tests con Detox
- Pruebas de rendimiento

---

## Resumen de URLs Importantes

| Servicio | URL | Uso |
|----------|-----|-----|
| **Strapi Admin** | http://localhost:1337/admin | Panel de administración |
| **API Productos** | http://localhost:1337/api/productos?populate=* | Testing de API |
| **API Auth** | http://localhost:1337/api/auth/local | Login |
| **Expo Web** | http://localhost:8081 | App en navegador |
| **Expo Metro** | http://localhost:8081 | Bundler de React Native |

---

## Contacto y Soporte

Si encuentras errores específicos:

1. **Revisar logs**: Terminal de Expo y terminal de Strapi
2. **Verificar configuración**: Comparar con [PLAN_PROYECTO.md](PLAN_PROYECTO.md)
3. **Documentación oficial**:
   - Strapi: https://docs.strapi.io
   - Expo: https://docs.expo.dev
   - React Navigation: https://reactnavigation.org

---

**¡Tu app está lista para testing! 🚀**

Sigue los pasos en orden y verifica cada funcionalidad. Si algo no funciona, consulta la sección de problemas comunes.
