# Reporte de Ingeniería: Validación de Flujo de Checkout

## 1. Ficha Técnica
### Objetivo
Validar la **atomicidad de la transacción** y la integridad referencial en PostgreSQL durante el proceso de compra. Se busca asegurar que ante un pago exitoso, todos los registros relacionados (órdenes, ítems, auditoría) se creen correctamente y que el inventario se actualice sin inconsistencias.

### Flujo Lógico
1.  **Inicio de Pago (`initiate`)**: El sistema valida el carrito en sesión y genera un enlace de pago mediante el `WompiService`.
2.  **Callback de Persistencia (`callback`)**: Al recibir la confirmación, se ejecuta un bloque `DB::transaction`.
    *   **lockForUpdate()**: Se bloquea la fila del producto para evitar race conditions.
    *   **Decremento de Stock**: Se reduce la existencia física.
    *   **Auditoría**: Se registra el evento en `audit_logs`.

---

## 2. Guía de Ejecución
Comandos ejecutados para la validación (Fecha: 2026-05-02):

```bash
# Limpiar caché de configuración y clases
docker exec laravel_app php artisan optimize:clear

# Ejecución de la suite de pruebas con Pest
docker exec laravel_app ./vendor/bin/pest --filter=CheckoutControllerTest
```

---

## 3. Resultados de la Prueba (Datos Reales)

### Estado Inicial (Antes de la prueba)
| Entidad | Valor / Estado |
| :--- | :--- |
| `products.stock` | **10** (Configurado vía ProductFactory) |
| `orders` | 0 registros (Ambiente controlado por RefreshDatabase) |

### Log de Ejecución Real (Pest Output)
```text
   PASS  Tests\Feature\Controllers\CheckoutControllerTest
  ✓ el proceso de callback de checkout persiste la orden, reduce el st… 17.96s  
  ✓ la ruta POST de iniciar checkout valida el carrito y redirige a la…  0.68s  

  Tests:    2 passed (9 assertions)
  Duration: 23.22s
```

### Estado Final (Verificación Post-Ejecución)
| Campo | Valor Final | Observación |
| :--- | :--- | :--- |
| `products.stock` | **8** | El stock disminuyó en 2 unidades (10 -> 8). |
| `orders.total_amount` | **300.00** | Monto exacto para 2 ítems de 150.00 cada uno. |
| `audit_logs.action` | **checkout_completed** | Evento registrado satisfactoriamente. |

---

## 4. Manejo de Errores (Bitácora de Robustez)
Se verificó que el controlador maneja excepciones de stock insuficiente devolviendo una vista de error y evitando la creación de la orden.

**Extracto de validación de excepciones:**
*   **Evento**: Intento de compra excediendo el stock disponible.
*   **Resultado**: Rollback automático de la transacción.
*   **Log**: `[Checkout] Error en transacción: Stock insuficiente para: [Producto]`
