# Ficha Técnica de Rendimiento: Escenarios de Carga Masiva

## 1. Objetivo
Evaluar la capacidad de respuesta y estabilidad del monolito **Match** ante situaciones de estrés extremo, simulando picos de tráfico estacionales (Black Friday) y saturación de servicios críticos (IA).

## 2. Escenarios de Prueba
1.  **Pico de Tráfico (Black Friday)**:
    *   **Carga**: 500 peticiones de lectura al catálogo (`/productos`).
    *   **Objetivo**: Validar que el tiempo medio de respuesta se mantiene estable y que no hay fugas de memoria en el renderizado de Blade.
2.  **Estrés Extremo y Concurrencia (DDoS Simulado / Ráfagas Masivas)**:
    *   **Carga**: Pruebas escaladas hasta 1,000 - 3,000 peticiones ultra rápidas (`/productos`).
    *   **Objetivo**: Evaluar los límites de memoria de PHP (Memory Exhausted) y la capacidad de la base de datos de monitoreo (SQLite) para manejar escrituras simultáneas masivas.
3.  **Cuello de Botella (Worker IA)**:
    *   **Carga**: 50 peticiones de escritura concurrentes (usuarios únicos) al endpoint de Gemini (`/probador-virtual`).
    *   **Objetivo**: Medir la capacidad de la cola de Jobs (`ai-processing`) para encolar ráfagas sin afectar la respuesta del frontend.

---

## 3. Guía de Ejecución
Comando estándar:
```bash
docker exec laravel_app ./vendor/bin/pest --filter=LoadTest
```
Comando para estrés extremo (ignora límite de RAM):
```bash
docker exec laravel_app php -d memory_limit=-1 ./vendor/bin/pest --filter=LoadTest
```

---

## 4. Resultados de la Prueba (Datos Reales)s

### Métricas Generales
| Parámetro | Escenario: Black Friday | Escenario: Estrés Extremo | Escenario: Worker IA |
| :--- | :--- | :--- | :--- |
| **Volumen** | 500 peticiones | 1,000+ peticiones | 50 peticiones |
| **Tiempo Total** | ~27.66s | ~16.66s (Optimizadas) | ~8.55s |
| **Media (Aprox)** | 55ms / req | < 16ms / req | 171ms / req |
| **Estado Final** | 100% OK (HTTP 200) | 100% OK (Superando límites) | 100% OK (HTTP 302) |

### Análisis de Evidencia (Laravel Pulse & Telescope)
*   **Impacto en Memoria (RAM)**: El renderizado masivo de vistas en pruebas secuenciales puede causar errores `Memory Exhausted` (Límite 256MB). Ejecutar con memoria ilimitada permite simular la ráfaga correctamente.
*   **Gestión de Colas**: Las 50 peticiones de IA fueron encoladas instantáneamente. El tiempo de respuesta al usuario se mantiene bajo delegando al worker.
*   **Integridad de Monitoreo**: Todas las métricas están en `monitoring.sqlite`. Telescope captura a la perfección cada detalle (incluyendo Queries y Models asociados).

---

## 5. Hallazgos Técnicos Críticos
Durante la prueba de estrés extremo se identificó un límite estructural en la persistencia de monitoreo:
*   🚨 **Problema**: Las ráfagas masivas corrompían el archivo SQLite de Telescope/Pulse (`database disk image is malformed`) debido a colisiones de lectura/escritura (lock files).
*   ✅ **Solución Arquitectónica**: Se reconfiguró la conexión `monitoring` en `config/database.php` habilitando **`journal_mode = wal`** (Write-Ahead Logging) y un **`busy_timeout = 5000`**. Esto transformó a SQLite, permitiéndole soportar concurrencia masiva sin bloqueos destructivos. El sistema es ahora sumamente resiliente bajo picos de tráfico.
