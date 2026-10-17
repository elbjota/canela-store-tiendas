# Canela Store: tienda online y punto de venta sobre una plataforma multi-tienda

**Comercio electrónico para MYPES · En producción en [tiendas.bjota.online](https://tiendas.bjota.online/canela-store) · 2026**

Es una plataforma tipo «Kyte» hecha a medida. Ofrece:
- **Tienda online:** catálogo con variantes, carrito y checkout **sin crear cuenta**.
- **Pagos:** por Yape o Plin, con confirmación de la dueña.
- **Mostrador (POS)** para vender en la tienda física.
- **Inventario** que baja solo con cada venta.
- **Reportes** de utilidad.

La primera clienta es **Canela Store**, una tienda de maquillaje y regalos de Sullana, Piura.

![Next.js](https://img.shields.io/badge/Next.js-15-000000?logo=nextdotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express-API-000000?logo=express&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16%20%2B%20RLS-4169E1?logo=postgresql&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-ORM-2D3748?logo=prisma&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-endurecido-2496ED?logo=docker&logoColor=white)
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
| **Tienda online** | Catálogo con variantes de tono, ofertas, buscador, carrito y checkout en 3 pasos (envío, pago y datos); seguimiento del pedido sin cuenta; SEO y sitemap |
| **Pagos** | Efectivo, Yape, Plin, transferencia u "otros". El cliente adjunta su constancia y la dueña la confirma |
| **Pedidos** | Por checkout, venta manual o proforma; máquina de estados; edición; cancelación que repone el stock |
| **Mostrador (POS)** | Venta en la tienda física con pago dividido y vuelto |
| **Inventario** | Kardex con costo promedio ponderado y alertas de agotado y "por acabarse" |
| **Reportes** | Ventas, ganancia, margen, utilidad neta tras gastos, más vendidos y cobros por método; exportación a CSV |
| **Documentos** | Notas de venta y proformas en PDF |
| **Plataforma** | Multi-tienda con panel master, roles, enlaces de acceso de un solo uso, avisos push y libro de reclamaciones |

## La tienda (en producción)

| Ficha con variantes | Carrito |
|---|---|
| ![Producto](img/03-producto-variantes.png) | ![Carrito](img/04-carrito.png) |
| **Checkout: envío** | **Checkout: pago** |
| ![Envío](img/05-checkout-envio.png) | ![Pago](img/06-checkout-pago.png) |

## El panel de la dueña

![Mostrador POS](img/10-mostrador-pos.png)

| Reportes de utilidad | Inventario | Clientes |
|---|---|---|
| ![Reportes](img/14-reportes.png) | ![Inventario](img/12-inventario.png) | ![Clientes](img/13-clientes.png) |

## Arquitectura

![Arquitectura](img/arquitectura.png)

**Decisiones clave:**
- **RLS forzado y un usuario sin privilegios:** cada fila lleva `storeId`, y PostgreSQL filtra con `FORCE ROW LEVEL SECURITY`. La app entra como `app_user` sin `BYPASSRLS`.
- **Pedidos transaccionales** con `SELECT … FOR UPDATE`: dos clientes no pueden comprar la última unidad a la vez.
- **Confirmación humana del pago:** las constancias quedan en almacenamiento privado, sin URL pública.
- **Sesiones revocables:** JWT con `tokenVersion` y enlaces de acceso de un solo uso.
- **Secretos:** el servidor no arranca si detecta secretos de plantilla sin cambiar.
- **Contenedores endurecidos:** `cap_drop: ALL`, `no-new-privileges`, solo lectura y la base de datos en una red interna.

## Seguridad probada

- Auditoría de la Fase 1 sin hallazgos críticos ni altos.
- Ataques de prueba en un entorno réplica:
  - XSS con nombres de tienda `<script>alert(1)</script>`;
  - `X-Forwarded-For` falsificado contra el rate limiting;
  - verificación de HTTPS, HSTS y CSP.
- Rate limiting en el login y en los pedidos públicos; imágenes validadas por su tipo real y sin metadatos EXIF.
- Backups cifrados con restic, con un ensayo de restauración.
- CI en GitHub Actions: pruebas, tipos y build.

## Lo que pidió la clienta

Tras ver la demo, la dueña pidió **6 mejoras** y **todas se implementaron**:
- modo mostrador (POS);
- checkout según el tipo de envío;
- pago "a coordinar";
- edición de pedidos;
- nota de venta en PDF;
- precio por mayor.

## Calidad

- **2063 pruebas de la web** (Vitest), todas pasan; las verifiqué el 17/10/2026.
- **3622 pruebas de la API** (Vitest + Supertest) en la última corrida completa registrada, más 7 omitidas.
- **83 migraciones** de Prisma y **366 commits**.

## Mi rol

Proyecto propio de principio a fin:
- levanté los requisitos con la dueña de Canela;
- diseñé la base de datos y la arquitectura;
- desarrollé la API y la web con apoyo de asistentes de IA, y escribí las pruebas;
- endurecí y desplegué la infraestructura;
- implementé los cambios que pidió la clienta.

---

> 🔒 **Repositorio vitrina:** no se publican el código fuente, los datos de clientes reales ni los detalles internos del servidor.
>
> Sobre las capturas:
> - La tienda pública se capturó en producción, sin enviar pedidos.
> - El panel viene de la base de desarrollo local, con datos de demostración.
> - Las fotos de productos son de Wikimedia Commons, con licencia libre.

**Bryan Jeremy Espinoza Herrera** · BJOTA Systems · Perú
