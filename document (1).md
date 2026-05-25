# ❓ CUESTIONARIO DE AUDITORÍA Y GUÍA DE VALIDACIÓN
> **Proyecto MATCH — Sistema E-commerce con Probador Virtual IA**
>
> Este cuestionario recopila preguntas clave de sustentación basadas en la implementación real de **Match**. Está organizado exclusivamente por **Área**, eliminando nombres personales, y se enfoca en explicar el funcionamiento técnico y los pasos exactos para validar cada punto de manera práctica.

---

## 🧪 ÁREA: CONTROL DE CALIDAD (QA)

### Pregunta 1: *“¿Qué pasa si un usuario intenta agregar 9999 camisas al carrito de compras si en la base de datos solo quedan 3 en stock?”*
*   **Funcionamiento Técnico:**  
    El sistema bloquea la manipulación de cantidades en el servidor. Si el usuario intenta burlar los límites visuales del navegador (por ejemplo, modificando el HTML con F12), el backend aplica una comparación restrictiva. En `CartController.php@add`, se evalúa la cantidad solicitada mediante la función matemática `min($requestedQuantity, $product->stock)`, lo que reajusta automáticamente el carrito al stock máximo real y notifica al cliente la limitación.
*   **Cómo Validarlo:**
    1. Ve al detalle de un producto que tenga existencias limitadas (ej. 3 unidades).
    2. En el navegador, abre las Herramientas de Desarrollador (F12) e inspecciona el campo numérico de cantidad.
    3. Edita el código HTML para cambiar el atributo `max="3"` a `max="9999"`, escribe manualmente `9999` en el input y presiona "Agregar al Carrito".
    4. **Resultado Esperado:** La página se recargará y el carrito mostrará exactamente **3 unidades** (el límite físico de la BD) junto con un mensaje de alerta: *"Cantidad ajustada al stock disponible"*.

### Pregunta 2: *“Si la API de Google Gemini experimenta una caída o se queda sin saldo, ¿el probador virtual se queda colgado cargando infinitamente?”*
*   **Funcionamiento Técnico:**  
    No. El procesamiento se ejecuta de manera asíncrona a través del Job `ProcessVirtualFittingJob.php`. El script de Node.js `ai_fitting.js` maneja reintentos controlados (máximo 3). Si falla de manera permanente, el Job captura la excepción en su método `failed()`, actualiza el estado en la base de datos a `'failed'` y guarda la traza del error en los logs. El frontend realiza sondeos constantes (polling) a `/fitting/poll`, detecta el estado fallido y detiene la animación de carga de forma segura.
*   **Cómo Validarlo:**
    1. Abre el archivo de configuración `.env` en la raíz del proyecto y cambia temporalmente el valor de `GEMINI_API_KEY` por una clave inválida o vacía.
    2. Inicia sesión como cliente, ve a un producto apto para probador virtual y sube una fotografía para procesar.
    3. **Resultado Esperado:** El sistema iniciará la animación de carga. En pocos segundos (después de fallar la comunicación con Google), la animación desaparecerá automáticamente y el modal mostrará el error controlado: *"No se pudo procesar la imagen en este momento"*.

---

## 📊 ÁREA: PRODUCT OWNER (PO)

### Pregunta 1: *“¿Cómo garantizamos a nivel de negocio que los clientes no puedan ver ni comprar productos inactivos o sin stock?”*
*   **Funcionamiento Técnico:**  
    Las consultas públicas del catálogo tienen filtros de negocio obligatorios e inyectados directamente en la capa de servicios. En `ProductService.php@getAvailableProducts`, se encadenan forzosamente las cláusulas de consulta SQL `where('is_active', true)` y `where('stock', '>', 0)`, aislando por completo al usuario de productos no disponibles.
*   **Cómo Validarlo:**
    1. Inicia sesión como Administrador en `/admin` y desactiva el interruptor ("is_active = false") de un producto en el listado, o pon su stock en `0`.
    2. Abre una pestaña de navegación en incógnito como cliente invitado y accede al catálogo en `/productos`.
    3. **Resultado Esperado:** El producto desactivado no se mostrará en el listado general del catálogo. Si intentas escribir manualmente la URL directa del producto (ej: `/productos/nombre-producto-desactivado`), el sistema retornará un error **404 Not Found**.

### Pregunta 2: *“¿Qué sucede a nivel de negocio si se suspende la cuenta de un cliente en el panel administrativo mientras él está navegando en la tienda?”*
*   **Funcionamiento Técnico:**  
    El bloqueo se aplica inmediatamente. Durante el flujo de autenticación y en cada transacción crítica, el sistema verifica el estado de la cuenta. En `AuthService.php`, el método de login y los filtros del middleware comprueban la columna `is_active` del usuario. Si el administrador la ha cambiado a `false`, la sesión activa se invalida (`session()->invalidate()`) y el usuario es redirigido al formulario de inicio de sesión de forma mandatoria.
*   **Cómo Validarlo:**
    1. Inicia sesión como Cliente en un navegador y añade productos al carrito.
    2. En otro navegador (o ventana de incógnito), inicia sesión como Administrador, busca la cuenta de ese cliente y cambia su estado a **Inactivo**.
    3. Regresa a la ventana del Cliente e intenta realizar una acción protegida, como presionar el botón de "Ir al Checkout" o ver el perfil de usuario.
    4. **Resultado Esperado:** La sesión se cerrará de inmediato, el cliente será expulsado al formulario de `/login` y verá un mensaje: *"Su cuenta ha sido inhabilitada. Contacte al soporte técnico"*.

---

## 💻 ÁREA: DESARROLLO (DEV)

### Pregunta 1: *“¿Por qué la generación de imágenes con IA no ralentiza la carga de la página web y dónde se ejecutan esos procesos pesados?”*
*   **Funcionamiento Técnico:**  
    Implementamos una arquitectura asíncrona desacoplada mediante colas (Queues). Al solicitar una prueba de ropa, el hilo web de PHP no espera la respuesta de la IA. Guarda la solicitud en la base de datos con estado `'pending'` y despacha de inmediato el Job `ProcessVirtualFittingJob`. El procesamiento lo realiza un contenedor de Docker aislado llamado `ai_worker` que corre en segundo plano mediante `queue:work`, liberando al servidor web instantáneamente.
*   **Cómo Validarlo:**
    1. En la consola de Windows, ejecuta `docker ps` para verificar que el contenedor `ai_worker` está corriendo de forma paralela a `laravel_app`.
    2. Ejecuta `docker logs -f ai_worker` para ver en tiempo real el log del trabajador de colas.
    3. Inicia una prueba virtual desde la web. Verás que la interfaz responde en menos de **200 milisegundos**, permitiéndote seguir navegando por la web, mientras en la consola del worker de Docker se observa cómo empieza a procesar el script de IA en segundo plano.

### Pregunta 2: *“¿Cómo monitoreamos en tiempo real qué consultas a la base de datos están lentas o si el servidor está sobrecargado de CPU/RAM?”*
*   **Funcionamiento Técnico:**  
    Integramos herramientas de observabilidad de nivel profesional directamente en el ecosistema de Laravel: **Laravel Pulse** para rendimiento y **Laravel Telescope** para auditoría profunda de ejecución.
*   **Cómo Validarlo:**
    1. Inicia sesión como Administrador y navega a la URL `/pulse`.
    2. **Resultado Esperado:** Visualizarás gráficas en tiempo real de consumo de CPU, uso de memoria RAM, las rutas web más lentas y la lista exacta de las consultas SQL que toman más tiempo en la base de datos PostgreSQL.
    3. Navega a la URL `/telescope`.
    4. **Resultado Esperado:** Accederás al panel de desarrollo donde puedes auditar cada petición HTTP realizada, las excepciones que han ocurrido en el código y los detalles de los Jobs de IA despachados.

### Pregunta 3: *“¿Cómo evitamos ataques de Inyección SQL (SQLi) en la barra de búsqueda del catálogo?”*
*   **Funcionamiento Técnico:**  
    Las consultas dinámicas se procesan utilizando el binding de parámetros de Eloquent ORM en lugar de concatenar cadenas directamente en las sentencias SQL. En `ProductService.php`, el buscador utiliza `where('name', 'LIKE', "%{$searchTerm}%")`. PostgreSQL precompila la estructura de la consulta SQL y trata al valor del buscador estrictamente como un dato de búsqueda plano, anulando cualquier comando SQL inyectado.
*   **Cómo Validarlo:**
    1. Ve a la barra de búsqueda pública del catálogo.
    2. Escribe una cadena de inyección SQL común en el input, como: `' OR '1'='1' --` o `' UNION SELECT * FROM users --`.
    3. **Resultado Esperado:** El sistema buscará literalmente esa cadena de texto especial. Al no existir ningún producto con ese nombre literal, el sistema retornará de forma segura: *"No se encontraron productos"* en lugar de colapsar, arrojar errores de base de datos o exponer las tablas internas.
    4. Ejecuta el test de seguridad automatizado en la terminal:
       ```powershell
       php artisan test --filter=SqlInjectionTest
       ```

### Pregunta 4: *“¿Cómo bloqueamos el acceso de usuarios no autorizados a las rutas de administración (/admin)?”*
*   **Funcionamiento Técnico:**  
    Definimos un control de acceso basado en roles (RBAC). En `routes/web.php`, las rutas administrativas están protegidas por el middleware `role:admin`. Cuando entra una petición, se dispara `RoleMiddleware.php`, el cual recupera la sesión activa y verifica mediante `$user->hasRole('admin')` si posee los permisos necesarios. Si no los cumple, se detiene la ejecución inmediatamente.
*   **Cómo Validarlo:**
    1. Inicia sesión con una cuenta que tenga el rol de "Cliente" común.
    2. Escribe directamente en la barra de direcciones del navegador la URL: `http://localhost:8000/admin`.
    3. **Resultado Esperado:** El middleware interceptará la solicitud, bloqueará la carga de la vista y te mostrará una pantalla de error estructurada con el código de estado **HTTP 403 Forbidden** (Acceso Denegado).
    4. Ejecuta el test automatizado en tu consola:
       ```powershell
       php artisan test --filter=RoleMiddlewareTest
       ```

### Pregunta 5: *“¿Qué pasa si dos clientes intentan comprar el último producto en stock exactamente al mismo segundo? ¿El stock podría quedar en negativo?”*
*   **Funcionamiento Técnico:**  
    Implementamos transacciones ACID combinadas con bloqueo pesimista de filas en la base de datos. En `CheckoutController.php@callback`, la escritura de la compra se envuelve en `DB::transaction(function() { ... })`. Al leer el stock del producto, se aplica `lockForUpdate()`. Esto le indica a PostgreSQL que bloquee la fila del producto para la primera transacción. La segunda transacción concurrente es obligada a esperar a que la primera termine; para entonces, el stock ya habrá bajado a 0 y la segunda transacción se cancelará de forma segura al validar el inventario.
*   **Cómo Validarlo:**
    1. Abre el archivo de pruebas unitarias de concurrencia (`tests/Feature/Controllers/CheckoutControllerTest.php`).
    2. Ejecuta el test en la consola para simular la compra simultánea en milisegundos:
       ```powershell
       php artisan test --filter=CheckoutConcurrenciaTest
       ```
    3. **Resultado Esperado:** El test pasará con éxito, confirmando que la base de datos restó correctamente el stock al primer comprador y rechazó de forma segura al segundo sin dejar el inventario en valores negativos ni corromper los totales de la venta.

### Pregunta 6: *“¿Cómo registramos eventos del sistema de forma segura y inmutable sin consumir excesiva memoria RAM al leer los logs?”*
*   **Funcionamiento Técnico:**  
    El sistema cuenta con un modelo robusto de logs en `AuditLog.php`. El método estático `record()` recupera de manera automática los datos del request global (dirección IP y usuario en sesión) de forma inmutable. Para prevenir la saturación de memoria RAM al consultar los logs en el panel administrativo, `AuditLogController.php` implementa paginación (`paginate(30)`) en lugar de consultas de volumen completo (`get()`), limitando el tráfico PostgreSQL-FPM.
*   **Cómo Validarlo:**
    1. Realiza una acción crítica en el sistema (ej. inicia sesión o edita el stock de un producto).
    2. Entra al panel de administración en la sección `/admin/auditoria`.
    3. **Resultado Esperado:** Verás una tabla paginada con la lista exacta de las acciones, con fecha, hora, dirección IP y el usuario responsable. Al presionar los botones de navegación inferior, se cargarán bloques de 30 en 30 registros, demostrando que la base de datos no se sobrecarga de memoria RAM.

---
🔒 **Cuestionario y Guía de Validación Certificada** | PROYECTO MATCH - 2026

