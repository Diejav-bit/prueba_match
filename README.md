# 🛡️ GUÍA MAESTRA DE INSTALACIÓN Y DESPLIEGUE - PROYECTO MATCH
> **Manual técnico exhaustivo para la puesta en marcha, auditoría y mantenimiento del ecosistema E-commerce.**

Este documento detalla cada comando necesario para clonar, levantar, configurar y monitorear el sistema completo en un entorno Dockerizado.

---

## 📋 1. PRE-REQUISITOS Y PREPARACIÓN

### 1.1 Herramientas Necesarias
Asegúrate de tener instalado y funcionando:
*   **Docker Desktop:** Indispensable para orquestar los 5 contenedores del proyecto.
*   **Git:** Para obtener el código fuente.

### 1.2 Clonación y Variables de Entorno
Ejecuta estos comandos en tu terminal (CMD o PowerShell):

```bash
# 1. Clonar el repositorio


# 2. Crear archivo de entorno
copy .env.example .env

# 3. Editar el archivo .env (Configuración Crítica)
# Abre el archivo .env y asegúrate de que estas líneas sean correctas:
# DB_CONNECTION=pgsql
# DB_HOST=laravel_db
# GEMINI_API_KEY=tu_api_key_real
```

---

## 🏗️ 2. LEVANTAMIENTO DE LA INFRAESTRUCTURA (DOCKER)

### 2.1 Construcción y Despliegue
Este comando descarga las imágenes, crea la red interna y levanta los servicios.

```bash
docker-compose up -d --build
```

### 2.2 Verificación de Salud de Contenedores
Es vital confirmar que los 5 servicios principales estén en estado `Up`.

```bash
# Listar contenedores activos y sus puertos
docker ps

# Ver el estado de uso de recursos (CPU/RAM) de cada contenedor
docker stats --no-stream
```
> **Servicios esperados:** `laravel_app`, `laravel_db`, `laravel_nginx`, `ai_worker`, `pulse_worker`.

---

## 🚀 3. CONFIGURACIÓN DEL CORE DE LA APLICACIÓN

Debes ejecutar estos comandos para inicializar Laravel dentro del contenedor principal (`laravel_app`).

```bash
# 1. Instalar librerías de PHP (Composer)
docker exec laravel_app composer install

# 2. Generar llave única de seguridad
docker exec laravel_app php artisan key:generate

# 3. Crear tablas en la base de datos PostgreSQL
docker exec laravel_app php artisan migrate

# 4. Cargar datos iniciales (Categorías, Productos y Admin)
docker exec laravel_app php artisan db:seed

# 5. Habilitar almacenamiento de imágenes (Storage Link)
docker exec laravel_app php artisan storage:link

# 6. Limpiar cualquier caché residual
docker exec laravel_app php artisan optimize:clear
```

> [!IMPORTANT]
> **Credenciales de Acceso Admin:**
> *   **URL:** `http://localhost:8000/login`
> *   **Usuario:** `admin@match.com`
> *   **Password:** `password`

---

## ⚡ 4. INSTALACIÓN DE HERRAMIENTAS DE MONITOREO (PULSE/TELESCOPE)

Si las herramientas no están instaladas por defecto, ejecuta esta secuencia para habilitar el Dashboard de Rendimiento y el Debugger de IA.

```bash
# 1. Descargar paquetes de monitoreo
docker exec laravel_app composer require laravel/pulse laravel/telescope

# 2. Instalar Telescope (Debugger de base de datos y Jobs)
docker exec laravel_app php artisan telescope:install

# 3. Publicar assets de Pulse (Monitor de Servidor)
docker exec laravel_app php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"

# 4. Aplicar nuevas tablas de monitoreo
docker exec laravel_app php artisan migrate
```

---

## ⚙️ 5. MANTENIMIENTO Y TROUBLESHOOTING (COMANDOS ÚTILES)

### 5.1 Reiniciar Procesamiento de IA
Si haces cambios en el código de la IA, debes reiniciar el trabajador para que los tome:
```bash
docker exec ai_worker php artisan queue:restart
```

### 5.2 Ver Logs en Tiempo Real
Si algo falla, revisa las tripas de los contenedores:
```bash
# Ver errores de la aplicación Laravel
docker logs -f laravel_app

# Ver por qué la IA no responde o da error
docker logs -f ai_worker
```

### 5.3 Entrar a la Consola del Contenedor
Si necesitas ejecutar comandos manuales directamente dentro de Linux:
```bash
docker exec -it laravel_app bash
```

### 5.4 Limpiar Todo y Reiniciar (Reset Total)
En caso de corrupción de datos o errores extraños de caché:
```bash
docker exec laravel_app php artisan config:clear
docker exec laravel_app php artisan cache:clear
docker exec laravel_app php artisan view:clear
```

---
**El sistema está ahora 100% operativo en `http://localhost:8000`.**
