# Guía de Deployment - ML Pricing Platform

## 1. Configurar Supabase

### Crear Proyecto

1. Ve a [https://supabase.com](https://supabase.com)
2. Crea un nuevo proyecto
3. Nombra el proyecto: `ml-pricing-platform`
4. Elige la región más cercana (por ejemplo, `South America (São Paulo)`)
5. Genera un password seguro para la base de datos
6. Espera a que el proyecto termine de inicializarse

### Obtener Connection String

1. En tu proyecto de Supabase, ve a **Settings** → **Database**
2. En la sección **Connection Pooling**, haz clic en **Connection string**
3. Modo: **Transaction**
4. Copia la connection string que se ve así:
   ```
   postgresql://postgres.xxx:[YOUR-PASSWORD]@aws-0-sa-east-1.pooler.supabase.com:6543/postgres
   ```
5. Reemplaza `[YOUR-PASSWORD]` con el password que generaste
6. Agrega los parámetros de conexión al final:
   ```
   ?pgbouncer=true&connection_limit=1&pool_timeout=20
   ```

La URL final debe verse así:
```
postgresql://postgres.xxx:tu_password@aws-0-sa-east-1.pooler.supabase.com:6543/postgres?pgbouncer=true&connection_limit=1&pool_timeout=20
```

---

## 2. Deployment en Vercel

### Conectar Repositorio

1. Ve a [https://vercel.com](https://vercel.com)
2. Click en **Add New** → **Project**
3. Importa el repositorio: `nacholeoo/Dardo`
4. Vercel detectará automáticamente que es un proyecto Next.js
5. **NO HAGAS DEPLOY TODAVÍA** - Primero vamos a configurar las variables de entorno

### Configurar Variables de Entorno

En la sección **Environment Variables**, agrega las siguientes variables:

#### Database
```
DATABASE_URL = <tu_connection_string_de_supabase>
```

#### Authentication (generar secrets con Node.js)
```bash
# En tu terminal local, ejecuta:
node -e "console.log('JWT_SECRET=' + require('crypto').randomBytes(32).toString('hex'))"
node -e "console.log('CRON_SECRET=' + require('crypto').randomBytes(32).toString('hex'))"
node -e "console.log('ENCRYPTION_KEY=' + require('crypto').randomBytes(32).toString('hex'))"

# Para generar el password hash del admin:
node -e "console.log('ADMIN_PASSWORD_HASH=' + require('bcryptjs').hashSync('TU_PASSWORD_ADMIN', 10))"
```

Copia los valores generados y agrégalos a Vercel:
```
JWT_SECRET = <valor_generado>
CRON_SECRET = <valor_generado>
ENCRYPTION_KEY = <valor_generado>
ADMIN_PASSWORD_HASH = <hash_generado>
```

#### App URL (se actualizará después del primer deploy)
```
NEXT_PUBLIC_APP_URL = https://tu-proyecto.vercel.app
```

#### Opcional (configurar después):
```
AIR_INTRA_API_TOKEN = <token_de_air_intra>
AIR_INTRA_API_URL = https://api.air-intra.com/v2
ML_CLIENT_ID = <tu_client_id>
ML_CLIENT_SECRET = <tu_client_secret>
RESEND_API_KEY = <tu_resend_api_key>
NOTIFICATION_EMAIL = <tu_email>
```

### Hacer el Deploy

1. Click en **Deploy**
2. Espera 2-3 minutos mientras Vercel construye y deploya
3. Una vez completado, verás la URL de tu aplicación

### Actualizar NEXT_PUBLIC_APP_URL

1. Copia la URL de producción (ej: `https://dardo.vercel.app`)
2. Ve a **Settings** → **Environment Variables**
3. Edita `NEXT_PUBLIC_APP_URL` con la URL de producción
4. Redeploy el proyecto:
   - Ve a **Deployments**
   - Click en el deployment más reciente
   - Click en **⋯** → **Redeploy**

---

## 3. Ejecutar Migraciones de Base de Datos

Después del primer deployment exitoso, necesitas crear las tablas en Supabase:

### Opción A: Desde tu computadora local

1. Actualiza el archivo `.env` local con la DATABASE_URL de Supabase
2. Ejecuta:
   ```bash
   npm run db:push
   ```
3. Esto sincronizará el esquema Prisma con la base de datos

### Opción B: Desde Vercel CLI (recomendado para producción)

1. Instala Vercel CLI:
   ```bash
   npm i -g vercel
   ```

2. Login a Vercel:
   ```bash
   vercel login
   ```

3. Link el proyecto:
   ```bash
   vercel link
   ```

4. Pull las variables de entorno:
   ```bash
   vercel env pull .env.local
   ```

5. Ejecuta las migraciones:
   ```bash
   npm run db:push
   ```

---

## 4. Seed de Datos Iniciales

### Parámetros de Pricing (Valores por defecto del workflow n8n)

Ejecuta esto en **Supabase SQL Editor**:

```sql
-- Pricing parameters
INSERT INTO pricing_parameters (id, name, value, valid_from) VALUES
  (gen_random_uuid(), 'margen_ganancia', 0.2000, NOW()),
  (gen_random_uuid(), 'comision_meli', 0.1500, NOW()),
  (gen_random_uuid(), 'alicuota_iibb', 0.0500, NOW()),
  (gen_random_uuid(), 'costo_envio_gratis', 0.0500, NOW());
```

### (Opcional) Category Mappings de Ejemplo

```sql
-- Example category mappings - actualizar con categorías reales
INSERT INTO category_mappings (id, air_intra_key, ml_category_id) VALUES
  (gen_random_uuid(), 'Electronica:Smartphones', 'MLM1055'),
  (gen_random_uuid(), 'Computacion:Notebooks', 'MLM1648');
```

---

## 5. Verificación Post-Deployment

### Checklist

- [ ] El sitio carga correctamente en la URL de Vercel
- [ ] La conexión a base de datos funciona (no hay errores en logs)
- [ ] Las tablas fueron creadas en Supabase (verifica en SQL Editor)
- [ ] Los parámetros de pricing fueron insertados
- [ ] Los cron jobs están configurados en Vercel (Settings → Cron Jobs)

### Ver Logs

- **Vercel Dashboard** → **Deployments** → Click en deployment → **Function Logs**
- **Supabase Dashboard** → **Database** → **Logs**

### Probar la Aplicación

1. Visita tu URL de Vercel
2. Debería aparecer la página de inicio: "ML Pricing Platform"
3. Los siguientes pasos serán implementar la autenticación y el dashboard

---

## 6. Próximos Pasos

Una vez deployado exitosamente, continuaremos con:

1. ✅ Implementar autenticación (login page + JWT)
2. Crear el dashboard principal
3. Implementar sincronización con Air Intra
4. Configurar OAuth de MercadoLibre
5. Implementar funcionalidades de pricing

---

## Troubleshooting

### Error: "Could not connect to database"
- Verifica que DATABASE_URL esté correcta en las variables de entorno de Vercel
- Asegúrate de que incluye `?pgbouncer=true&connection_limit=1&pool_timeout=20`
- Verifica que el password no tenga caracteres especiales sin escapar

### Error: "Table does not exist"
- Ejecuta `npm run db:push` para crear las tablas
- Verifica que las migraciones se ejecutaron correctamente

### Los Cron Jobs no se ejecutan
- Los cron jobs de Vercel solo funcionan en planes Pro o superior
- Para testing, puedes usar el endpoint manual: `/api/cron/sync-hourly` con el header `Authorization: Bearer <CRON_SECRET>`

### Build fails en Vercel
- Verifica que todas las dependencias estén en package.json
- Revisa los logs de build en Vercel
- Asegúrate de que no hay errores de TypeScript

---

**¿Listo para deployar?** Sigue los pasos en orden y no dudes en preguntar si encuentras algún error.
