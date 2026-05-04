# Ficha Técnica de Seguridad: Control de Acceso y Rutas (RBAC)

## 1. Objetivo
Validar la efectividad del `RoleMiddleware` en la protección de rutas administrativas. Se busca asegurar que solo los usuarios con el rol `admin` puedan acceder al panel de control, mientras que los clientes (`customer`) o usuarios no autenticados sean bloqueados o redirigidos.

## 2. Metodología de Prueba
1.  **Acceso no autorizado (403)**: Un usuario con rol `customer` intenta realizar un `GET /admin/dashboard`.
2.  **Acceso autorizado (200)**: Un usuario con rol `admin` intenta acceder a la misma ruta.
3.  **Redirección de Invitados**: Un usuario no autenticado intenta acceder y debe ser redirigido a la pantalla de login.

---

## 3. Guía de Ejecución
Comando ejecutado en el entorno Docker (Fecha: 2026-05-02):

```bash
docker exec laravel_app ./vendor/bin/pest --filter=AccessControlTest
```

---

## 4. Resultados de la Prueba (Datos Reales)

### Estado Inicial
*   **Mecanismo**: Middleware `auth` + `role:admin`.
*   **Configuración**: El sistema utiliza una relación `belongsTo` entre el modelo `User` y `Role`.

### Log de Ejecución Real (Pest Output)
```text
   PASS  Tests\Feature\Security\AccessControlTest
  ✓ un cliente normal no puede acceder al panel de administración (403… 17.96s  
  ✓ un usuario administrador puede acceder correctamente al panel        0.59s  
  ✓ un usuario no autenticado es redirigido al login al intentar entrar… 0.48s  

  Tests:    3 passed (4 assertions)
  Duration: 24.25s
```

### Análisis de Evidencia
*   **Bloqueo de Clientes**: Se confirmó que el intento de acceso por parte de un cliente resultó en un estado HTTP **403 Forbidden**. El middleware interceptó la petición antes de llegar al controlador.
*   **Permisividad de Admin**: El usuario administrador accedió con éxito (HTTP **200 OK**), validando que la lógica de comparación de slugs de roles es correcta.
*   **Protección de Invitados**: El sistema redirigió correctamente al login, cumpliendo con la jerarquía de middlewares (`auth` -> `role`).

---

## 5. Conclusión Técnica
El sistema de Control de Acceso Basado en Roles (RBAC) es sólido y se comporta según lo esperado. La separación de responsabilidades entre el middleware de autenticación y el de autorización garantiza una capa de seguridad profunda en el área administrativa.
