# MercadoLibre Pricing Platform

Sistema de gestión de precios para publicaciones en MercadoLibre, con sincronización automática desde Air Intra.

## Características

- 📊 Gestión dinámica de parámetros de pricing
- 🔄 Sincronización automática con Air Intra API
- 🛍️ Publicación selectiva en MercadoLibre
- 💰 Cálculo automático de precios con dólar BNA
- 📈 Actualizaciones automáticas de precio/stock
- 🔐 Autenticación segura con JWT
- 📧 Notificaciones por email de errores críticos

## Stack Tecnológico

- **Frontend:** Next.js 14 (App Router), React 18, Tailwind CSS
- **Backend:** Next.js API Routes (serverless)
- **Database:** PostgreSQL (Supabase)
- **ORM:** Prisma
- **Auth:** JWT con httpOnly cookies
- **Email:** Resend
- **Deployment:** Vercel

## Comenzar en Desarrollo

1. **Instalar dependencias:**
   ```bash
   npm install
   ```

2. **Configurar variables de entorno:**
   ```bash
   cp .env.example .env
   ```

   Editar `.env` con tus credenciales.

3. **Configurar base de datos:**
   ```bash
   npm run db:push
   ```

4. **Ejecutar servidor de desarrollo:**
   ```bash
   npm run dev
   ```

   Abrir [http://localhost:3000](http://localhost:3000)

## Scripts Disponibles

- `npm run dev` - Inicia el servidor de desarrollo
- `npm run build` - Construye la aplicación para producción
- `npm start` - Ejecuta el servidor de producción
- `npm run lint` - Ejecuta el linter
- `npm run db:generate` - Genera Prisma Client
- `npm run db:push` - Sincroniza el esquema con la base de datos
- `npm run db:migrate` - Crea y aplica migraciones
- `npm run db:studio` - Abre Prisma Studio
- `npm test` - Ejecuta tests unitarios
- `npm run test:e2e` - Ejecuta tests E2E

## Estructura del Proyecto

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Rutas de autenticación
│   ├── (dashboard)/       # Rutas del dashboard
│   └── api/               # API routes
├── components/            # Componentes de React
│   ├── ui/               # Componentes base (shadcn/ui)
│   ├── dashboard/        # Componentes del dashboard
│   ├── products/         # Componentes de productos
│   ├── pricing/          # Componentes de pricing
│   └── layout/           # Componentes de layout
├── lib/                  # Lógica de negocio
│   ├── prisma.ts        # Cliente de Prisma
│   ├── auth.ts          # Utilidades de autenticación
│   ├── air-intra.ts     # Cliente Air Intra API
│   ├── mercadolibre.ts  # Cliente ML API
│   ├── pricing.ts       # Motor de cálculo de precios
│   └── dolar.ts         # Scraping BNA
├── hooks/                # React hooks personalizados
└── __tests__/           # Tests
    ├── unit/            # Tests unitarios
    └── e2e/             # Tests E2E
```

## Deployment

El proyecto está configurado para deploy automático en Vercel.

1. **Conectar repositorio GitHub a Vercel**
2. **Configurar variables de entorno en Vercel**
3. **Deployar automáticamente al hacer push a main**

Ver [claude-plan.md](./ml-pricing-platform-planning/claude-plan.md) para documentación completa de implementación.

## Seguridad

- Todos los tokens OAuth están encriptados en la base de datos
- JWT con httpOnly cookies para sesiones
- Rate limiting con p-queue para APIs externas
- Distributed locks para prevenir race conditions
- Validación de datos con Zod (próximamente)

## Monitoreo

- Logs automáticos en Vercel
- Notificaciones por email para errores críticos
- Sync logs en base de datos para auditoría

## Licencia

Privado - Todos los derechos reservados
