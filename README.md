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

## ⚙️ Configuración

1) Copiar y completar el `.env` (no subir a Git):
```bash
cp .env.example .env
vim .env
