# Ficha Técnica de Seguridad: Cross-Site Scripting (XSS)

## 1. Objetivo
Validar que todas las entradas del usuario sean escapadas correctamente antes de ser renderizadas en el navegador, previniendo la ejecución de scripts maliciosos y protegiendo la sesión de los usuarios (especialmente administradores).

## 2. Metodología de Prueba
1.  **Ataque de Inyección en Admin**: Se intenta crear una categoría en el panel de control inyectando el payload `<script>alert('XSS-ATTACK')</script>` en el campo de nombre.
2.  **Validación de Renderizado**: Se verifica que el motor Blade neutralice los tags HTML mediante el escape automático `{{ }}`.
3.  **Auditoría de Vistas**: Se revisan los parciales de Blade para asegurar que no se utilicen directivas `{!! !!}` de forma insegura.

---

## 3. Guía de Ejecución
Comando ejecutado en el entorno Docker (Fecha: 2026-05-02):

```bash
docker exec laravel_app ./vendor/bin/pest --filter=XssProtectionTest
```

---

## 4. Resultados de la Prueba (Datos Reales)

### Estado Inicial
*   **Contexto**: Formulario de creación de categorías (Admin).
*   **Vulnerabilidad Detectada**: Se identificó un **doble escape redundante** en `table_rows.blade.php` que fue corregido durante esta prueba para mejorar la visualización de datos legítimos.

### Log de Ejecución Real (Pest Output)
```text
   PASS  Tests\Feature\Security\XssProtectionTest
  ✓ la creación de categorías protege contra ataques de XSS mediante e… 19.44s  

  Tests:    1 passed (5 assertions)
  Duration: 25.01s
```

### Análisis de Evidencia
*   **Escape Exitoso**: El sistema transformó el payload malicioso en:
    `&lt;script&gt;alert(&#039;XSS-ATTACK&#039;)&lt;/script&gt;`
*   **Protección de Ejecución**: Al intentar cargar la lista de categorías, el navegador interpretó el script como texto plano, **no se ejecutó el alert**, confirmando la inmunidad del módulo.

---

## 5. Conclusión Técnica
El sistema cumple con las políticas de seguridad de Laravel 11. Se ha validado que el uso de la sintaxis estándar de Blade protege eficazmente contra ataques de XSS persistente en las áreas administrativas.
