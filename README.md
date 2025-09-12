# DeliveryApp1
Motorcycle delivery app
# 🚚 Route Consolidator MVP

MVP de aplicación móvil que consolida pedidos de múltiples fuentes y optimiza rutas para repartidores en tiempo real.

## 🏗️ Arquitectura

```
route-consolidator/
├── apps/
│   ├── api/           # Backend NestJS + TypeScript
│   ├── mobile/        # App móvil Expo + React Native
│   └── admin/         # Panel web Next.js
├── services/
│   └── optimizer/     # Servicio Python OR-Tools
├── packages/
│   └── shared/        # Tipos y SDK compartido
├── docker-compose.yml # Orquestación local
└── scripts/           # Scripts de desarrollo
```

## 🚀 Quick Start

### Prerrequisitos
- Docker Desktop
- Node.js 20+
- pnpm
- Git

### Instalación

```bash
git clone <repository>
cd route-consolidator

# Instalar dependencias
pnpm install

# Copiar variables de entorno
cp .env.example .env

# Levantar servicios (BD, Redis, API, Optimizer)
docker-compose up -d

# Ejecutar migraciones
pnpm db:migrate

# Cargar datos de ejemplo
pnpm seed

# Iniciar desarrollo (API + Mobile + Admin)
pnpm dev
```

### URLs de desarrollo

- **API Backend**: http://localhost:3001
- **Panel Admin**: http://localhost:3000
- **Mobile App**: Expo DevTools (puerto dinámico)
- **PostgreSQL**: localhost:5432
- **Redis**: localhost:6379
- **Optimizer Service**: http://localhost:8001

## 📱 Aplicación Móvil

### Instalación Expo
```bash
cd apps/mobile
npx expo install
npx expo start
```

### Simulador iOS/Android
- Instala Expo Go en tu dispositivo
- Escanea el QR code
- O usa simuladores: `i` para iOS, `a` para Android

## 🔐 Variables de Entorno

```env
# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/route_consolidator
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-jwt-key
JWT_REFRESH_SECRET=your-refresh-secret

# Google Maps (obtener de Google Cloud Console)
GOOGLE_MAPS_API_KEY=your-google-maps-key

# WhatsApp Cloud API (obtener de Meta Developers)
WHATSAPP_TOKEN=your-whatsapp-token
WHATSAPP_VERIFY_TOKEN=your-verify-token
PHONE_NUMBER_ID=your-phone-number-id

# Optimizer Service
OPTIMIZER_SERVICE_URL=http://localhost:8001
```

## 🗄️ Base de Datos

### Migraciones
```bash
# Crear nueva migración
pnpm db:migration:create nombre_migracion

# Ejecutar migraciones pendientes
pnpm db:migrate

# Revertir última migración
pnpm db:rollback
```

### Seeds
```bash
# Cargar datos de ejemplo
pnpm seed

# Limpiar y recargar
pnpm seed:fresh
```

### Modelo de datos
- `users` - Usuarios del sistema
- `couriers` - Información de repartidores
- `orders` - Pedidos y entregas
- `assignments` - Asignaciones pedido-repartidor
- `routes` - Rutas optimizadas
- `route_stops` - Paradas de cada ruta
- `webhook_events` - Eventos de WhatsApp

## 🧪 Testing

```bash
# Tests unitarios
pnpm test

# Tests e2e
pnpm test:e2e

# Coverage
pnpm test:coverage

# Tests específicos
pnpm test:api        # Solo backend
pnpm test:mobile     # Solo mobile
```

## 🤖 Bot de WhatsApp

### Configuración webhook
1. Ir a Meta Developers > WhatsApp > Configuration
2. Configurar webhook: `https://tu-dominio.com/whatsapp/webhook`
3. Verificar con `WHATSAPP_VERIFY_TOKEN`

### Formatos soportados

**Formulario guiado:**
```
Bot: ¡Hola! Te ayudo a crear un pedido. ¿Cuál es la dirección de recogida?
Usuario: Calle 60 #123, Centro, Mérida
Bot: ¿Y la dirección de entrega?
Usuario: Calle 20 #200, Itzimná
...
```

**Una línea:**
```
Usuario: Pickup: Calle 60 123, Centro, Mérida; Dropoff: Calle 20 200, Itzimná; Ventana: 16:00-17:00; Fee: 80; Nota: frágil
Bot: ✅ Pedido creado. ID: #12345
```

## 🗺️ Optimización de Rutas

### Servicio Optimizer (Python OR-Tools)

```bash
# Desarrollo local
cd services/optimizer
pip install -r requirements.txt
python app.py

# Docker
docker build -t route-optimizer .
docker run -p 8001:8001 route-optimizer
```

### API de optimización
```http
POST /optimize
Content-Type: application/json

{
  "courier_id": "123",
  "current_location": {"lat": 20.9674, "lng": -89.5926},
  "stops": [
    {
      "id": "order_1_pickup",
      "type": "pickup",
      "location": {"lat": 20.9697, "lng": -89.5928},
      "time_window": {"start": "2025-01-15T16:00:00Z", "end": "2025-01-15T17:00:00Z"}
    }
  ]
}
```

## 📊 Monitoreo y Logs

### Health Checks
- **API**: `GET /health`
- **Optimizer**: `GET /health`

### Logs
```bash
# Logs en tiempo real
docker-compose logs -f api
docker-compose logs -f optimizer

# Logs estructurados en JSON
tail -f apps/api/logs/app.log | jq
```

### Métricas básicas
- Pedidos procesados/minuto
- Tiempo de optimización promedio
- Repartidores activos
- Tasa de asignación exitosa

## 🚀 Deployment

### Desarrollo (Render/Railway)
```bash
# Build para producción
pnpm build

# Variables de entorno en plataforma
# Conectar repositorio Git
# Deploy automático en push
```

### Producción (AWS)
```bash
# Terraform (opcional)
cd infrastructure/
terraform init
terraform plan
terraform apply

# O manual: ECS + RDS + ElastiCache
```

## 📱 Flujo de Usuario

### Repartidor
1. **Registro/Login** → Email + contraseña
2. **Feed de pedidos** → Lista filtrable por distancia/tarifa
3. **Aceptar pedido** → Automáticamente se añade a ruta
4. **Ver ruta optimizada** → Secuencia pickup → dropoffs
5. **Navegación** → Deep link a Google Maps/Waze
6. **Actualizar estado** → Recogido → Entregado

### Bot WhatsApp
1. **Recibir mensaje** → Parsear formato
2. **Validar direcciones** → Geocodificar con Google
3. **Crear pedido** → Insertar en BD + emitir evento
4. **Notificar repartidores** → WebSocket + push notifications

### Admin
1. **Dashboard** → Estado general del sistema
2. **Pedidos activos** → Monitorear entregas en curso
3. **Repartidores** → Estado y ubicación en tiempo real
4. **Métricas** → KPIs de rendimiento

## 🔧 Scripts Útiles

```bash
# Desarrollo
pnpm dev              # Inicia todo (API + Mobile + Admin)
pnpm dev:api          # Solo backend
pnpm dev:mobile       # Solo app móvil
pnpm dev:admin        # Solo panel admin

# Base de datos
pnpm db:migrate       # Ejecutar migraciones
pnpm db:rollback      # Revertir migración
pnpm seed             # Cargar datos ejemplo
pnpm db:reset         # Reset completo

# Testing
pnpm test             # Todos los tests
pnpm test:watch       # Tests en modo watch
pnpm lint             # Linting
pnpm format           # Formatear código

# Build
pnpm build            # Build para producción
pnpm build:api        # Solo backend
pnpm build:admin      # Solo admin panel

# Docker
docker-compose up     # Servicios (BD, Redis, etc)
docker-compose down   # Parar servicios
docker-compose logs   # Ver logs
```

## 🐛 Troubleshooting

### Error de conexión a BD
```bash
# Verificar que PostgreSQL está corriendo
docker-compose ps postgres

# Recrear contenedor
docker-compose down postgres
docker-compose up -d postgres
```

### WhatsApp webhook no recibe eventos
1. Verificar que el webhook esté públicamente accesible (ngrok en desarrollo)
2. Confirmar `WHATSAPP_VERIFY_TOKEN` coincide
3. Revisar logs del webhook: `docker-compose logs api`

### Optimización de rutas falla
```bash
# Verificar servicio optimizer
curl http://localhost:8001/health

# Revisar logs Python
docker-compose logs optimizer
```

### App móvil no conecta al backend
1. Verificar IP en `apps/mobile/src/config.ts`
2. En simulador iOS usar `localhost`, en Android usar IP de la máquina
3. Confirmar que el backend está corriendo en puerto 3001

## 📋 Colección Postman

Importar `postman/Route-Consolidator-MVP.json` para probar todos los endpoints:

- Autenticación (registro/login)
- CRUD de pedidos
- Feed de pedidos para repartidores
- Aceptar/declinar asignaciones
- Actualizar estados de entrega
- WebSocket de tiempo real

## 🚧 Roadmap

### ✅ MVP (Completado)
- Autenticación JWT
- Bot WhatsApp básico
- Optimización de rutas
- App móvil funcional
- Panel admin mínimo

### 🔄 Fase 2 (Futuro)
- Integraciones oficiales Uber/Didi
- Sistema de pagos y penalizaciones
- Push notifications nativas
- Analítica avanzada
- Pricing dinámico

## 🤝 Contribuir

1. Fork del repositorio
2. Crear branch: `git checkout -b feature/nueva-funcionalidad`
3. Commit: `git commit -m 'feat: nueva funcionalidad'`
4. Push: `git push origin feature/nueva-funcionalidad`
5. Pull Request

## 📝 Licencia

MIT License - ver `LICENSE` para detalles

## 📞 Soporte

- Issues: GitHub Issues
- Email: dev@route-consolidator.com
- Docs: [docs.route-consolidator.com](docs.route-consolidator.com)

---

**¡MVP listo para desarrollo! 🎉**

Ejecuta `docker-compose up -d && pnpm dev` y tendrás todo funcionando localmente.
