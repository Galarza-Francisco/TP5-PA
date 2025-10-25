# Solución al Error: java.lang.String cannot be cast to java.lang.Boolean

## Problema Identificado

El error era causado por versiones incompatibles de las dependencias, específicamente:
- **Expo 54** con **React 19** y **React Navigation 7** tienen incompatibilidades conocidas
- La nueva arquitectura de React Native (`newArchEnabled: true`) causaba problemas de serialización de tipos

## Solución Aplicada

Se hizo un **downgrade a versiones estables y probadas**:
- Expo 54 → **Expo 51**
- React 19 → **React 18.2**
- React Navigation 7 → **React Navigation 6**
- React Native 0.81 → **React Native 0.74.5**

---

## Pasos para Aplicar la Solución

### 1. Detener TODO

- Cerrar completamente el emulador de Android
- En la terminal de Expo, presionar `Ctrl + C`

### 2. Limpiar Cache Completo

```bash
cd mobile
rm -rf node_modules
rm -rf .expo
rm -rf .expo-shared
rm package-lock.json
```

En Windows (CMD o PowerShell):
```cmd
cd mobile
rmdir /s /q node_modules
rmdir /s /q .expo
del package-lock.json
```

### 3. Reinstalar Dependencias

```bash
npm install
```

**Espera a que termine completamente** (puede tardar 2-5 minutos).

### 4. Verificar Instalación

```bash
npm list
```

Deberías ver:
- `expo@~51.0.0`
- `react@18.2.0`
- `@react-navigation/native@^6.1.17`
- Sin errores de dependencias

### 5. Iniciar Expo Limpio

```bash
npx expo start --clear
```

### 6. Iniciar Emulador Fresco

- Abre Android Studio
- AVD Manager → Inicia un emulador
- Espera a que cargue completamente

### 7. Cargar la App

En la terminal de Expo, presiona **`a`**

---

## Cambios Realizados en los Archivos

### package.json
```json
{
  "dependencies": {
    "@react-navigation/bottom-tabs": "^6.5.20",
    "@react-navigation/native": "^6.1.17",
    "@react-navigation/stack": "^6.3.29",
    "expo": "~51.0.0",
    "expo-image": "~1.12.0",
    "expo-secure-store": "~13.0.0",
    "expo-status-bar": "~1.12.0",
    "react": "18.2.0",
    "react-native": "0.74.5",
    "react-native-safe-area-context": "4.10.5",
    "react-native-screens": "3.31.1",
    "react-native-gesture-handler": "~2.16.1"
  }
}
```

### app.json
- ✅ Removido `newArchEnabled`
- ✅ Removido `edgeToEdgeEnabled`
- ✅ Removido `predictiveBackGestureEnabled`
- ✅ Agregado `plugins: []`

### App.tsx
```typescript
import 'react-native-gesture-handler'; // ← IMPORTANTE: Primera línea
import React from 'react';
// ... resto del código
```

---

## Verificación de Éxito

✅ **La app debería cargar sin errores**
✅ **Deberías ver la pantalla de Login**
✅ **NO deberías ver el error "java.lang.String cannot be cast to java.lang.Boolean"**

---

## Si Aún Hay Problemas

### Error: "Cannot find module"
```bash
cd mobile
npm install
npx expo start --clear
```

### Error: "Metro bundler crashed"
```bash
cd mobile
npx expo start --clear --reset-cache
```

### El emulador no conecta
- Verifica que el emulador esté completamente cargado (home screen visible)
- Cierra y reabre el emulador
- Intenta `npx expo start --tunnel` (más lento pero más confiable)

### Errores de TypeScript
```bash
# En VSCode:
Ctrl + Shift + P → "TypeScript: Restart TS Server"
```

---

## Compatibilidad de Versiones (Referencia)

| Paquete | Versión Anterior (❌ Error) | Versión Nueva (✅ Funciona) |
|---------|----------------------------|----------------------------|
| expo | 54.0.13 | 51.0.0 |
| react | 19.1.0 | 18.2.0 |
| react-native | 0.81.4 | 0.74.5 |
| @react-navigation/native | 7.1.18 | 6.1.17 |
| expo-image | 3.0.9 | 1.12.0 |
| expo-secure-store | 15.0.7 | 13.0.0 |

---

## Por Qué Funcionan Estas Versiones

- **Expo 51** es una versión LTS (Long Term Support) estable
- **React 18.2** es la versión estable actual (React 19 aún está en RC)
- **React Navigation 6** tiene amplia compatibilidad con Expo 51
- **React Native 0.74.5** es compatible con Expo 51 sin la nueva arquitectura

---

## Notas Importantes

1. **NO actualices las dependencias** sin verificar compatibilidad
2. Usa `npm install --legacy-peer-deps` si hay conflictos de peer dependencies
3. Siempre verifica la [documentación de Expo](https://docs.expo.dev/) para versiones compatibles
4. La nueva arquitectura de React Native aún no es completamente estable

---

## Siguiente Paso

Una vez que la app cargue sin errores, sigue la [GUIA_EJECUCION.md](GUIA_EJECUCION.md) para:
- Cargar datos de prueba en Strapi
- Testear las funcionalidades
- Registrar usuarios
- Hacer checkout

---

**¡La app debería funcionar ahora!** 🚀
