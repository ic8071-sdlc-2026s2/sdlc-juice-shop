# Mapa de la app · OWASP Juice Shop

Este documento es su punto de partida para la fase 1. Describe qué hace la app, quién la usa, qué guarda y dónde vive cada parte del código, para que puedan escribir casos de abuso concretos. No trae debilidades ni respuestas: eso lo descubren ustedes con Cornucopia y las herramientas.

## Qué es

OWASP Juice Shop es una tienda en línea de jugos y productos. Es una aplicación web moderna: un backend en Node.js con TypeScript y Express que expone una API REST, y un frontend de página única (SPA) en Angular que la consume. Los datos viven en una base SQLite. Como tienda, deja: registrarse e iniciar sesión, buscar productos, verlos y reseñarlos, armar una canasta, aplicar cupones, pagar, ver el historial de pedidos, administrar direcciones y tarjetas, y usar un chat de soporte.

## Roles

| Rol | Qué puede hacer |
|:---|:---|
| Visitante anónimo | Navegar el catálogo, buscar productos, registrarse e iniciar sesión |
| Cliente (`customer`) | Todo lo anterior más canasta, pedidos, direcciones, tarjetas, reseñas y perfil |
| Cliente deluxe (`deluxe`) | Un cliente con beneficios de membresía |
| Contabilidad (`accounting`) | Funciones relacionadas con productos y facturación |
| Administración (`admin`) | Sección de administración: usuarios, pedidos, feedback y configuración |

La app arranca con cuentas de demostración ya sembradas (clientes, deluxe, contabilidad y administración). El proyecto **no publica las credenciales** de esas cuentas: encontrarlas es parte de los retos. Para trabajar la fase 1 regístrense su propia cuenta de cliente desde la pantalla de registro; es la vía soportada para entrar.

## Pantallas principales (frontend Angular)

- Catálogo y búsqueda de productos (página principal).
- Detalle de producto con reseñas.
- Registro e inicio de sesión; recuperación de contraseña por pregunta de seguridad.
- Canasta y flujo de compra (dirección, entrega, pago, cupón, monedero).
- Historial de pedidos y seguimiento.
- Perfil del usuario y foto de perfil (por archivo o por URL).
- Sección de administración.
- Chat de soporte y formulario de quejas.

## Endpoints principales (API REST)

La API tiene dos familias: `/api/<Modelo>` (CRUD generado sobre los modelos) y `/rest/<función>` (lógica a la medida). Los más relevantes para el laboratorio:

| Método y ruta | Para qué |
|:---|:---|
| `POST /rest/user/login` | Iniciar sesión; devuelve un token |
| `POST /api/Users` | Registrar un usuario |
| `GET /api/Users` | Listar usuarios (sección de administración) |
| `GET /rest/products/search?q=` | Buscar productos |
| `GET /api/Products` | Listado de productos |
| `GET/POST /rest/basket/{id}` y `/api/BasketItems` | Canasta del usuario |
| `POST /api/Feedbacks` | Dejar feedback |
| `POST /rest/user/reset-password` | Restablecer contraseña |
| `POST /api/Cards`, `/api/Addresses` | Medios de pago y direcciones |
| `POST /b2b/v2/orders` y carga de archivos | Pedidos B2B (incluye carga de XML) |
| `GET /rest/admin/application-version` | Versión de la app (se usa como ruta de salud) |
| `GET /rest/admin/application-configuration` | Configuración pública de la app |
| `GET /metrics` | Métricas estilo Prometheus |

La especificación Swagger está en `app/swagger.yml` y hay documentación interactiva servida por la propia app.

## Datos que guarda

En SQLite (vía Sequelize): usuarios, productos, canastas y sus ítems, pedidos, direcciones, tarjetas, feedback, quejas, preguntas y respuestas de seguridad, monederos. Las reseñas de productos viven en una base NoSQL en memoria (MarsDB). En el sistema de archivos hay una carpeta de archivos públicos (`ftp/`), cargas de usuarios (`uploads/`), bitácoras (`logs/`) y claves de ejemplo (`encryptionkeys/`).

Hay datos sensibles de verdad para el ejercicio: credenciales de usuarios, datos de tarjetas y direcciones, y tokens de sesión.

## Integraciones externas

- **Chatbot de soporte**: puede conectarse a una API compatible con OpenAI (configurable).
- **Imagen de perfil por URL**: el servidor puede descargar la imagen desde una URL que indica el usuario.
- **Funciones Web3/NFT**: hay módulos de monedero y token (librería `ethers`).
- **Métricas**: endpoint estilo Prometheus.

No depende de una base de datos externa ni de otros servicios para levantar: todo corre en un contenedor.

## Dónde vive cada parte en `app/`

| Carpeta o archivo | Contenido |
|:---|:---|
| `app/server.ts` | Arranque del servidor y montaje de rutas y middleware |
| `app/routes/` | Lógica de cada endpoint `/rest/...` (login, search, basket, order, etc.) |
| `app/models/` | Modelos de datos de Sequelize (User, Product, Basket, Order, Card, ...) |
| `app/lib/` | Utilidades: seguridad y tokens (`insecurity.ts`), logger, XML, utilidades varias |
| `app/data/` | Datos sembrados, i18n y material estático |
| `app/frontend/` | Aplicación Angular (pantallas, componentes, estilos) |
| `app/config/` | Archivos de configuración (`default.yml` y variantes) |
| `app/swagger.yml` | Especificación de la API |
| `app/Dockerfile` | Cómo se construye la imagen |
| `app/ORIGEN.md` | De dónde salió este código y su versión |

## Cómo levantarla

Desde la raíz del repositorio de equipo:

```bash
bash lab/app.sh rapido      # baja la imagen oficial y la levanta (lo más rápido)
bash lab/app.sh humo        # confirma que responde
bash lab/app.sh detener     # la apaga
```

La app queda en el puerto privado que indica `lab/app.env`. No lo haga público.

## Para la fase 1

1. Levante la app y recórrala como cliente: regístrese, busque, arme una canasta, deje una reseña, cambie su perfil.
2. Juegue una ronda de Cornucopia (edición Website App) en https://copi.owasp.org. Por cada carta pregúntese: ¿esto le puede pasar a Juice Shop? ¿en qué endpoint o pantalla de la tabla de arriba?
3. Cada «sí» con un lugar concreto es un caso de abuso. Escríbalo con el formulario «Caso de abuso» y enlácelo con el requerimiento ASVS que lo impediría.
