
# DOCUMENTACIÓN DEL SISTEMA STOCKCEL

## ANÁLISIS COMPLETO DEL PROYECTO

### 1. ARQUITECTURA DEL SISTEMA

**Tipo**: Aplicación web Full-Stack con arquitectura cliente-servidor
**Frontend**: React + TypeScript + Vite + TailwindCSS
**Backend**: Node.js + Express + TypeScript
**Base de Datos**: PostgreSQL (Neon) con Drizzle ORM
**Autenticación**: Sistema personalizado con roles y permisos

### 2. ESTRUCTURA DE CARPETAS Y ARCHIVOS

```
StockCel/
├── client/                          # Frontend React
│   ├── src/
│   │   ├── components/              # Componentes reutilizables
│   │   │   ├── currency/            # Convertidor de monedas
│   │   │   ├── dashboard/           # Panel principal
│   │   │   ├── layout/              # Layout y navegación
│   │   │   ├── orders/              # Gestión de órdenes
│   │   │   ├── products/            # Gestión de productos
│   │   │   └── ui/                  # Componentes base UI
│   │   ├── hooks/                   # Custom hooks
│   │   ├── lib/                     # Utilidades y configuración
│   │   ├── pages/                   # Páginas principales
│   │   └── stores/                  # Estado global
│   └── index.html
├── server/                          # Backend Node.js
│   ├── routes.ts                    # Rutas API principales
│   ├── storage.ts                   # Capa de acceso a datos
│   ├── index.ts                     # Servidor principal
│   ├── auto-sync-monitor.ts         # Monitor automático
│   ├── cash-storage.ts              # Gestión de caja
│   ├── email.ts                     # Sistema de emails
│   └── password-reset.ts            # Reset de contraseñas
├── shared/
│   └── schema.ts                    # Esquemas compartidos (Drizzle)
└── database_schema.txt              # Documentación de BD
```

### 3. FUNCIONALIDADES PRINCIPALES

#### 3.1 GESTIÓN DE PRODUCTOS
- **Ubicación**: `client/src/pages/products.tsx`
- **Funciones**: CRUD completo de productos (celulares)
- **Estados**: disponible, reservado, vendido, técnico_interno, técnico_externo, a_reparar, extravío
- **Campos**: IMEI, modelo, almacenamiento, color, precio, estado, proveedor, batería, calidad

#### 3.2 CONTROL DE STOCK
- **Archivos principales**:
  - `client/src/pages/stock-control-final.tsx` (PRINCIPAL)
  - `client/src/pages/stock-control.tsx`
  - `client/src/pages/stock-control-new.tsx`
  - `client/src/pages/stock-control-ultra.tsx`

#### 3.3 GESTIÓN DE ÓRDENES
- **Ubicación**: `client/src/pages/orders.tsx`
- **Funciones**: Crear, editar, gestionar órdenes de venta
- **Integración**: Con productos, clientes, vendedores, pagos

#### 3.4 SISTEMA DE CAJA
- **Ubicación**: `client/src/pages/cash-advanced.tsx`
- **Funciones**: Control de caja diario, movimientos, reportes
- **Monedas**: USD, ARS, USDT con conversiones automáticas

#### 3.5 MULTI-TENANCY
- **Sistema**: Basado en `clientId` para separar datos por empresa
- **Roles**: superuser, admin, vendor con permisos granulares

### 4. ESTRUCTURA DE BASE DE DATOS

#### 4.1 TABLAS PRINCIPALES

**Clientes (Multi-tenancy)**:
- `clients` - Empresas que usan el sistema
- `users` - Usuarios por empresa (admin, vendor)

**Inventario**:
- `products` - Productos (celulares) con todos sus datos
- `product_history` - Historial de cambios de productos

**Ventas**:
- `orders` - Órdenes de venta
- `order_items` - Productos en cada orden
- `payments` - Pagos realizados

**Caja y Finanzas**:
- `cash_register` - Estado diario de caja
- `cash_movements` - Movimientos de caja detallados
- `expenses` - Gastos del negocio
- `customer_debts` - Deudas de clientes
- `debt_payments` - Pagos de deudas

**Control de Stock**:
- `stock_control_sessions` - Sesiones de control de inventario
- `stock_control_items` - Items verificados en cada sesión

**Configuración**:
- `configuration` - Configuración por cliente
- `company_configuration` - Datos de la empresa
- `vendors` - Vendedores
- `customers` - Clientes/compradores

#### 4.2 RELACIONES CLAVE
- Todos los datos están segmentados por `client_id` (multi-tenancy)
- `products` → `product_history` (1:N)
- `orders` → `order_items` → `products` (1:N:1)
- `orders` → `payments` (1:N)
- `customers` → `customer_debts` → `debt_payments` (1:N:N)

### 5. FLUJO DE DATOS

#### 5.1 AUTENTICACIÓN
1. Login en `/api/auth/login`
2. Verificación de usuario y cliente
3. Retorno de datos de usuario y empresa
4. Headers `x-user-id` para autenticación

#### 5.2 GESTIÓN DE PRODUCTOS
1. Frontend solicita productos: `GET /api/products?clientId=X`
2. Backend filtra por `clientId` en `storage.ts`
3. Drizzle ORM consulta PostgreSQL
4. Retorno de productos filtrados

#### 5.3 VENTAS (ÓRDENES)
1. Creación de orden: `POST /api/orders`
2. Creación de items: `POST /api/order-items`
3. Procesamiento de pagos: `POST /api/payments`
4. Actualización de estado de productos
5. Movimientos automáticos en caja

### 6. ANÁLISIS ESPECÍFICO DEL SISTEMA DE STOCK

#### 6.1 ARCHIVO PRINCIPAL: `stock-control-final.tsx`

**Funcionalidades Actuales**:
- ✅ Visualización de estadísticas por sectores (Disponible, Reservado, Técnico Interno)
- ✅ Sistema de escaneo de IMEI en tiempo real
- ✅ Detección automática de productos faltantes
- ✅ Control por sesiones con progreso en tiempo real
- ✅ Análisis de inventario con gráficos y métricas

**Estados Válidos para Control**:
- `disponible` - Productos listos para venta
- `reservado` - Productos apartados
- `tecnico_interno` - Productos en reparación interna

**Flujo de Funcionamiento**:
1. **Inicio de Sesión**: Calcula productos esperados (disponible + reservado + técnico_interno)
2. **Escaneo**: Permite escanear/ingresar IMEIs uno por uno
3. **Validación**: Verifica que el IMEI existe y está en estado válido
4. **Seguimiento**: Actualiza contadores en tiempo real
5. **Finalización**: Detecta productos faltantes y permite marcarlos como extravío

#### 6.2 PROBLEMAS IDENTIFICADOS EN EL SISTEMA DE STOCK

🚨 **PROBLEMA CRÍTICO**: **NO SE GUARDAN LAS MODIFICACIONES**

**Análisis del Problema**:

1. **Falta de Persistencia en Base de Datos**:
   - El sistema actual NO utiliza las tablas `stock_control_sessions` y `stock_control_items`
   - Los datos de escaneo solo se mantienen en memoria local (useState)
   - No hay llamadas a APIs para guardar sesiones de control

2. **Funcionalidades Faltantes**:
   - ❌ No guarda sesiones de control de stock
   - ❌ No registra items escaneados en BD
   - ❌ No permite pausar y continuar sesiones
   - ❌ No hay guardado por sectores independientes
   - ❌ No hay historial de controles previos

3. **APIs Disponibles pero No Utilizadas**:
   ```typescript
   // APIs existentes en routes.ts pero NO usadas en el frontend:
   POST /api/stock-control/sessions     // Crear sesión
   GET /api/stock-control/sessions/:id  // Obtener sesión
   POST /api/stock-control/scan         // Registrar escaneo
   PUT /api/stock-control/sessions/:id/finish // Finalizar sesión
   ```

#### 6.3 BACKEND INVOLUCRADO

**Archivos del Backend**:
1. **`server/routes.ts`** (líneas ~1400-1600):
   - Rutas completas para stock control implementadas
   - Métodos para crear sesiones, escanear, finalizar

2. **`server/storage.ts`** (DrizzleStorage):
   - Métodos implementados para stock control
   - `createStockControlSession()`, `createStockControlItem()`, etc.

3. **`shared/schema.ts`**:
   - Tablas `stock_control_sessions` y `stock_control_items` definidas
   - Esquemas de validación implementados

**El problema es que el frontend NO ESTÁ CONECTADO al backend para estas funcionalidades.**

### 7. SOLUCIÓN REQUERIDA

Para corregir el problema de "No se guardan las modificaciones", se necesita:

1. **Conectar Frontend con Backend**:
   - Modificar `stock-control-final.tsx` para usar las APIs existentes
   - Implementar llamadas para crear/guardar sesiones
   - Guardar cada escaneo en la base de datos

2. **Implementar Funcionalidades Faltantes**:
   - Guardado por sectores (Técnico interno, Disponibles, Reservados)
   - Sistema de pausa/continuación
   - Persistencia de sesiones entre sesiones del navegador

3. **Mantener Funcionalidades Existentes**:
   - Preservar toda la lógica de detección de extraviados
   - Mantener la interfaz y experiencia de usuario actual
   - No alterar el funcionamiento que ya está operativo

### 8. ESTADO ACTUAL DEL PROYECTO

**✅ Funciona Correctamente**:
- Sistema de autenticación y roles
- Gestión completa de productos
- Órdenes y ventas
- Sistema de caja avanzado
- Multi-tenancy
- Backend completo para stock control

**🚨 Requiere Corrección**:
- **Persistencia en stock control** (problema principal)
- Conexión frontend-backend para stock control
- Funcionalidades de pausa/guardado por sectores

**📊 Métricas del Sistema**:
- **26 tablas** en base de datos
- **52 relaciones/secuencias** 
- **Conexión a Neon PostgreSQL** funcionando
- **Multi-tenancy** operativo para múltiples clientes

### 9. RECOMENDACIONES

1. **Prioridad ALTA**: Corregir sistema de guardado en stock-control-final.tsx
2. **Prioridad MEDIA**: Implementar funcionalidades de pausa y sectores
3. **Prioridad BAJA**: Optimizar UI y experiencia de usuario

El sistema está muy bien estructurado y el backend está completo. Solo necesita conectar el frontend de stock control con las APIs ya existentes.
