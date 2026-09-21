# 🤖 Teable Docker - AI Spreadsheet Self-Hosted

[![GitHub Stars](https://img.shields.io/github/stars/teableio/teable?style=social)](https://github.com/teableio/teable)
[![Docker Pulls](https://img.shields.io/docker/pulls/ghcr.io/teableio/teable)](https://github.com/teableio/teable/pkgs/container/teable)
[![License](https://img.shields.io/github/license/teableio/teable)](https://github.com/teableio/teable/blob/main/LICENSE)
[![Docker Image](https://img.shields.io/badge/docker-ghcr.io%2Fteableio%2Fteable-blue)](https://github.com/teableio/teable/pkgs/container/teable)

## 📋 Descripción general

**Teable** es un **AI Spreadsheet autohospedado** basado en PostgreSQL que proporciona una alternativa profesional a Airtable/Notion. Combina la potencia de una base de datos relacional real con la flexibilidad de una hoja de cálculo visual, añadiendo capacidades nativas de IA: chat sobre datos, App Builder con agentes, automatizaciones inteligentes y relleno automático de campos.

Este repositorio contiene la configuración **Docker Compose** para desplegar Teable en modo **Standalone** (tablas + API REST, sin funciones IA) de forma sencilla y lista para producción. Ideal para equipos que buscan soberanía de datos, cero suscripciones cloud y rendimiento a escala de millones de filas.

> 📖 Basado en la guía: [Cómo instalar Teable en Docker - AI Spreadsheet alternativa Airtable/Notion](https://genbyte.blogspot.com/2026/09/como-instalar-teable-en-docker-ai.html)

## ✨ Características principales

- 🗂️ **Vistas múltiples**: Grid (hoja cálculo), Form (entrada estructurada), Kanban (tableros), Gallery (miniaturas), Calendar (línea temporal)
- 🐘 **PostgreSQL real**: Backend SQL escalable, confiable, sin vendor lock-in — consultas SQL nativas
- 🤖 **IA nativa (Full-featured)**: Chat en lenguaje natural, App Builder (agentes generan apps), AI Automations, AI Field Filling
- 👥 **Colaboración real-time**: Múltiples usuarios simultáneos, comentarios, historial de registros, undo/redo
- 🧮 **Fórmulas avanzadas**: Field functions, validación de campos, computaciones complejas
- 🔌 **Plugins + API REST**: Extensiones, webhooks, API completa para integraciones custom
- ⚡ **Millones de filas sin lag**: PostgreSQL optimizado, carga casi instantánea en datasets grandes
- 🔐 **Permisos granulares**: Multi-usuario con roles (Admin, Editor, Viewer), control por tabla/vista
- 📦 **Tres modos de despliegue**: Cloud (teable.ai), Full-featured self-host (IA completa), Standalone (API + datos)
- 🐳 **Docker multi-arquitectura**: Despliegue sencillo, actualizaciones con `docker compose pull`
- 📄 **Licencia AGPL-3.0**: Open source (paquetes MIT), enterprise-ready

## 📋 Requisitos del sistema

- ✅ **Docker** & **Docker Compose v2+**
- 💾 **RAM**: 4–8 GB mínimo (Full-featured con IA); 2 GB suficiente para Standalone
- 💿 **Disco**: 20–100+ GB según volumen de datos y modelos IA si se incluyen
- 🌐 **Puerto TCP**: 3000 (Web UI) o reverse proxy con HTTPS
- 🐘 **PostgreSQL 14+** (incluido en el contenedor de despliegue)
- 🟢 **Node.js 18+** (backend Next.js, dentro del contenedor)
- 🔑 **IA (opcional)**: API Key de LLM (OpenAI, etc.) o modelos locales
- 🏗️ **App Builder**: Requiere Docker separado (sandbox de agentes) + compute
- ☁️ **Storage S3 compatible (opcional)**: Para adjuntos en producción

## 🐳 Instalación

### Modo Standalone: tablas, vistas, API REST (sin IA)

#### Paso 1: `docker-compose.yml`

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
# Copiar el output y crear archivo .env o exportar variable
echo "NEXTAUTH_SECRET=tu_secret_aqui" > .env
```

#### Paso 3: Iniciar Teable

```bash
# Guardar el compose como docker-compose.yml
docker compose up -d

# Esperar ~15 segundos para migraciones
docker compose logs -f teable
# Debería mostrar "Ready" cuando esté listo
```

### Acceder a Teable

📊 **Teable Web UI**: http://localhost:3000

#### Setup inicial (primer acceso)

1. Abre http://localhost:3000
2. **Sign up** → crear usuario (email + contraseña)
3. **Setup workspace**
4. Crear primera base de datos → agregar tablas
5. ¡Empezar a construir!

> 💡 **Desde otros dispositivos**: Usa la IP de tu servidor: `http://192.168.1.100:3000`  
> Para obtener tu IP: `hostname -I`

## ⚙️ Configuración

1. **Variables de entorno críticas**  
   - `DATABASE_URL`: Conexión PostgreSQL (usuario, password, host, puerto, DB)  
   - `NEXTAUTH_SECRET`: Clave secreta para autenticación (generar con `openssl rand -hex 32`)  
   - `NEXTAUTH_URL`: URL pública de acceso (ej. `https://teable.tudominio.com`)  
   - `TZ`: Zona horaria (ej. `Europe/Madrid`)

2. **Credenciales PostgreSQL**  
   - Usuario/DB/Password definidos en `postgres` service y referenciados en `DATABASE_URL`  
   - Cambiar `teable123` por password seguro en producción

3. **Persistencia de datos**  
   - Volumen `postgres_data` mapea `/var/lib/postgresql/data`  
   - Backups automáticos vía `pg_dump` (ver sección mantenimiento)

4. **Reverse Proxy (recomendado producción)**  
   - Nginx/Traefik/Caddy terminando TLS  
   - Actualizar `NEXTAUTH_URL` a `https://tu-dominio.com`  
   - Configurar headers `X-Forwarded-Proto`, `X-Forwarded-For`

5. **Modo Full-featured (IA)**  
   - Requiere despliegue `teableio/teable-deployment` (repo separado)  
   - O licencia key en teable.ai para activar IA en imagen oficial  
   - Migración suave desde Standalone: los datos permanecen

## 🚀 Primeros pasos

1. **Crear workspace + base de datos**  
   Dashboard → **Create Base** → nombra la base (ej: "CRM", "Projects") → se crea workspace automáticamente → eres owner

2. **Crear tablas**  
   Base → **Add Table** → nombra tabla (ej: "Contacts", "Tasks") → auto-crea columna "Name" (String) → agrega más columnas: Email, Phone, Status, etc. → elige field type: Text, Number, Checkbox, Date, Attachment, etc.

3. **Agregar filas (records)**  
   Vista Grid → Click **"+"** para agregar fila → ingresa datos → guardado automático real-time

4. **Explorar vistas**  
   Tabla → **"+" Add View** → elige **Form** (input estructurado), **Kanban** (tarjetas), **Gallery** (miniaturas), **Calendar** (línea temporal)

5. **Fórmulas + validación**  
   Tabla → **Add Column** → elige **"Formula"** → escribe fórmula (ej: `{Name} + " " + {Email}`) → o **Validation**: marca requerida, rango numérico, etc.

6. **Colaboración**  
   Base Settings → **Share** → **Add Member** → email usuario, selecciona rol (Admin, Editor, Viewer) → multi-usuario simultáneo, comentarios en registros

7. **API REST (para integraciones)**  
   Settings → **API Tokens** → **Create token** → usa para webhooks, integraciones custom → Docs en `help.teable.ai/api-doc`

## 💡 Casos de uso

- 🔄 **Reemplazo Airtable**: Base de datos flexible, vistas múltiples, API REST, colaboración, self-hosted, sin suscripción
- 📇 **CRM interno**: Contactos, deals, pipeline, vista Kanban, comentarios, historial de cambios
- 📋 **Gestión de proyectos**: Tasks, timelines, asignación de equipo, vista Calendar, comentarios colaborativos
- 📦 **Inventario**: SKU tracking, niveles de stock, reorder points, fórmulas automáticas, adjuntos de fotos
- 🤖 **Workflows con IA (Full-featured)**: AI Chat analiza datos, AI Automations reaccionan a eventos, App Builder genera tools
- 🛠️ **Plataforma no-code/low-code**: Usuarios no-técnicos construyen apps con agentes IA

## 🔒 Acceso remoto seguro

> **Recomendado para producción**: No exponer puerto 3000 directamente a Internet.

1. **Reverse Proxy con TLS** (Nginx, Traefik, Caddy)  
   - Terminación HTTPS en proxy  
   - `NEXTAUTH_URL=https://teable.tudominio.com`  
   - Headers: `X-Forwarded-Proto: https`, `X-Forwarded-For`

2. **Autenticación adicional**  
   - Authelia / Authentik / OAuth2 Proxy delante del proxy  
   - 2FA obligatorio para acceso externo

3. **VPN / Tailscale / WireGuard**  
   - Acceso solo via red privada virtual  
   - Cero exposición pública

4. **Firewall**  
   - Restringir puerto 3000 solo a IP del proxy / red VPN

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
docker compose exec postgres pg_dump -U teable teable > backup_$(date +%F).sql
```

### Restore datos
```bash
docker compose exec -T postgres psql -U teable teable < backup_2026-01-15.sql
```

### PostgreSQL tuning (producción)
```bash
# Ver guía oficial: help.teable.ai/en/deploy/docker
# Parámetros clave: shared_buffers, work_mem, effective_cache_size, maintenance_work_mem
# Ajustar según RAM disponible (ej. shared_buffers = 25% RAM)
```

### Monitorear consumo
```bash
docker stats teable postgres
# Típicamente:
# teable: 300-500 MB RAM
# postgres: 200-800 MB RAM (según tuning y dataset)
```

### Full-featured mode (con IA)
Para desbloquear **AI Chat, App Builder, AI Automations**:

| Opción | Descripción |
|--------|-------------|
| `teableio/teable-deployment` | Repo completo para full-featured self-host (requiere compute, orquestación) |
| **Teable Cloud** (teable.ai) | Hosted managed, pago por uso, sin self-host |
| **License Key** | Activa IA en imagen oficial (ver pricing teable.ai) |

> 💡 **Empieza con Standalone** (esta guía). Si necesitas IA, migración suave a full-featured (datos se mantienen).

## 📝 Licencia

- **Community Edition**: AGPL-3.0 (open source)
- **Paquetes internos**: MIT
- **IA Features / Enterprise**: Licencia comercial (ver [teable.ai/pricing](https://teable.ai/pricing))

---

> 📌 **Referencia completa**: [Cómo instalar Teable en Docker - AI Spreadsheet alternativa Airtable/Notion](https://genbyte.blogspot.com/2026/09/como-instalar-teable-en-docker-ai.html)  
> 🐙 **Repo oficial**: [github.com/teableio/teable](https://github.com/teableio/teable)  
> 📚 **Docs despliegue**: [help.teable.ai/en/deploy/docker](https://help.teable.ai/en/deploy/docker)  
> 🤝 **Comunidad**: [Forum](https://github.com/teableio/teable/discussions) | [Discord](https://discord.gg/teable)