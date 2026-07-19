# Capri Viajes

Sitio web de la agencia de viajes Capri Viajes (Rosario, Santa Fe). Es un sitio estático (HTML/CSS/JS sin frameworks ni build tools) con un panel de administración conectado a una base de datos en la nube (Supabase), publicado en GitHub Pages.

## Archivos principales

- [index.html](index.html) — sitio público.
- [admin.html](admin.html) — panel de administración (requiere login).
- [supabase-config.js](supabase-config.js) — inicializa el cliente de Supabase (URL + clave pública `anon`). Se referencia desde ambos HTML.
- [favicon.svg](favicon.svg) — ícono del sitio.
- [robots.txt](robots.txt) / [sitemap.xml](sitemap.xml) — SEO.
- [404.html](404.html) — página de error personalizada (la sirve GitHub Pages automáticamente).

No hay `package.json` ni proceso de build: los archivos se editan y se suben tal cual.

## Arquitectura

- **Hosting**: GitHub Pages, rama `main`, carpeta raíz. URL: `https://ezequiellingua1996.github.io/CapriViajes/`.
- **Backend/datos**: [Supabase](https://supabase.com) (Postgres + Auth), proyecto `capriviajes`.
  - Tabla `site_data`: guarda todo el contenido editable como JSON en dos filas (`id`):
    - `capri_data`: paquetes, grupales, quince (fiestas de 15), testimonios, destinos, team, blog, asistencias, servicios.
    - `capri_settings`: moneda por defecto y tipo de cambio (no se usa para conversión automática — ver "Moneda" más abajo).
  - Tabla `page_events`: registro liviano de eventos (`view`, `whatsapp_click`, `blog_view`) usado por el panel de Estadísticas del admin.
  - **RLS (Row Level Security)**: lectura pública en ambas tablas; escritura solo para usuarios autenticados (`auth.role() = 'authenticated'`) en `site_data`, e inserción pública / lectura autenticada en `page_events`.
  - **Auth**: Supabase Auth (email + contraseña). El admin se loguea con un usuario real de Supabase, no con credenciales hardcodeadas.
- **Analytics**: Google Analytics 4 (gtag.js) integrado en `index.html`, con pageviews virtuales manuales para cada "página" de la SPA (paquetes, grupales, detalle de cada paquete, notas de blog, etc.), ya que todo el sitio es una sola página HTML con navegación por JavaScript (no hay rutas reales de servidor).
- **Repo Git**: remoto configurado por **SSH** (`git@github.com:ezequiellingua1996/CapriViajes.git`), no HTTPS — esto evita un bloqueo de Application Control de Windows sobre `libcurl-4.dll` que impide el transporte HTTPS de git en esta máquina.

## Secciones del sitio público (index.html)

Todo vive en un único HTML; la navegación entre "páginas" es JavaScript (`showPage(id)`), no hay recarga real, salvo por los pageviews virtuales que se mandan a GA4.

- **Inicio**: hero, destinos destacados, "¿Por qué contratar con nosotros?" (editable), contadores, testimonios, banner de WhatsApp.
- **Paquetes**: viajes individuales, filtro Nacional/Internacional, buscador.
- **Grupales**: viajes grupales, mismo filtro. Soporta múltiples países de destino, itinerario de vuelo, pagos en cuotas, ciudades del viaje y opciones de hotel agrupadas por ciudad (igual que Paquetes).
- **Quinceañeras**: paquetes para fiestas de 15, con estilo visual propio (rosa/violeta/dorado). Mismos campos que Grupales, sin filtro de categoría.
- **Asistencia al viajero**: empresas de asistencia al viajero con las que trabaja la agencia (logo, descripción, botón de consulta por WhatsApp).
- **¿Quiénes somos?**: equipo, misión, estadísticas.
- **Contacto**: datos de contacto.
- **Blog**: implementado y funcional (grilla + detalle de nota, con URL compartible `?post=slug`), pero **oculto del menú por pedido del cliente** — el código sigue activo, solo hay que descomentar los links de nav (buscar "Blog oculto temporalmente" en `index.html`) para reactivarlo.
- **Detalle de paquete/grupal/quince**: hero, itinerario de vuelo, selección de hotel (si hay hoteles cargados), resumen por ciudad, incluye/no incluye, excursiones, itinerario día a día, precio y cuotas, botón de WhatsApp.
- **Buscador global** (ícono de lupa en el nav): busca por título/destino en paquetes y grupales.
- **Botón flotante de WhatsApp**: visible en todas las páginas excepto en el detalle de paquete (que ya tiene su propio CTA).

## Panel de administración (admin.html)

Login con email + contraseña (Supabase Auth). Secciones:

- **Paquetes**, **Grupales**, **Quinceañeras**: alta/edición/eliminación/duplicado. Cada uno con imagen (+ vista previa), fechas (con selector de calendario), precio, moneda, pagos en cuotas, itinerario de vuelo (con fecha por tramo), ciudades del viaje, hoteles por ciudad, incluye/no incluye, excursiones, itinerario día a día.
- **Testimonios**: nombre, viaje, texto, estrellas, foto.
- **Destinos inicio**: tarjetas de destinos destacados en el home (país, ciudad, precio, tag, imagen, destacado grande).
- **Servicios (home)**: los ítems de "¿Por qué contratar con nosotros?" (ícono emoji, título, texto).
- **Blog**: alta/edición de notas (oculto en el sitio público, ver arriba).
- **Asistencia al viajero**: empresas de asistencia (nombre, logo, descripción, teléfono, web).
- **Estadísticas**: totales y ranking de vistas de paquetes / clicks a WhatsApp / vistas de blog, leídos desde la tabla `page_events`.
- **Datos de contacto**: teléfono, email, dirección, redes, moneda por defecto.
- **Quiénes somos**: equipo (nombres, roles, fotos).
- **Seguridad**: cambio de contraseña del usuario logueado (vía Supabase Auth).

## Cómo trabajar en este proyecto

1. Editar `index.html` / `admin.html` directamente (no hay paso de build).
2. Probar abriendo los archivos en un navegador.
3. `git add` + `git commit` + `git push origin main`.
4. GitHub Pages redeploya solo en 1-2 minutos.

### Configurar Supabase desde cero (si hay que recrear el proyecto)

1. Crear proyecto en supabase.com.
2. Correr en el SQL Editor la creación de la tabla `site_data` (id text PK, value jsonb, updated_at) con policies de lectura pública / escritura autenticada, y la tabla `page_events` (id bigserial, type text, package_id text, package_title text, created_at) con policy de inserción pública y lectura autenticada.
3. Crear un usuario en Authentication → Users (ese es el login del admin).
4. Copiar Project URL y **anon public key** (nunca la `service_role`) a [supabase-config.js](supabase-config.js).

## Notas y decisiones a tener en cuenta

- **Moneda**: el selector de USD/$ en Paquetes es solo visual — cada paquete siempre muestra el precio en la moneda que tiene guardada (`p.moneda`). A pedido del cliente, **no hay conversión automática** por tipo de cambio (es muy variable y no se quiere depender de mantenerlo actualizado).
- **Grupales/Quinceañeras y Paquetes comparten funciones de admin** (vuelos, pagos, hoteles, ciudades) parametrizadas por `listId` para evitar IDs duplicados en el DOM — al tocar esas funciones genéricas, verificar que no se rompa ninguno de los tres formularios.
- **Datos legado**: campos agregados después de la migración a Supabase (categoría de grupales, ciudad de hoteles, etc.) pueden faltar en registros viejos; el código tiene fallbacks para no ocultar contenido existente, pero conviene revisar/resguardar esos registros desde el admin.
- Reseñas de Google: se cargan a mano como Testimonios (copiando el texto real desde la ficha de Google Negocio) — no hay integración automática con la API de Google.
