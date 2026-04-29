# 👑 MANUAL DEL ADMINISTRADOR - MATCH E-COMMERCE
> **Guía oficial para la gestión de catálogo, ventas y monitoreo del sistema.**

Bienvenido al panel de control de Match. Desde aquí podrás gestionar todo el ciclo de vida de tu tienda.

---

## 1. Acceso al Panel de Control

Para acceder al panel administrativo, debes iniciar sesión con una cuenta de rol `Admin`.
1.  Ingresa a `http://localhost:8000/login`.
2.  Una vez autenticado, haz clic en el botón **⚙️ Panel Admin** en el menú superior.

> 📸 CAPTURA SUGERIDA: [Captura la pantalla del Dashboard principal mostrando las tarjetas de estadísticas (Total Ventas, Productos, Usuarios) en Dark Mode].

---

## 2. Gestión del Catálogo (Productos y Categorías)

### 2.1 Categorías
Organiza tus productos por tipo (ej. "Camisas", "Calzado").
*   Ve a la pestaña **Categorías**.
*   Puedes crear nuevas categorías con un nombre y una descripción para el SEO.

### 2.2 Productos
*   **Stock:** Es fundamental mantener el stock actualizado. El sistema bloquea compras si el stock llega a cero.
*   **Imágenes:** Puedes subir una imagen principal y una galería de fotos.
*   **IA Ready:** Asegúrate de que las fotos de los productos tengan un fondo limpio para que la IA del probador virtual funcione mejor.

> 📸 CAPTURA SUGERIDA: [Captura el formulario de edición de un producto, destacando los campos de Precio, Stock y la sección de imágenes subidas].

---

## 3. Gestión de Pedidos (Ventas)

En la sección **💰 Pedidos**, verás todas las transacciones realizadas por los clientes.
*   **Estados:** Puedes cambiar el estado de `Pendiente` a `Enviado` o `Entregado`.
*   **Detalle:** Haz clic en una orden para ver qué productos compró el usuario y cuánto pagó exactamente.

---

## 4. Auditoría y Seguridad

Este es el módulo de "Caja Negra" de tu tienda. En **🛡️ Auditoría Global**, el sistema registra automáticamente:
1.  **Ventas:** Cada vez que alguien paga.
2.  **Inventario:** Quién cambió el stock de un producto y cuándo.
3.  **IA:** Si la generación de imágenes fue exitosa o falló.

> 📸 CAPTURA SUGERIDA: [Captura la tabla de Auditoría Global mostrando los badges de colores (Verde para transacciones, Morado para IA)].

---

## 5. Monitoreo Avanzado (Herramientas de Ingeniería)

### ⚡ Laravel Pulse
Dashboard en tiempo real para ver si el servidor está lento o si hay demasiados usuarios conectados.
*   **Uso:** Sirve para detectar picos de carga durante ofertas o Black Friday.

### 🔭 Laravel Telescope
Herramienta de depuración profunda.
*   **Uso:** Si un cliente reporta que la IA le dio un error, aquí puedes ver exactamente qué respondió el servidor de Google Gemini y por qué falló el proceso.

> 📸 CAPTURA SUGERIDA: [Captura la pantalla de Laravel Pulse mostrando las gráficas de uso de CPU y memoria del contenedor].

---
**Recuerda cerrar sesión cuando termines de usar el panel administrativo.**
