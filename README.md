# 🐳 Redmine UNCuyo – Docker Deployment

Despliegue de **Redmine 5.1.3** con **PostgreSQL 14.11** usando **Docker Compose**.  
Versiones **pinneadas** para reproducibilidad y upgrades controlados.  
Incluye configuración SMTP UNCuyo y guías de **backup / restore**.

---

## 🧩 Requisitos

- Docker Engine ≥ 24  
- Docker Compose ≥ 2  
- Acceso a internet para descargar imágenes  
- Archivo `.env` basado en `.env.example` *(no se versiona)*

---

## ⚙️ Configuración

1️⃣ Copiar y completar el archivo `.env` (no subir a Git):

```bash
cp .env.example .env
vim .env
```

---

## ▶️ Puesta en marcha

Iniciar los contenedores:

```bash
docker compose pull
docker compose up -d
```

Aplicación disponible en: [http://localhost:3000](http://localhost:3000)

Ver logs en tiempo real:

```bash
docker compose logs -f
```

---

## 🧱 Estructura del proyecto

```
.
├── docker-compose.yml        # versiones pinneadas
├── .env.example              # plantilla sin secretos
├── .gitignore                # evita subir .env y binarios
├── circle/README.md          # instrucciones tema Circle
└── purplemine2/README.md     # instrucciones tema PurpleMine2
```

---

## 🎨 Temas (no versionados por peso/licencias)

- **Circle** → [https://github.com/akabekobeko/redmine-theme-circle](https://github.com/akabekobeko/redmine-theme-circle) *(recomendado v2.2.4)*  
- **PurpleMine2** → [https://github.com/mrliptontea/PurpleMine2](https://github.com/mrliptontea/PurpleMine2)

Copiar dentro del contenedor (volumen `redmine_themes`):

```bash
/usr/src/redmine/public/themes/<tema>
```

Reiniciar Redmine para aplicar el tema:

```bash
docker compose restart redmine
```

---

## 🔁 Upgrade controlado

1️⃣ Probar primero en un entorno de test.  
2️⃣ Cambiar los tags en `docker-compose.yml` (por ejemplo `redmine:5.1.4-alpine`).  
3️⃣ Actualizar imágenes y recrear contenedores:

```bash
docker compose pull
docker compose up -d
```

Si algo falla, volver a la versión anterior (las versiones *pinneadas* facilitan rollback).

---

## 💾 Backups

Ubicaciones / nombres (volúmenes Docker):

- DB: `db_data`  
- Archivos: `redmine_files`  
- Plugins: `redmine_plugins`  
- Temas: `redmine_themes`

### 🧱 Backup de base de datos (pg_dump)

```bash
# Dump comprimido (.sql.gz)
docker exec -e PGPASSWORD="${POSTGRES_PASSWORD}" postgres-redmine   pg_dump -U "${POSTGRES_USER}" -d "${POSTGRES_DB}"   | gzip > backups/redmine_db_$(date +%F).sql.gz
```

### 📦 Backup de volúmenes

```bash
mkdir -p backups

# Archivos adjuntos
docker run --rm -v redmine_files:/data -v "$(pwd)/backups:/backup" busybox   tar -czf /backup/redmine_files_$(date +%F).tar.gz -C / data

# Plugins
docker run --rm -v redmine_plugins:/data -v "$(pwd)/backups:/backup" busybox   tar -czf /backup/redmine_plugins_$(date +%F).tar.gz -C / data

# Temas
docker run --rm -v redmine_themes:/data -v "$(pwd)/backups:/backup" busybox   tar -czf /backup/redmine_themes_$(date +%F).tar.gz -C / data
```

💡 *Tip:* automatizar con cron o systemd timers, manteniendo rotación (7 diarios + 4 semanales).

---

## ♻️ Restore

### 🧱 Restaurar base de datos

```bash
gunzip -c backups/redmine_db_YYYY-MM-DD.sql.gz | docker exec -i -e PGPASSWORD="${POSTGRES_PASSWORD}" postgres-redmine   psql -U "${POSTGRES_USER}" -d "${POSTGRES_DB}"
```

### 📦 Restaurar volúmenes

```bash
# Archivos
docker run --rm -v redmine_files:/data -v "$(pwd)/backups:/backup" busybox   sh -c "rm -rf /data/* && tar -xzf /backup/redmine_files_YYYY-MM-DD.tar.gz -C /"

# Plugins
docker run --rm -v redmine_plugins:/data -v "$(pwd)/backups:/backup" busybox   sh -c "rm -rf /data/* && tar -xzf /backup/redmine_plugins_YYYY-MM-DD.tar.gz -C /"

# Temas
docker run --rm -v redmine_themes:/data -v "$(pwd)/backups:/backup" busybox   sh -c "rm -rf /data/* && tar -xzf /backup/redmine_themes_YYYY-MM-DD.tar.gz -C /"
```

Recrear contenedores:

```bash
docker compose up -d
```

---

## 🩺 Healthchecks incluidos

- DB: `pg_isready`  
- App: `GET /login` local

---

## 🔒 Seguridad

- Nunca subir `.env` ni secretos al repositorio.  
- Rotar `REDMINE_SECRET_KEY_BASE` si se expone.  
- Mantener tags *pinneadas* y actualizaciones controladas.  
- Activar protecciones de rama en GitHub:  
  - No permitir `force-push` ni `delete` sobre `main`.  
  - Usar *Pull Requests* para actualizaciones.

---

## 📄 Licencia

MIT / Apache-2.0 *(definir según política institucional)*  

---

**Autor:** Sebastián Armando Vargas – Secretaría de Transformación Digital, UNCuyo  
**Contacto:** sebastian.vargas@uncuyo.edu.ar
