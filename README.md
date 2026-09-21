# 🤖 Teable Docker - AI Spreadsheet Self-Hosted

[![GitHub Stars](https://img.shields.io/github/stars/teableio/teable?style=flat-square&logo=github)](https://github.com/teableio/teable)
[![Docker Pulls](https://img.shields.io/docker/pulls/ghcr.io/teableio/teable?style=flat-square&logo=docker)](https://github.com/teableio/teable/pkgs/container/teable)
[![License](https://img.shields.io/github/license/teableio/teable?style=flat-square)](https://github.com/teableio/teable/blob/main/LICENSE)
[![GitHub Release](https://img.shields.io/github/v/release/teableio/teable?style=flat-square&logo=github)](https://github.com/teableio/teable/releases)

## 📋 Descripción general

**Teable** es un **AI Spreadsheet autohospedado** basado en PostgreSQL que proporciona una alternativa profesional a Airtable/Notion. Combina la potencia de una base de datos relacional real con la flexibilidad de una hoja de cálculo visual, añadiendo capacidades nativas de IA: chat sobre datos, App Builder con agentes, automatizaciones inteligentes y relleno automático de campos.

Esta implementación Docker permite desplegar Teable en modo **Standalone** (tablas + API REST, sin IA) o **Full-featured** (con IA completa: chat, App Builder, automatizaciones). Todo bajo tu control, sin suscripciones cloud, con 21.7k⭐ en GitHub y licencia AGPL-3.0.

## ✨ Características principales

- 🗂️ **Vistas múltiples**: Grid (hoja cálculo), Form (entrada estructurada), Kanban (tableros), Gallery (miniaturas), Calendar (línea temporal)
- 🐘 **PostgreSQL real**: Backend escalable, confiable, consultas SQL nativas, sin vendor lock-in
- 🤖 **AI Chat nativo**: Pregunta sobre tus datos en lenguaje natural, análisis y operaciones directas
- 🏗️ **App Builder**: Describe la app que necesitas, agente IA la construye en sandbox aislado, deploy one-click
- ⚡ **AI Automations**: Reacciona a cambios de registros, schedules, webhooks con pasos de razonamiento IA
- ✨ **AI Field Filling**: Generación y enriquecimiento masivo de valores de campos automático e inteligente
- 👥 **Colaboración real-time**: Múltiples usuarios simultáneos, comentarios, historial de registros, undo/redo
- 🧮 **Fórmulas avanzadas**: Field functions, validación, computaciones complejas sobre datos
- 🔌 **Plugins + API REST**: Extensiones, API completa para integraciones custom, webhooks
- 📊 **Millones de filas**: Rendimiento sin lag, PostgreSQL escala, Grid rápido incluso en huge datasets
- 🔐 **Permisos granulares**: Multi-user con roles, control de acceso por tabla/vista
- 🚀 **Tres modos deployment**: Cloud (teable.ai), Full-featured self-host (IA), Standalone (solo API)

## 📋 Requisitos del sistema

- **Docker & Docker Compose v2+**
- **RAM**: 4-8 GB mínimo (full-featured con IA); 2 GB suficiente para Standalone
- **Disco**: 20-100+ GB según volumen de datos y modelos IA si se incluyen
- **Puerto TCP**: 3000 (Web UI) o reverse proxy HTTPS
- **PostgreSQL 14+** (incluido en contenedor de deployment Teable)
- **Node.js 18+** (backend Next.js, incluido en imagen)
- **IA features (opcional)**: API key LLM (OpenAI, etc.) o modelos locales
- **App Builder**: Requiere Docker separado (agent sandbox) + compute adicional
- **Opcional**: S3 o storage compatible para attachments

## 🐳 Instalación

### Modo Standalone: Tablas, vistas, API REST (sin IA)

#### Paso 1: Crear `docker-compose.yml`

```yaml
version: '3.8'

services:
  teable:
    image: ghcr.io/teableio/teable:latest
    container_name: teable
    restart: unless-stopped
    environment:
      - DATABASE_URL=postgresql://teable:teable123@postgres:5432/teable
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      - NEXTAUTH_URL=http://localhost:3000
      - TZ=Europe/Madrid
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy

  postgres:
    image: postgres:16-alpine
    container_name: teable-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=teable
      - POSTGRES_USER=teable
      - POSTGRES_PASSWORD=teable123
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U teable"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

#### Paso 2: Generar `NEXTAUTH_SECRET`

```bash
# Generar secret aleatorio seguro
openssl rand -hex 32
```

Copia el output y crea un archivo `.env` en el mismo directorio:

```env
NEXTAUTH_SECRET=tu_secret_generado_aqui
```

#### Paso 3: Iniciar Teable

```bash
# Guardar compose como docker-compose.yml
docker compose up -d

# Espera ~15 segundos para migraciones
docker compose logs -f teable
# Debería mostrar "Ready" cuando esté listo
```

### Acceder a Teable

📊 **Teable Web UI**: http://localhost:3000

Desde otros dispositivos en la red local:
```bash
# Obtener IP del servidor
hostname -I
# Acceder via: http://192.168.1.100:3000
```

## ⚙️ Configuración

1. **Variables de entorno críticas**: `DATABASE_URL`, `NEXTAUTH_SECRET`, `NEXTAUTH_URL`, `TZ`
2. **Base de datos**: PostgreSQL 16 Alpine con healthcheck para arranque ordenado
3. **Persistencia**: Volumen `postgres_data` para datos persistentes
4. **Zona horaria**: Ajusta `TZ` según tu ubicación (ej: `America/Mexico_City`, `UTC`)
5. **Reverse Proxy (producción)**: Configura nginx/Traefik/Caddy con HTTPS y actualiza `NEXTAUTH_URL`
6. **Attachments S3 (opcional)**: Añade `S3_ENDPOINT`, `S3_BUCKET`, `S3_ACCESS_KEY`, `S3_SECRET_KEY`
7. **IA Features (Full-featured)**: Requiere deployment separado `teableio/teable-deployment` o license key

## 🚀 Primeros pasos

1. **Crear workspace + base de datos**
   - Abre http://localhost:3000
   - Sign up → crear usuario (email + contraseña)
   - Setup workspace → Create Base → nombra base (ej: "CRM", "Projects")
   - Se crea workspace automáticamente, eres owner

2. **Crear tablas**
   - Base → Add Table → nombra tabla (ej: "Contacts", "Tasks")
   - Auto-crea columna "Name" (String)
   - Agrega más columnas: Email, Phone, Status, etc.
   - Elige field type: Text, Number, Checkbox, Date, Attachment, etc.

3. **Agregar filas (records)**
   - View Grid → Click "+" para agregar fila
   - Ingresa datos → guardado automático real-time

4. **Explorar vistas**
   - Tabla → "+" Add View
   - Elige: Form (input structured), Kanban (cards), Gallery (thumbnails), Calendar (timeline)

5. **Fórmulas + validación**
   - Tabla → Add Column → elige "Formula"
   - Escribe fórmula (ej: `{Name} + " " + {Email}`)
   - Validation: marca requerida, rango numérico, etc.

6. **Colaboración**
   - Base Settings → Share → Add Member
   - Email usuario, selecciona role (Admin, Editor, Viewer)
   - Multi-user acceso simultáneo, comments on records

7. **API REST**
   - Settings → API Tokens → Create token
   - Usa para webhooks, integraciones custom
   - Docs en help.teable.ai/api-doc

## 💡 Casos de uso

- 🔄 **Reemplazo Airtable**: Base datos flexible, vistas múltiples, API REST, colaboración, self-hosted
- 📇 **CRM interno**: Contactos, deals, pipeline, vista Kanban, comentarios, historial
- 📋 **Gestión proyectos**: Tasks, timelines, asignación equipo, vista Calendar, comentarios colaborativos
- 📦 **Inventory management**: SKU tracking, stock levels, reorder, fórmulas automáticas, fotos adjuntas
- 🤖 **AI-powered workflows (Full-featured)**: AI chat analiza datos, AI automations reaccionan eventos, App Builder genera tools
- 🛠️ **No-code/low-code platform**: Usuarios no-tech construyen apps con agentes IA

## 🔒 Acceso remoto seguro

Para exponer Teable de forma segura a internet:

1. **Reverse Proxy recomendado**: Nginx Proxy Manager, Traefik o Caddy con Let's Encrypt
2. **Configurar `NEXTAUTH_URL`** con tu dominio HTTPS (ej: `https://teable.tudominio.com`)
3. **Autenticación**: Teable usa NextAuth (email/password, OAuth providers configurables)
4. **Firewall**: Limita acceso directo a puerto 3000 solo a red local
5. **VPN alternativa**: WireGuard/Tailscale para acceso privado sin exposición pública

## 🛠️ Gestión y mantenimiento

```bash
# Ver estado contenedores
docker compose ps

# Ver logs en tiempo real
docker compose logs -f teable
docker compose logs -f postgres

# Detener Teable
docker compose down

# Actualizar versión
docker compose pull
docker compose up -d

# Backup datos PostgreSQL
docker compose exec postgres pg_dump -U teable teable > backup.sql

# Restore datos
docker compose exec -T postgres psql -U teable teable < backup.sql

# PostgreSQL tuning (producción)
# Ver help.teable.ai/en/deploy/docker
# Ajustar: RAM, shared_buffers, work_mem, effective_cache_size, etc.

# Monitorear consumo recursos
docker stats teable postgres
# Típicamente:
# teable: 300-500MB RAM
# postgres: 200-800MB RAM (según tuning)
```

### Full-featured mode (con IA)

Para desbloquear **AI Chat, App Builder, AI Automations**:

- **Opción A**: `teableio/teable-deployment` - Repo completo para full-featured self-host (requiere compute, orchestration)
- **Opción B**: Teable Cloud (teable.ai) - Managed hosted, pago por uso, sin self-host
- **Opción C**: License key - Activa IA features en imagen oficial (ver pricing teable.ai)

> 💡 **Migración suave**: Empieza con Standalone (esta guía). Si necesitas IA, migras a full-featured manteniendo tus datos.

## 📝 Licencia

- **Community Edition**: AGPL-3.0 (open source)
- **AI Features / Enterprise**: Licencia comercial (ver pricing en teable.ai)
- **Packages internos**: MIT

---

> 📖 **Guía completa y detalles**: [Cómo instalar Teable en Docker - AI Spreadsheet alternativa Airtable/Notion](https://genbyte.blogspot.com/2026/09/como-instalar-teable-en-docker-ai.html)