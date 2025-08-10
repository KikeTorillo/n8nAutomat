# Carpeta de Datos

Esta carpeta contiene los datos persistentes de los contenedores Docker.

## Estructura:

- `postgres/` - Datos de la base de datos PostgreSQL
- `n8n/` - Workflows, credenciales y configuraciones de n8n  
- `evolution/` - Instancias y datos de Evolution API

## ⚠️ Importante:

- Estas carpetas están en `.gitignore` y NO se suben al repositorio
- Contienen datos sensibles como credenciales y configuraciones
- Hacer respaldos regulares de estas carpetas en producción

## Respaldos:

```bash
# Respaldar todo
tar -czf backup-$(date +%Y%m%d).tar.gz data/

# Respaldar solo base de datos
tar -czf postgres-backup-$(date +%Y%m%d).tar.gz data/postgres/
```