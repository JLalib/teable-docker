# 🤖 Teable Docker - AI Spreadsheet Self-Hosted

[![GitHub Stars](https://img.shields.io/github/stars/teableio/teable?style=for-the-badge&logo=github)](https://github.com/teableio/teable)
[![Docker Pulls](https://img.shields.io/docker/pulls/ghcr.io/teableio/teable?style=for-the-badge&logo=docker)](https://github.com/teableio/teable/pkgs/container/teable)
[![License](https://img.shields.io/github/license/teableio/teable?style=for-the-badge)](https://github.com/teableio/teable/blob/main/LICENSE)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue?style=for-the-badge&logo=postgresql)](https://www.postgresql.org/)
[![AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-orange?style=for-the-badge)](https://www.gnu.org/licenses/agpl-3.0.html)

---

## 📋 Descripción general

**Teable** es un **AI Spreadsheet autohospedado** basado en PostgreSQL que proporciona una alternativa profesional a Airtable/Notion. Combina la potencia de una base de datos relacional real con la flexibilidad de una hoja de cálculo moderna, añadiendo capacidades nativas de IA: chat sobre datos, App Builder con agentes, automatizaciones inteligentes y relleno automático de campos.

Esta implementación Docker permite desplegar Teable en modo **Standalone** (tablas + API REST, sin IA) o **Full-featured** (con IA completa: chat, App Builder, automatizaciones). Todo bajo tu control, sin suscripciones cloud, con 21.7k+ estrellas en GitHub y licencia AGPL-3.0 open source.

---

## ✨ Características principales

- 🗂️ **Vistas múltiples**: Grid (hoja cálculo), Form (input estructurado), Kanban (tableros), Gallery (miniaturas), Calendar (línea tiempo)
- 🐘 **PostgreSQL real**: Backend SQL escalable, confiable, queries nativas, sin vendor lock-in
- 🤖 **AI Chat nativo**: Pregunta sobre tus datos en lenguaje natural, análisis y operaciones directas
- 🏗️ **App Builder**: Describe la app que necesitas, agente IA la construye en sandbox aislado, deploy one-click
- ⚡ **AI Automations**: Reacciona a cambios de registros, schedules, webhooks con pasos de razonamiento IA
- ✨ **AI Field Filling**: Generación y enriquecimiento masivo de valores de campos automático e inteligente
- 👥 **Colaboración real-time**: Múltiples usuarios simultáneos, comentarios, historial, undo/redo
- 🧮 **Fórmulas avanzadas**: Field functions, validación, computaciones complejas
- 🔌 **Plugins + API REST**: Extensiones, API completa para integraciones custom, webhooks
- 📊 **Millones de filas**: Rendimiento sin lag, PostgreSQL escala horizontalmente
- 🔐 **Permisos granulares**: Multi-usuario con roles, control de acceso por tabla/vista
- 🚀 **Tres modos despliegue**: Cloud (teable.ai), Full-featured self-host (IA), Standalone (solo datos + API)

---

## 📋 Requisitos del sistema

- ✅ **Docker & Docker Compose v2+**
- 💾 **RAM**: 4-8 GB mínimo (full-featured con IA); 2 GB suficiente para Standalone
- 💿 **Disco**: 20-100+ GB según volumen de datos y modelos IA si se incluyen
- 🌐 **Puerto TCP**: 3000 (Web UI) o reverse proxy con HTTPS
- 🐘 **PostgreSQL 14+** (incluido en contenedor de despliegue Teable)
- 🟢 **Node.js 18+** (backend Next.js, incluido en imagen)
- 🔑 **IA features (opcional)**: API key LLM (OpenAI, etc.) o modelos locales
- 🏗️ **App Builder**: Requiere Docker separado (agent sandbox) + compute adicional
- ☁️ **Opcional**: S3 o storage compatible para attachments

---

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

# Copiar el output y crear archivo .env
echo "NEXTAUTH_SECRET=tu_secret_generado_aqui" > .env
```

#### Paso 3: Iniciar Teable

```bash
# Guardar compose como docker-compose.yml
docker compose up -d

# Esperar ~15 segundos para migraciones
docker compose logs -f teable

# Debería mostrar "Ready" cuando esté listo
```

#### Acceder a Teable

📊 **Teable Web UI**: http://localhost:3000

Desde otros dispositivos en la red:
```bash
# Obtener IP del servidor
hostname -I
# Acceder via: http://192.168.1.100:3000
```

---

## ⚙️ Configuración

1. **Variables de entorno críticas**: `DATABASE_URL`, `NEXTAUTH_SECRET`, `NEXTAUTH_URL`, `TZ`
2. **Seguridad**: Cambiar contraseña por defecto de PostgreSQL (`teable123`) en producción
3. **Reverse Proxy**: Configurar Nginx/Traefik/Caddy para HTTPS y dominio personalizado
4. **Timezone**: Ajustar `TZ` a tu zona horaria (ej: `America/Mexico_City`, `UTC`)
5. **Persistencia**: El volume `postgres_data` persiste la base de datos entre reinicios
6. **Actualizaciones**: Usar `docker compose pull && docker compose up -d` para nuevas versiones
7. **Backup**: Ver sección [Gestión y mantenimiento](#-gestión-y-mantenimiento)

---

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
   - Ingresa datos → Guardado automático real-time

4. **Explorar vistas**
   - Tabla → "+" Add View
   - Elige: Form (input estructurado), Kanban (tarjetas), Gallery (miniaturas), Calendar (timeline)

5. **Fórmulas + validación**
   - Tabla → Add Column → elige "Formula"
   - Escribe fórmula (ej: `{Name} + " " + {Email}`)
   - O Validation: marca requerida, rango numérico, etc.

6. **Colaboración**
   - Base Settings → Share → Add Member
   - Email usuario, selecciona role (Admin, Editor, Viewer)
   - Multi-usuario acceso simultáneo, comentarios en registros

7. **API REST (para integraciones)**
   - Settings → API Tokens → Create token
   - Usar para webhooks, integraciones custom
   - Docs en: https://help.teable.ai/api-doc

---

## 💡 Casos de uso

- 🔄 **Reemplazo Airtable**: Base datos flexible, vistas múltiples, API REST, colaboración, self-hosted
- 📇 **CRM interno**: Contactos, deals, pipeline, vista Kanban, comentarios, historial
- 📋 **Gestión proyectos**: Tasks, timelines, asignación equipo, vista Calendar, comentarios colaborativos
- 📦 **Inventory management**: SKU tracking, stock levels, reorder, fórmulas automáticas, fotos adjuntas
- 🤖 **Workflows con IA (Full-featured)**: AI chat analiza datos, AI automations reaccionan eventos, App Builder genera tools
- 🛠️ **Plataforma no-code/low-code**: Usuarios no-tech construyen apps con agentes IA

---

## 🔒 Acceso remoto seguro

Para exponer Teable de forma segura a internet:

1. **Reverse Proxy recomendado**: Nginx Proxy Manager, Traefik o Caddy con Let's Encrypt
2. **Configurar `NEXTAUTH_URL`** a tu dominio HTTPS (ej: `https://teable.tudominio.com`)
3. **Autenticación**: Teable usa NextAuth.js, compatible con OAuth (GitHub, Google, etc.)
4. **Firewall**: Restringir puerto 3000 solo al reverse proxy
5. **VPN alternativa**: WireGuard/Tailscale para acceso privado sin exposición pública

---

## 🛠️ Gestión y mantenimiento

### Ver estado
```bash
docker compose ps
```

### Ver logs
```bash
docker compose logs -f teable
docker compose logs -f postgres
```

### Detener Teable
```bash
docker compose down
```

### Actualizar versión
```bash
docker compose pull
docker compose up -d
```

### Backup datos PostgreSQL
```bash
docker compose exec postgres pg_dump -U teable teable > backup.sql
```

### Restore datos
```bash
docker compose exec -T postgres psql -U teable teable < backup.sql
```

### PostgreSQL tuning (producción)
```bash
# Ver guía oficial: https://help.teable.ai/en/deploy/docker
# Parámetros clave: shared_buffers, work_mem, effective_cache_size, max_connections
```

### Monitorear consumo
```bash
docker stats teable postgres

# Típicamente:
# teable: 300-500MB RAM
# postgres: 200-800MB RAM (según tuning y carga)
```

---

## 🤖 Modo Full-featured (con IA)

Para desbloquear **AI Chat, App Builder, AI Automations**:

| Opción | Descripción |
|--------|-------------|
| **teableio/teable-deployment** | Repo completo para full-featured self-host (requiere compute, orquestación) |
| **Teable Cloud (teable.ai)** | Hosted managed, pago por uso, sin self-host |
| **License key** | Activa IA features en imagen oficial (ver pricing teable.ai) |

> 💡 **Recomendación**: Empieza con **Standalone** (esta guía). Si necesitas IA, migración suave a full-featured (datos se mantienen).

---

## 🏗️ Stack técnico

| Componente | Tecnología |
|------------|------------|
| **Backend** | NestJS (Node.js, TypeScript) |
| **Frontend** | Next.js (React, TypeScript, Vite) |
| **Database** | PostgreSQL 14+ (real SQL backend) |
| **ORM** | TypeORM (query builder) |
| **API** | REST + GraphQL capable (webhooks, integraciones) |
| **Real-time** | WebSockets para colaboración |
| **IA (Full)** | Agent sandboxing, LLM integrations |
| **App Builder** | Contenedores aislados por app deployment |

---

## 📝 Licencia

- **Community Edition**: AGPL-3.0 (open source)
- **AI Features / Commercial**: Licencia comercial (ver [teable.ai/pricing](https://teable.ai/pricing))
- **Packages internos**: MIT (componentes específicos)

---

## 📚 Referencias oficiales

- 🌐 [Teable Official Website](https://teable.ai) - Cloud hosted
- 🐙 [Teable GitHub](https://github.com/teableio/teable) - Community Edition AGPL-3.0
- 📖 [Deployment Guide](https://help.teable.ai/en/deploy/docker) - Cloud vs self-host modes
- 🐳 [Docker Deployment Guide](https://help.teable.ai/en/deploy/docker) - Standalone + full-featured
- 🚀 [teable-deployment](https://github.com/teableio/teable-deployment) - Full-featured self-host
- 📄 [API Documentation](https://help.teable.ai/api-doc) - REST endpoints
- 💬 [Community Forum](https://github.com/teableio/teable/discussions) - Support, ideas
- 💰 [Pricing](https://teable.ai/pricing) - Cloud + license keys

---

> 📖 **Guía completa original**: [Cómo instalar Teable en Docker - AI Spreadsheet alternativa Airtable/Notion](https://genbyte.blogspot.com/2026/09/como-instalar-teable-en-docker-ai.html)