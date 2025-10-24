# Redmine UNCuyo – Docker Deployment

Despliegue de **Redmine 5.1.3** con **PostgreSQL 14.11** usando Docker Compose.  
Versiones **pinneadas** para reproducibilidad y upgrades controlados. Incluye SMTP UNCuyo y guías de backup/restore.

---

## 🧩 Requisitos
- Docker Engine ≥ 24
- Docker Compose ≥ 2
- Acceso a internet para descargar imágenes
- Archivo `.env` basado en `.env.example` (no se versiona)

---
 ⚙️ Configuración

1) Copiar y completar el `.env` (no subir a Git):
```bash
cp .env.example .env
vim .env

---

## ▶️ Puesta en marcha

```bash
docker compose pull
docker compose up -d
App disponible en: http://localhost:3000
Ver logs en tiempo real:

bash
Copiar código
docker compose logs -f
🧱 Estructura
arduino
Copiar código
.
├── docker-compose.yml        # versiones pinneadas
├── .env.example              # plantilla sin secretos
├── .gitignore                # no subir .env ni binarios
├── circle/README.md          # instrucciones tema Circle
└── purplemine2/README.md     # instrucciones tema PurpleMine2
🎨 Temas (no versionados por peso/licencias)
Circle: https://github.com/akabekobeko/redmine-theme-circle (recomendado v2.2.4)

PurpleMine2: https://github.com/mrliptontea/PurpleMine2

Copiar dentro del contenedor (volumen redmine_themes):

swift
Copiar código
/usr/src/redmine/public/themes/<tema>
y reiniciar:

bash
Copiar código
docker compose restart redmine
🔁 Upgrade (controlado)
Probar primero en un entorno de test.

Cambiar tags en docker-compose.yml a las nuevas versiones (por ej. redmine:5.1.4-alpine).

Actualizar imágenes y recrear:

bash
Copiar código
docker compose pull
docker compose up -d
Si algo falla, volver a la versión anterior (las versiones pinneadas facilitan rollback).

💾 Backups
Ubicaciones / nombres (docker volumes):

DB: db_data

Archivos: redmine_files

Plugins: redmine_plugins

Temas: redmine_themes

1) Backup de la base de datos (pg_dump)
bash
Copiar código
# Dump comprimido (.sql.gz)
docker exec -e PGPASSWORD="${POSTGRES_PASSWORD}" postgres-redmine \
  pg_dump -U "${POSTGRES_USER}" -d "${POSTGRES_DB}" \
  | gzip > backups/redmine_db_$(date +%F).sql.gz
2) Backup de volúmenes (archivos adjuntos, plugins, temas)
bash
Copiar código
mkdir -p backups

# Archivos adjuntos
docker run --rm -v redmine_files:/data -v "$(pwd)/backups:/backup" busybox \
  tar -czf /backup/redmine_files_$(date +%F).tar.gz -C / data

# Plugins
docker run --rm -v redmine_plugins:/data -v "$(pwd)/backups:/backup" busybox \
  tar -czf /backup/redmine_plugins_$(date +%F).tar.gz -C / data

# Temas
docker run --rm -v redmine_themes:/data -v "$(pwd)/backups:/backup" busybox \
  tar -czf /backup/redmine_themes_$(date +%F).tar.gz -C / data
💡 Tip: automatizá con cron/systemd timers y conservá rotaciones (por ejemplo, 7 diarios + 4 semanales).

♻️ Restore
1) Restaurar DB
bash
Copiar código
gunzip -c backups/redmine_db_YYYY-MM-DD.sql.gz | \
docker exec -i -e PGPASSWORD="${POSTGRES_PASSWORD}" postgres-redmine \
  psql -U "${POSTGRES_USER}" -d "${POSTGRES_DB}"
2) Restaurar volúmenes
bash
Copiar código
# Archivos
docker run --rm -v redmine_files:/data -v "$(pwd)/backups:/backup" busybox \
  sh -c "rm -rf /data/* && tar -xzf /backup/redmine_files_YYYY-MM-DD.tar.gz -C /"

# Plugins
docker run --rm -v redmine_plugins:/data -v "$(pwd)/backups:/backup" busybox \
  sh -c "rm -rf /data/* && tar -xzf /backup/redmine_plugins_YYYY-MM-DD.tar.gz -C /"

# Temas
docker run --rm -v redmine_themes:/data -v "$(pwd)/backups:/backup" busybox \
  sh -c "rm -rf /data/* && tar -xzf /backup/redmine_themes_YYYY-MM-DD.tar.gz -C /"
Recrear contenedores:

bash
Copiar código
docker compose up -d
🩺 Healthchecks incluidos
DB: pg_isready

App: GET /login local

🔒 Seguridad
Nunca subir .env ni secretos al repositorio.

Rotar REDMINE_SECRET_KEY_BASE si se expone.

Mantener tags pinneadas y actualizar de forma controlada.

Activar protecciones de rama en GitHub:

No permitir force-push ni delete sobre main.

Usar pull requests para actualizaciones.

📄 Licencia
MIT / Apache-2.0 (definir según política institucional)

Autor: Sebastián Armando Vargas – Secretaría de Transformación Digital, UNCuyo
Contacto: nodo@uncu.edu.ar

yaml
Copiar código


