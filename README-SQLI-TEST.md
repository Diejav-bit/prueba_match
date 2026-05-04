# Ficha Técnica de Seguridad: SQL Injection (SQLi)

## 1. Objetivo
Validar la inmunidad del motor de búsqueda de productos frente a ataques de inyección de código SQL. Se busca asegurar que el uso de parámetros enlazados en Eloquent/PostgreSQL previene la fuga de información o la alteración de la lógica de negocio.

## 2. Metodología de Prueba
1.  **Ataque de Bypass de Lógica**: Se inyecta `' OR 1=1 --` en el parámetro de búsqueda para intentar forzar la visualización de todo el catálogo, saltándose los filtros de `is_active` o `category_id`.
2.  **Ataque de Interrupción de Consulta**: Se inyectan caracteres de control (`;`, `--`, `/*`) y sentencias destructivas simuladas (`DROP TABLE`) para verificar que el ORM escapa correctamente las entradas.

---

## 3. Guía de Ejecución
Comando ejecutado en el entorno Docker (Fecha: 2026-05-02):

```bash
docker exec laravel_app ./vendor/bin/pest --filter=SqlInjectionTest
```

---

## 4. Resultados de la Prueba (Datos Reales)

### Estado Inicial (PostgreSQL)
*   **Motor**: PostgreSQL 16
*   **Protección**: Parámetros enlazados (PDO) activos en el `ProductRepository`.

### Log de Ejecución Real (Pest Output)
```text
   PASS  Tests\Feature\Security\SqlInjectionTest
  ✓ la búsqueda de productos es inmune a ataques de SQL Injection bási… 17.21s  
  ✓ la búsqueda con caracteres especiales no rompe la consulta SQL       1.05s  

  Tests:    2 passed (5 assertions)
  Duration: 23.25s
```

### Análisis de Evidencia
*   **Inmunidad Confirmada**: Al enviar el payload `' OR 1=1 --`, el sistema devolvió **0 resultados**. Esto confirma que el motor buscó la cadena de texto literal en lugar de ejecutar el comando lógico `OR`.
*   **Integridad de Tablas**: Tras el intento de inyección con `; DROP TABLE`, se verificó la persistencia de la tabla `users`, confirmando que el `;` fue tratado como parte de la cadena de búsqueda y no como un terminador de sentencia SQL.

---

## 5. Conclusión Técnica
El proyecto **Match** implementa de forma correcta las defensas de nivel 1 (Sanitización de entradas) y nivel 2 (Parameter Binding) exigidas por OWASP. Las consultas en el `ProductRepository` son seguras y no permiten la manipulación del árbol de sintaxis SQL (AST).
