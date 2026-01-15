# VPS Context for Claude - Información del Ambiente

**Última actualización:** 2025-12-16
**Propósito:** Documento de contexto para sesiones de Claude trabajando en proyectos del VPS

---

## Conexión al VPS

```bash
# Conectar via SSH (ya configurado en ~/.ssh/config)
ssh vps

# Equivalente a:
ssh adminmgo@100.64.216.28
```

**Detalles:**
- **Host:** vps (alias en SSH config)
- **IP Tailscale:** 100.64.216.28
- **Usuario:** adminmgo
- **Sistema:** Ubuntu Server
- **Proveedor:** Contabo (vmi2563925.contaboserver.net)

---

## Stack del VPS

### Infraestructura Base

| Servicio | Puerto | Acceso | Notas |
|----------|--------|--------|-------|
| **Coolify** | 8000 | http://100.64.216.28:8000 | Panel de administración |
| **Traefik** | 80/443 | Proxy reverso | Maneja SSL y routing |
| **Traefik Dashboard** | 8081 | http://100.64.216.28:8081 | Solo Tailscale |

### Supabase (Docker Standalone)

> **Migrado de Coolify a standalone el 2025-12-16**

| Componente | Puerto/Acceso | Notas |
|------------|---------------|-------|
| **API URL** | https://supabase.galacticaia.com | URL pública |
| **Studio** | https://supabase.galacticaia.com | Dashboard |
| **PostgreSQL** | Puerto 5432 interno, 5433 externo | Via Docker network |
| **DB Container** | `supabase-db` | Docker standalone |
| **Ubicación** | `~/projects/supabase-standalone/` | docker-compose.yml |
| **Red Docker** | `b0wkw4s04csk4cckwcococw0` | 10.0.5.0/24 |

**Conectar a PostgreSQL desde VPS:**
```bash
# Ejecutar SQL directamente
docker exec supabase-db psql -U postgres -c "TU_SQL_AQUI"

# Entrar a psql interactivo
docker exec -it supabase-db psql -U postgres

# Listar schemas
docker exec supabase-db psql -U postgres -c "\dn"

# Listar tablas de un schema
docker exec supabase-db psql -U postgres -c "SELECT table_name FROM information_schema.tables WHERE table_schema = 'kioskoai';"
```

**Schemas existentes en Supabase:**
- `public` - Esquema por defecto
- `kioskoai` - Schema de KioskoAI
- `becgi` - Schema de BECGI
- `newsletter` - Schema de Newsletter AI
- `finanzas_personales` - Dashboard de finanzas
- `auth` - Supabase Auth (usuarios)
- `storage` - Supabase Storage (archivos)

### n8n (Docker Standalone)

> **Migrado de Coolify a standalone el 2025-12-16**

| Campo | Valor |
|-------|-------|
| **URL** | https://n8n.galacticaia.com |
| **Container** | `n8n-standalone` |
| **PostgreSQL** | `n8n-postgres` (dedicado) |
| **Ubicación** | `~/projects/n8n-standalone/` |
| **Red Docker** | `coolify` + `b0wkw4s04csk4cckwcococw0` |

**Comandos útiles:**
```bash
# Reiniciar n8n
cd ~/projects/n8n-standalone && docker compose restart

# Ver logs
docker logs n8n-standalone --tail 50 -f
```


---

## Proyectos en el VPS

### Ubicación: `/home/adminmgo/projects/`

```
/home/adminmgo/projects/
├── supabase-standalone/  # Base de datos centralizada (14 contenedores)
├── n8n-standalone/       # Automatización de workflows
├── kioskoai/             # Web app de scraping
├── becgi/                # Web app de madurez empresarial
├── graphiti/             # Knowledge graph MCP
├── neo4j/                # Graph database
├── metabase/             # Dashboard de finanzas
├── matomo/               # Web analytics
├── countly/              # Product analytics
└── mindsdb/              # AI/ML (pesado)
```

## Redes Docker

> **Actualizado 2025-12-16:** n8n y Supabase migrados a Docker standalone. Ambos accesibles via red `coolify`.

### Redes Principales

| Red | Subnet | Uso | Servicios Conectados |
|-----|--------|-----|----------------------|
| `coolify` | 10.0.1.0/24 | Infraestructura y routing | Traefik proxy, n8n-standalone, supabase-kong |
| `b0wkw4s04csk4cckwcococw0` | 10.0.5.0/24 | Supabase + servicios compartidos | Supabase (14 containers), newsletter-api, finanzas-api, metabase |
| `kioskoai_kioskoai-network` | 10.0.4.0/24 | KioskoAI producción | kioskoai, stripe-api, crawl4ai |
| `mindsdb_mindsdb_net` | 172.16.241.0/24 | MindsDB | mindsdb_server |

### Redes Obsoletas (pendientes de eliminar)

| Red | Subnet | Estado |
|-----|--------|--------|
| ~~`oowgkg8k48cggcc4sgcs0w0g`~~ | 10.0.2.0/24 | Deprecated - n8n migrado a standalone |

### Conectar Contenedor a Red Externa

```bash
# Conectar servicio a la red de Supabase (para acceso a DB y APIs internas)
docker network connect b0wkw4s04csk4cckwcococw0 nombre-contenedor

# Conectar servicio a coolify (para acceso desde Traefik)
docker network connect coolify nombre-contenedor

# Verificar conexión
docker inspect nombre-contenedor | grep -A 20 Networks
```

### Declarar Redes Externas en docker-compose.yml

```yaml
networks:
  coolify:                       # Para routing via Traefik
    external: true
  b0wkw4s04csk4cckwcococw0:      # Para acceso a Supabase y servicios compartidos
    external: true
    name: b0wkw4s04csk4cckwcococw0
```

### Comunicación entre Servicios

| Desde | Hacia | Via Red |
|-------|-------|---------|
| n8n | Supabase DB | `b0wkw4s04csk4cckwcococw0` (n8n conectado) |
| n8n | MindsDB | `coolify` |
| Metabase | Supabase DB | `b0wkw4s04csk4cckwcococw0` |
| Newsletter API | Supabase DB | `b0wkw4s04csk4cckwcococw0` |
| Finanzas API | Supabase DB | `b0wkw4s04csk4cckwcococw0` |

---

## Firewall (UFW)

**Reglas relevantes:**

| Puerto | Acceso | Servicio |
|--------|--------|----------|
| 22 | Tailscale | SSH |
| 80/443 | Público | Traefik (HTTP/HTTPS) |
| 8000 | Tailscale | Coolify Panel |
| 47334 | Tailscale | MindsDB |
| 5432 | Tailscale | PostgreSQL |

**Agregar regla para nuevo servicio:**
```bash
sudo ufw allow from 100.0.0.0/8 to any port PUERTO comment "SERVICIO - Tailscale"
```

---

## Comandos Útiles

### Docker

```bash
# Ver todos los contenedores con formato
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Logs de un contenedor
docker logs CONTENEDOR --tail 100 -f

# Ejecutar comando en contenedor
docker exec -it CONTENEDOR bash

# Reiniciar servicio
cd /home/adminmgo/projects/PROYECTO && docker compose restart
```

### Supabase SQL

```bash
# Crear schema
docker exec supabase-db psql -U postgres -c "CREATE SCHEMA IF NOT EXISTS nombre_schema;"

# Ejecutar migration desde archivo
docker exec -i supabase-db psql -U postgres < /path/to/migration.sql

# Ver estructura de tabla
docker exec supabase-db psql -U postgres -c "\d kioskoai.subscriptions"
```

### Git en el VPS

```bash
# Los proyectos con GitHub usan tokens configurados
cd /home/adminmgo/projects/kioskoai
git pull origin main
git push origin main

# Ver remote configurado
git remote -v
```

---

## Repositorio my-privateserver

Este documento viene del repositorio `my-privateserver` que documenta toda la infraestructura del VPS.

**Ubicación local:** `D:\APPs\my-privateserver\` o `C:\Users\ronal\APPs\my-privateserver\`

**Estructura relevante:**
```
my-privateserver/
├── CLAUDE.md                    # Guía de trabajo general
├── DISASTER-RECOVERY.md         # Plan de recuperación
├── vps-context-for-claude.md    # ESTE ARCHIVO
├── core/                        # Infraestructura fundamental
│   ├── 00-server-base/          # Configuración base (UFW, redes)
│   ├── 01-coolify/              # Orquestador de deployments
│   ├── 02-supabase/             # Database & Auth (standalone)
│   └── 03-n8n/                  # Automatización workflows (standalone)
├── services/                    # docker-compose de servicios adicionales
└── apps/                        # Referencias a apps con repo propio
```


---

**Contacto con el VPS:** `ssh vps`
**Base de datos:** `docker exec supabase-db psql -U postgres`
