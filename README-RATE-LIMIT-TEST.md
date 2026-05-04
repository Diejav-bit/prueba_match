# Ficha Técnica de Seguridad: Protección de Endpoints (Rate Limiting)

## 1. Objetivo
Proteger el endpoint de Inteligencia Artificial (Gemini) contra abusos y ataques de denegación de servicio (DoS). Se busca asegurar que ningún usuario o atacante pueda agotar la cuota de la API de Google mediante peticiones masivas, garantizando la disponibilidad del servicio para todos los clientes.

## 2. Metodología de Prueba
1.  **Configuración de Umbral**: Se define un límite de **10 peticiones por minuto** por usuario (o IP).
2.  **Prueba de Carga**: Se realizan 11 peticiones consecutivas en menos de un segundo.
3.  **Validación de Bloqueo**: Se verifica que las primeras 10 peticiones sean aceptadas (HTTP 302) y que la petición 11 sea rechazada con el código de error correspondiente.

---

## 3. Guía de Ejecución
Comando ejecutado en el entorno Docker (Fecha: 2026-05-02):

```bash
docker exec laravel_app ./vendor/bin/pest --filter=RateLimitTest
```

---

## 4. Resultados de la Prueba (Datos Reales)

### Estado Inicial e Infraestructura
*   **Aislamiento**: La prueba se ejecutó en **SQLite :memory:** para proteger la integridad del usuario administrador en PostgreSQL.
*   **Persistencia de Logs**: Los resultados se grabaron en la conexión dedicada `monitoring` (`database/monitoring.sqlite`).
*   **Endpoint**: `POST /probador-virtual`
*   **Middleware**: `throttle:gemini`

### Log de Ejecución Real (Pest Output)
```text
   PASS  Tests\Feature\Security\RateLimitTest
  ✓ el endpoint de IA (Gemini) tiene un limitador de tasa de 10 petici… 18.22s  

  Tests:    1 passed (11 assertions)
  Duration: 22.91s
```

### Análisis de Evidencia
*   **Peticiones 1-10**: Aceptadas correctamente (Redirección 302).
*   **Petición 11**: Bloqueada con **HTTP 429 Too Many Requests**.
*   **Monitoreo**: El rastro de estas 11 peticiones es **visible y persistente** en el dashboard de **Laravel Pulse** y **Telescope**, facilitando auditorías posteriores sin afectar la base de datos de producción.

---

## 5. Conclusión Técnica
La arquitectura de seguridad implementada protege eficazmente los recursos costosos de IA. Gracias a la separación de bases de datos para monitoreo, las pruebas de integración ahora proporcionan visibilidad total sin comprometer la estabilidad del entorno de desarrollo.
