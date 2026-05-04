# 🧪 ESTRATEGIA INTEGRAL DE TESTING - PROYECTO MATCH
> **Documento de Arquitectura de Pruebas (QA) para E-commerce con IA en Laravel 11**

Este documento define la estrategia exhaustiva de pruebas adaptada estrictamente al ecosistema actual del proyecto **Match** (Laravel 11, PostgreSQL, Docker, Gemini AI, Wompi).

---

## 📂 1. CLASIFICACIÓN DE PRUEBAS E IMPLEMENTACIÓN TÉCNICA

Para este stack (PHP/Laravel), utilizaremos las herramientas nativas y estándares de la industria para garantizar la máxima compatibilidad.

### 1.1 Pruebas Unitarias (Unit Testing)
*   **Objetivo:** Validar la lógica aislada (ej. Servicios, Modelos).
*   **Herramienta:** **Pest PHP** (o PHPUnit nativo de Laravel).
*   **Ejemplo Real en Match:** Probar que `VirtualFittingService` genera el payload correcto para Gemini, sin hacer la petición HTTP real (haciendo un *Mock* del cliente HTTP).
*   **Estructura:** `tests/Unit/Services/VirtualFittingServiceTest.php`

### 1.2 Pruebas de Integración y Base de Datos (Feature Testing)
*   **Objetivo:** Validar que los componentes trabajan juntos, incluyendo la BD.
*   **Herramienta:** **Pest PHP** + trait `RefreshDatabase` de Laravel.
*   **Ejemplo Real en Match:** Simular un checkout completo. Verificar que al llamar a `CheckoutController`, se reduce el stock en PostgreSQL, se crea la orden y se genera el `AuditLog`.
*   **Estructura:** `tests/Feature/Controllers/CheckoutControllerTest.php`

### 1.3 Pruebas Funcionales y E2E (End-to-End)
*   **Objetivo:** Simular la interacción real de un usuario en el navegador.
*   **Herramienta:** **Laravel Dusk** (Controla un navegador Chrome headless).
*   **Ejemplo Real en Match:** Un script que abra el navegador, haga login, navegue a un producto, haga clic en "Probar con IA", suba una foto y espere a ver el modal con el resultado.
*   **Estructura:** `tests/Browser/VirtualFittingFlowTest.php`

### 1.4 Pruebas de UI/UX
*   **Objetivo:** Validar la coherencia visual y el Dark Mode.
*   **Herramienta:** Integrado en **Laravel Dusk** (tomando capturas de pantalla) + revisión manual.

---

## 🛡️ 2. PRUEBAS DE SEGURIDAD (DevSecOps)

Laravel ya previene muchas vulnerabilidades (usando PDO para SQLi, Blade escapa XSS, middleware VerifyCsrfToken). Sin embargo, debemos probar configuraciones específicas.

### 2.1 Inyección SQL (SQLi)
*   **Cómo Probar (Herramienta):** Usar **OWASP ZAP** (Zed Attack Proxy) escaneando los endpoints públicos.
*   **Cómo Probar (Código):** Verificar en `tests/Feature` que enviar parámetros como `' OR 1=1 --` en el buscador de productos no rompa la consulta ni devuelva datos sensibles.

### 2.2 Cross-Site Scripting (XSS)
*   **Cómo Probar:** En el formulario de "Comentarios de Orden" o al crear una Categoría en el Admin, inyectar `<script>alert('XSS')</script>`.
*   **Validación:** Asegurarse de que Blade imprime los datos con `{{ $data }}` y no con `{!! $data !!}` a menos que sea estrictamente necesario.

### 2.3 Control de Acceso y Rutas
*   **Ejemplo Real en Match:** Escribir un Feature Test que intente hacer un `GET /admin/dashboard` autenticado como cliente normal.
*   **Resultado Esperado:** Laravel debe retornar `403 Forbidden` o redirigir, validando que el `RoleMiddleware` funciona.

### 2.4 Protección de Endpoints (Rate Limiting)
*   **Riesgo:** Un usuario malicioso podría llamar repetidamente al endpoint de IA de Gemini, consumiendo la cuota de la API.
*   **Prueba:** Configurar un test que haga 20 peticiones seguidas al endpoint de IA.
*   **Resultado Esperado:** A partir de la petición 10, Laravel debe devolver `429 Too Many Requests`.

---

## 📈 3. PRUEBAS DE ESTRÉS Y RENDIMIENTO

Dado que usamos contenedores Docker y un Job Worker pesado para la IA, el rendimiento es crítico.

### 3.1 Herramienta Recomendada: K6 (Grafana)
K6 es ideal para probar APIs y flujos HTTP. Se ejecuta mediante un script en JavaScript.

### 3.2 Escenarios de Carga en Match
*   **Pico de Tráfico (Black Friday):** 500 usuarios concurrentes navegando el catálogo.
*   **Cuello de Botella (Worker IA):** 50 usuarios enviando peticiones de "Probar con IA" al mismo tiempo.

**Ejemplo de Script K6 (`tests/Performance/ai_load_test.js`):**
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
    vus: 50, // 50 usuarios virtuales
    duration: '30s', // durante 30 segundos
};

export default function () {
    let payload = JSON.stringify({ product_id: 1, image: 'base64_string...' });
    let params = { headers: { 'Content-Type': 'application/json' } };
    
    // Suponiendo un endpoint de API
    let res = http.post('http://localhost:8000/api/ai/fitting', payload, params);
    
    check(res, {
        'status is 200 or 202': (r) => r.status === 200 || r.status === 202,
        'response time < 2s': (r) => r.timings.duration < 2000,
    });
    sleep(1);
}
```
*Cómo ejecutar:* `k6 run tests/Performance/ai_load_test.js`

---

## 🤖 4. AUTOMATIZACIÓN E INTEGRACIÓN CONTINUA (CI/CD)

Las pruebas no sirven si no se ejecutan automáticamente.

### 4.1 Pipeline (GitHub Actions)
Crear un archivo `.github/workflows/testing.yml` que:
1.  Levante los servicios con Docker (`postgres`, `redis`).
2.  Instale dependencias de Composer.
3.  Ejecute SonarQube para análisis estático.
4.  Ejecute `php artisan test` (Unitarias e Integración).
5.  Bloquee el *Merge* si la cobertura de código es menor al 70%.

---

## 📝 5. DOCUMENTACIÓN DE RESULTADOS (PLANTILLA)

Cada ciclo de pruebas debe documentarse en un archivo (ej. `REPORTES/PRUEBAS_V1.md`) con la siguiente estructura:

### Plantilla de Reporte de Ejecución
*   **Módulo:** Checkout / Pagos
*   **Objetivo de la Prueba:** Validar bloqueo de stock concurrente.
*   **Método/Herramienta:** Feature Test (Pest) simulando 2 transacciones simultáneas al mismo producto con stock = 1.
*   **Resultados Esperados:** La primera compra es exitosa, la segunda falla con error de "Stock Insuficiente" gracias a `lockForUpdate()`.
*   **Resultados Obtenidos:** [PASS] Las transacciones respetaron el ACID.
*   **Evidencia:** 
    *   *Logs de Telescope adjuntos.*
    *   *Tiempo de ejecución: 140ms.*

---

## 🎯 6. MEJORES PRÁCTICAS Y MEJORAS DETECTADAS EN MATCH

Durante este análisis de QA, he detectado áreas de mejora para aplicar en el código actual:

1.  **Rate Limiting en IA:** Urgente implementar un límite estricto (ej. `RateLimiter::for('ai-fitting', function(...) { return Limit::perMinute(3)->by($request->user()->id); })`) para evitar agotamiento de cuota de Gemini.
2.  **Seguridad CSP (Content Security Policy):** Agregar cabeceras CSP en Nginx o middleware de Laravel para bloquear ejecución de scripts externos (protección XSS).
3.  **Monitoreo Constante:** Configurar alertas (ej. Slack/Discord) desde Telescope cuando el `ProcessVirtualFittingJob` falle más de 3 veces seguidas, indicando un posible fallo en la API de Google.
