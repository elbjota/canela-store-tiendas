# Canela Store: tienda online y punto de venta sobre una plataforma multi-tienda

**Comercio electrónico para MYPES · En producción en [tiendas.bjota.online](https://tiendas.bjota.online/canela-store) · 2026**

Es una plataforma tipo «Kyte» hecha a medida. Varias tiendas pueden usarla a la vez, cada una con sus propios datos. Ofrece:
- **Tienda online:** catálogo con variantes (por ejemplo, tonos de labial), carrito y compra **sin crear cuenta**.
- **Pagos:** por Yape o Plin. La dueña revisa la constancia y confirma el pago.
- **Mostrador (POS):** para vender en la tienda física.
- **Inventario** que baja solo con cada venta.
- **Reportes** para saber cuánto se ganó.

La primera clienta es **Canela Store**, una tienda de maquillaje y regalos de Sullana, Piura.

![Next.js](https://img.shields.io/badge/Next.js-15-000000?logo=nextdotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-API-000000?logo=express&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16%20%2B%20RLS-4169E1?logo=postgresql&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-ORM-2D3748?logo=prisma&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-protegido-2496ED?logo=docker&logoColor=white)
![Tests](https://img.shields.io/badge/pruebas-5%2C600%2B-15803D)

🛍️ **[Ver la tienda en vivo](https://tiendas.bjota.online/canela-store)** · 📄 **[Caso de estudio completo (PDF)](docs/caso-de-estudio.pdf)**

![Portada de la tienda](img/01-tienda-portada.png)

---

## El problema

Canela Store vendía por WhatsApp e Instagram:

- **Cada venta era una conversación:** fotos, precios y datos de envío a mano, y sin catálogo propio.
- **El stock no se actualizaba:** la app que usaba no descontaba el stock al entrar un pedido online.
- **Pagos por Yape difíciles de controlar:** había constancias sueltas en el chat.
- **Sin saber cuánto se ganaba:** no había costo por producto ni gastos registrados.

## La solución

| Módulo | Qué hace |
|---|---|
| **Tienda online** | Catálogo con variantes de tono, ofertas, buscador, carrito y compra en 3 pasos (envío, pago y datos). El cliente sigue su pedido sin crear cuenta. La tienda aparece en Google |
| **Pagos** | Efectivo, Yape, Plin, transferencia u "otros". El cliente adjunta su constancia y la dueña la confirma |
| **Pedidos** | Entran por la tienda, por venta manual o por proforma. Cada pedido pasa por estados (nuevo, pagado, enviado…). Se pueden editar, y si se cancelan, el stock vuelve |
| **Mostrador (POS)** | Venta en la tienda física. Permite pagar con dos métodos a la vez y calcula el vuelto |
| **Inventario** | Registra cada entrada y salida de productos, calcula el costo real de cada uno y avisa cuando algo se agota o está por acabarse |
| **Reportes** | Ventas, ganancia, margen, utilidad después de gastos, productos más vendidos y cobros por método. Se pueden bajar en Excel (CSV) |
| **Documentos** | Notas de venta y proformas en PDF |
| **Plataforma** | Varias tiendas en un solo sistema, con un panel general para darlas de alta. Roles para dueña y empleados, enlaces de acceso que solo sirven una vez, avisos al celular y libro de reclamaciones |

## La tienda (en producción)

| Ficha con variantes | Carrito |
|---|---|
| ![Producto](img/03-producto-variantes.png) | ![Carrito](img/04-carrito.png) |
| **Compra: envío** | **Compra: pago** |
| ![Envío](img/05-checkout-envio.png) | ![Pago](img/06-checkout-pago.png) |

## El panel de la dueña

![Mostrador POS](img/10-mostrador-pos.png)

| Reportes de utilidad | Inventario | Clientes |
|---|---|---|
| ![Reportes](img/14-reportes.png) | ![Inventario](img/12-inventario.png) | ![Clientes](img/13-clientes.png) |

## Cómo está armado

![Arquitectura](img/arquitectura.png)

- **La web** está hecha con Next.js. Es lo que ven el cliente y la dueña.
- **La API** está hecha con Express. Es el programa del servidor que recibe los pedidos y aplica las reglas.
- **La base de datos** es PostgreSQL. Uso Prisma para trabajar con ella desde el código.
- Todo corre en **Docker** (cada parte va en su propio contenedor) dentro de un **VPS** (un servidor Linux alquilado).

**Decisiones clave:**
- **Cada tienda ve solo sus datos.** Uso RLS: la propia base de datos separa los datos de cada tienda. Aunque la API tenga un error, una tienda no puede ver los pedidos de otra.
- **No se vende lo que no hay.** Si dos clientes compran la última unidad al mismo tiempo, solo uno lo logra.
- **Una persona confirma cada pago.** Las constancias se guardan en un lugar privado; nadie puede abrirlas con un enlace público.
- **Sesiones que se pueden cerrar a distancia.** Si un empleado pierde el celular, la dueña puede cerrar su sesión.
- **Contraseñas del servidor obligatorias.** El sistema no arranca si alguien dejó las claves de ejemplo sin cambiar.
- **Permisos mínimos.** Cada programa del servidor tiene solo los permisos que necesita, y la base de datos no es accesible desde internet.

## Seguridad probada

- Revisé la seguridad de la primera etapa y no encontré fallas graves.
- Probé ataques comunes en una copia del sistema:
  - meter código malicioso en un formulario (XSS), por ejemplo en el nombre de una tienda;
  - intentar burlar el límite de intentos haciéndose pasar por otra dirección IP;
  - revisar que todo vaya por HTTPS (conexión cifrada) y con las protecciones del navegador activas.
- Límite de intentos en el inicio de sesión y en los pedidos públicos, para frenar abusos.
- Las fotos subidas se revisan para confirmar que de verdad son imágenes, y se les borran datos ocultos como la ubicación del celular.
- Backups cifrados y probados: hice una prueba real de recuperar los datos desde un backup.
- CI: cada cambio se prueba automáticamente en GitHub.

## Lo que pidió la clienta

Tras ver la demo, la dueña pidió **6 mejoras** y **todas se implementaron**:
- modo mostrador (POS);
- compra según el tipo de envío;
- pago "a coordinar";
- edición de pedidos;
- nota de venta en PDF;
- precio por mayor.

## Calidad

- **2063 pruebas automáticas de la web**, todas pasan. Las verifiqué el 17/10/2026.
- **3622 pruebas automáticas de la API** en la última corrida completa registrada, más 7 omitidas.
- **83 migraciones** (cambios ordenados a la base de datos) y **366 commits**.

## Mi rol

Proyecto propio de principio a fin:
- conversé con la dueña de Canela para entender qué necesitaba;
- diseñé la base de datos y cómo se conectan las partes;
- desarrollé la API y la web con apoyo de asistentes de IA, y escribí las pruebas;
- protegí el servidor y publiqué el sistema;
- implementé los cambios que pidió la clienta.

---

> 🔒 **Repositorio vitrina:** no se publican el código fuente, los datos de clientes reales ni los detalles internos del servidor.
>
> Sobre las capturas:
> - La tienda pública se capturó en producción, sin enviar pedidos.
> - El panel viene de una copia local del sistema, con datos de demostración.
> - Las fotos de productos son de Wikimedia Commons, con licencia libre.

**Bryan Jeremy Espinoza Herrera** · BJOTA Systems · Perú
