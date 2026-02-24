# MEDIUM - Marketplace de Servicios Locales

## Overview
MEDIUM es una plataforma de marketplace de servicios locales que conecta clientes con proveedores de servicios verificados en México. El sistema cuenta con tres roles de usuario (Cliente, Negocio, Admin), un modelo de suscripción con tres planes (Gratis, Pro $299 MXN/mes, Premium $799 MXN/mes), flujo de verificación de negocios, solicitudes de servicio con mensajería, sistema de reseñas/calificaciones e integración con Stripe.

## Arquitectura

### Stack Tecnológico
- **Frontend**: React 18 + TypeScript + Vite
- **Backend**: Express.js + TypeScript
- **Base de Datos**: PostgreSQL con Drizzle ORM
- **Autenticación**: Replit Auth (Google, GitHub, email/password)
- **Pagos**: Stripe (suscripciones + recarga de billetera legacy)
- **Estilos**: Tailwind CSS + shadcn/ui

### Estructura del Proyecto
```
├── client/                 # Frontend React
│   ├── src/
│   │   ├── components/     # Componentes reutilizables
│   │   ├── pages/          # Páginas de la aplicación
│   │   ├── hooks/          # Hooks personalizados
│   │   └── lib/            # Utilidades
├── server/                 # Backend Express
│   ├── routes.ts           # Rutas API
│   ├── storage.ts          # Capa de acceso a datos
│   ├── seed.ts             # Datos semilla
│   └── db.ts               # Conexión a base de datos
├── shared/                 # Código compartido
│   ├── schema.ts           # Esquema de base de datos Drizzle
│   └── models/             # Modelos de autenticación
```

### Base de Datos (PostgreSQL)
10 tablas principales:
- **categories**: Categorías de servicios
- **user_profiles**: Perfiles de usuario con roles
- **businesses**: Información de negocios (incluye latitude/longitude para geolocalización)
- **business_services**: Servicios estructurados con precios (nombre, descripción, minPrice, maxPrice)
- **wallets**: Billeteras de negocios
- **wallet_transactions**: Historial de transacciones
- **service_requests**: Solicitudes de servicios
- **messages**: Mensajes entre usuarios
- **reviews**: Reseñas y calificaciones
- **verifications**: Documentos de verificación

## Modelo de Negocio (Suscripciones)
- **Plan Gratis**: Perfil básico, recibir solicitudes, chat con clientes
- **Plan Pro ($299 MXN/mes)**: Badge Pro azul, prioridad en resultados, estadísticas de visitas
- **Plan Premium ($799 MXN/mes)**: Badge Premium dorado, máxima prioridad, estadísticas avanzadas, soporte VIP
- Los negocios se ordenan por: plan (Premium > Pro > Gratis), luego por rating/distancia
- Suscripciones gestionadas con Stripe Checkout + Billing Portal
- Webhooks para manejar: checkout.session.completed, customer.subscription.updated/deleted, invoice.payment_succeeded/failed
- Tabla: business_subscriptions (plan, status, stripeSubscriptionId, stripeCustomerId, currentPeriodEnd)
- Sistema de billetera legacy sigue existente pero ya no bloquea negocios ni cobra por leads

## Roles de Usuario
1. **Cliente**: Busca servicios, solicita cotizaciones, deja reseñas
2. **Negocio**: Recibe solicitudes, gestiona perfil, recarga billetera
3. **Admin**: Verifica documentos, gestiona usuarios y negocios

## Flujo de Verificación
1. El negocio sube INE + comprobante de domicilio
2. Admin revisa documentos
3. Si ambos son aprobados, el negocio recibe sello de verificación

## Rutas API Principales
- `GET /api/categories` - Lista de categorías
- `GET /api/businesses` - Búsqueda de negocios (incluye plan)
- `GET /api/businesses/:slug` - Perfil de negocio (incluye plan)
- `GET /api/businesses/:slugOrId/plan` - Info del plan de un negocio
- `POST /api/requests` - Crear solicitud de servicio (sin cobro de lead)
- `GET /api/dashboard` - Dashboard de negocio (incluye subscription)
- `GET /api/subscription` - Info de suscripción del negocio actual
- `POST /api/subscription/checkout` - Crear sesión Stripe Checkout para suscripción
- `POST /api/subscription/portal` - Crear sesión Stripe Billing Portal
- `POST /api/subscription/trial` - Iniciar período de prueba
- `POST /api/wallet/recharge` - Recargar billetera (legacy)
- `GET /api/admin/*` - Rutas de administración

## Páginas Frontend
- `/pricing` - Página de planes y precios con 3 tiers
- `/dashboard` con tab "Mi Plan" para gestionar suscripción

## Diseño
- Paleta de colores: Teal/Blue profesional (primario: hsl(173 58% 39%))
- Interfaz en español
- Soporte para modo oscuro
- Diseño responsivo

## PWA (Progressive Web App)
- Instalable en dispositivos móviles y escritorio
- Service worker con caché offline para navegación básica
- Iconos PWA en 8 tamaños (72px a 512px)
- Prompt de instalación personalizado

## Mapa de Negocios
- Vista de mapa integrada con Leaflet/OpenStreetMap
- Marcadores personalizados con iconos teal
- Geolocalización del usuario con prompt de permisos
- Popups interactivos con información del negocio (sanitizados contra XSS)
- Toggle lista/mapa en página de búsqueda
- Ordenamiento por distancia cuando el usuario comparte su ubicación
- Geocodificación automática de direcciones usando OpenStreetMap Nominatim

## Perfil del Cliente
- Página /profile para visualizar historial de servicios
- Sección de servicios completados con opción de dejar reseñas
- Edición y eliminación de reseñas propias con menú desplegable
- Sección de servicios activos/pendientes
- Compartir negocios por WhatsApp, Facebook o copiar enlace
- Redirección automática: negocios → /dashboard, admins → /admin

## Lista de Chats
- Página /chats accesible desde menú hamburguesa para usuarios autenticados
- Muestra todas las conversaciones con último mensaje
- Ordenamiento por mensaje más reciente
- Vista diferenciada para clientes y negocios
- API: GET /api/chats retorna solicitudes con mensajes

## Administración
- Admin automático: el email diegocleta6@gmail.com obtiene rol admin al iniciar sesión
- Panel de admin en /admin con métricas consolidadas
- Gestión de verificaciones de negocios
- Moderación de reseñas sospechosas
- Historial de transacciones de billetera

## Registro de Negocios
- Carga de logo/imagen de portada (1 imagen)
- Galería de trabajos anteriores (hasta 5 imágenes)
- Servicios estructurados con precios mínimos y máximos
- Selector de ubicación interactivo con mapa (LocationPicker)
  - Mapa interactivo con Leaflet/OpenStreetMap
  - Botón GPS para usar ubicación actual
  - Búsqueda de direcciones
  - Marcador arrastrable para ajuste preciso
  - Geocodificación inversa para mostrar dirección seleccionada

## Visor de Imágenes (ImageViewer)
- Galería clickeable en perfil de negocio
- Modal fullscreen con fondo oscuro
- Controles de zoom (botones + rueda del mouse)
- Navegación entre imágenes con flechas
- Indicadores de posición (dots)
- Arrastrar imagen cuando está ampliada
- Atajos de teclado (Escape, flechas, +/-)
- Información de zoom y posición actual

## Notificaciones por Email (Resend)
- Notificación al negocio cuando recibe nueva solicitud
- Notificación cuando un negocio es verificado
- Alerta de saldo bajo (menos de $40 MXN)
- Requiere configurar integración Resend

## Transacciones Atómicas
- Creación de solicitud + deducción de billetera en una transacción
- Previene inconsistencias si falla alguna operación
- Actualización automática de visibilidad del negocio

## Panel de Administración Mejorado
- Pestaña de historial de transacciones de billetera
- Últimas 100 transacciones con detalles del negocio
- Montos codificados por color (verde/rojo/gris)
- Métricas consolidadas de usuarios, negocios y solicitudes

## Mensajería con Imágenes
- Soporte para enviar hasta 3 imágenes por mensaje en chat
- Límite de 5MB por imagen, solo formatos de imagen (sin SVG)
- Validación del lado del servidor para prevenir XSS
- Vista previa de imágenes con opción de ampliar

## Control de Negocios
- Botón de pausar/reanudar negocio en dashboard
- Contador de vistas del perfil del negocio
- Toggle de visibilidad independiente del saldo

## Sistema Anti-Spam de Reseñas
- Rate limiting: máximo 5 reseñas por hora por IP
- Detección de actividad sospechosa: múltiples reseñas en 7 días
- Auto-verificación: reseñas de servicios completados se marcan como verificadas
- Campos adicionales en reseñas: isVerified, isSuspicious, suspiciousReason, ipAddress
- Panel de admin muestra reseñas sospechosas con opción de eliminar
- Badge "Verificada" en perfiles de negocio para reseñas de servicios completados

## Sistema de Favoritos
- Botón de corazón en perfil de negocio para agregar/eliminar favoritos
- Tab "Favoritos" en perfil del cliente con lista de negocios guardados
- Enlace en menú móvil a favoritos (/profile?tab=favorites)
- API: GET /api/favorites, POST/DELETE /api/favorites/:businessId, GET /api/favorites/check/:businessId

## Sistema de Reportes
- Componente ReportDialog reutilizable para reportar negocios, chats o mensajes
- Razones predefinidas: Contenido inapropiado, Información falsa, Spam, Acoso, Otro
- Botón de reportar (bandera) en perfil de negocio
- Tab "Reportes" en panel de admin con acciones de revisar/descartar
- API: POST /api/reports, GET/PATCH /api/admin/reports

## Filtros Avanzados de Búsqueda
- Filtro por calificación mínima (estrellas)
- Toggle "Solo verificados"
- Rango de precios (mínimo/máximo en MXN)
- Subcategorías jerárquicas con expansión/colapso
- Botón "Aplicar filtros" con estado draft/aplicado
- Disponible en sidebar desktop y Sheet móvil

## Dashboard Visual con Gráficas
- Gráfica de barras: leads semanales (últimas 8 semanas)
- Gráfica de área: ingresos semanales (últimas 8 semanas)
- Usando recharts con colores del tema
- API: GET /api/dashboard/charts

## Gestión de Estado de Solicitudes
- Negocio puede aceptar (in_progress) y completar solicitudes
- Cliente puede cancelar solicitudes con confirmación
- Badges de estado con colores diferenciados
- API: PATCH /api/requests/:id/status

## Perfil de Cliente Mejorado
- Edición de perfil (nombre, teléfono, WhatsApp, dirección)
- Carga de foto de perfil con compresión
- Tab de favoritos con lista de negocios guardados
- Botón de cancelar en solicitudes activas

## Comandos de Desarrollo
```bash
npm run dev          # Iniciar servidor de desarrollo
npm run db:push      # Sincronizar esquema de base de datos
```

## Preferencias del Usuario
- Idioma: Español (México)
- Moneda: MXN (Pesos mexicanos)
- Tema por defecto: Claro
