# DataSkills Hub

Plataforma de guías de referencia organizada por roles profesionales para el Data Team de TIGO.

## Stack
- **Motor**: Hexo (Node.js 20+)
- **Estilos**: Tailwind CSS
- **Deploy**: Docker + Nginx

## Roles
- **Data Engineer**: Python, Docker, PostgreSQL, Bash, Git, Kubernetes, Terraform
- **Data Analyst**: Python, Pandas, NumPy, Matplotlib, MySQL, PostgreSQL
- **Data Visualization**: Superset, Metabase, Grafana (guías por crear)
- **DevOps**: Docker, Kubernetes, Terraform, Bash

## Flujo de Trabajo

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  GitHub         │     │  VPS Personal   │     │  GitLab Corp    │
│  (código)       │────▶│  (desarrollo)   │────▶│  (producción)   │
│                 │     │  localhost:4000 │     │  Rocky Linux    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Desarrollo (VPS Personal)
```bash
ssh vps
cd ~/projects/dataskills-hub
pnpm run server  # http://localhost:4000
```

### Deploy a Producción
```bash
# En servidor corporativo (Rocky Linux)
git clone <repo-gitlab>
docker build -t dataskills-hub .
docker run -d -p 8080:80 dataskills-hub
```

## Archivos Clave
- `source/_data/roles.yml` - Configuración de roles
- `themes/coo/layout/index.ejs` - Landing page
- `themes/coo/layout/_partial/index/role-cards.ejs` - Cards de roles
- `.github/workflows/deploy.yml` - CI/CD GitHub
- `.gitlab-ci.yml` - CI/CD GitLab (para migración futura)

## Enlaces
- **GitHub**: https://github.com/ronaldmego/dataskills-hub
- **VPS**: `ssh vps` → `~/projects/dataskills-hub/`
- Ver [STATUS.md](./STATUS.md) para estado actual
