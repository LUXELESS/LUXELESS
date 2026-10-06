# Luxeless · Prompts para construir la web (Lovable + Supabase)

Cómo usarlo: copia cada bloque en el chat de Lovable, **uno por día o uno por mensaje**. Espera a que termine y revisa antes de pasar al siguiente. Antes del Prompt 0, adjunta en el chat el archivo `luxeless-icono.png` (o `.svg`) para que lo use como logo y favicon.

---

## PROMPT 0 · Base, diseño y estructura

```
Construye una plataforma web llamada **Luxeless**: catálogo de productos de belleza + buscador de dupes + comparador + comunidad + afiliación. Idioma de toda la interfaz: español (España). El usuario busca un producto de lujo, descubre alternativas más baratas (dupes), las compara, lee opiniones y compra en tiendas externas mediante enlaces de afiliación. No hay pagos ni inventario propio.

STACK: React + TypeScript + Tailwind + React Router, backend en Supabase (PostgreSQL, Auth, Storage, Row Level Security). Nada de datos hardcodeados: todo sale de la base de datos.

LOGO: usa la imagen adjunta como logo (header, footer) y como favicon. El wordmark es "Luxe" en negro y "less" en cursiva burdeos.

DISEÑO (premium, femenino, minimalista, editorial; inspiración Glossier, Rhode, Sephora, Space NK; no debe parecer tienda barata ni plantilla):
- Colores: burdeos #A0446F (acento principal, botones, enlaces), negro carbón #161616 (texto), blanco rosado #FDF9F9 (fondo), gris rosado #F4F0F0 (secciones alternas), rosa empolvado #D9B8B2 (detalles). Modo oscuro opcional.
- Tipografía: sans moderna tipo Inter con titulares muy grandes, peso semibold e interlineado ajustado. Una palabra clave de cada titular en cursiva burdeos (ej. "Belleza de lujo, *sin pagar de más*.").
- Etiquetas pequeñas en mayúsculas con letter-spacing amplio y una línea fina delante, color burdeos.
- Mucho espacio en blanco, tarjetas con bordes redondeados y sombras muy suaves, fotos de producto grandes, microanimaciones suaves (hover, fade-in).
- Mobile first; perfecto en móvil, tablet y escritorio.

HEADER fijo: logo, buscador con sugerencias instantáneas, Explorar, Marcas, Categorías, Comunidad, Favoritos, perfil / iniciar sesión. Menú hamburguesa en móvil. FOOTER con enlaces legales, newsletter y aviso de afiliación.

RUTAS (crea todas, aunque algunas queden en fase posterior): /, /buscar, /explorar, /marcas, /marcas/:slug, /productos/:slug, /dupes/:slug, /categorias/:slug, /comunidad, /mejores-dupes, /favoritos, /perfil/:username, /login, /registro, /blog, /blog/:slug, /sobre-nosotros, /contacto, /privacidad, /cookies, /terminos, /afiliacion, /admin. Página 404 con el texto "Este dupe se nos ha escapado." y botón "Volver a explorar".

En este primer paso: crea el layout, el sistema de diseño, el header/footer, las rutas y la home con el hero:
- Título: "Encuentra el dupe perfecto de tus productos favoritos"
- Subtítulo: "Descubre alternativas de belleza similares por mucho menos."
- Buscador grande: "Buscar producto, marca o categoría..." con chips de ejemplo (Dior Lip Glow, Charlotte Tilbury, Rare Beauty, Perfumes).
Las secciones de la home (se rellenarán con datos reales de la BD en el siguiente paso): Dupes en tendencia, Mejor valorados, Mayor ahorro, Últimos añadidos, Categorías (Maquillaje, Skincare, Perfumes, Cabello, Cuerpo, Uñas, Herramientas de belleza), Marcas populares, Comunidad.
```

---

## PROMPT 1 · Base de datos, catálogo, buscador, comparador, admin y afiliación (MVP)

```
Conecta Supabase y crea el esquema con migraciones, RLS y datos demo.

TABLAS: profiles (vinculada a auth.users: username, nombre, avatar_url, bio, role: user|admin|blocked), brands, categories, products, dupes, stores, prices, affiliate_links, reviews, comments, favorites, dupe_votes, dupe_suggestions, articles, reports, notifications, newsletter_subscribers, site_settings.

RELACIONES: una marca tiene muchos productos; un producto pertenece a una marca y a una categoría; un dupe relaciona dos productos (original_id, dupe_id) con similarity_pct, descripción, similitudes[], diferencias[], editorial_score, status (draft|published); un producto puede tener varios dupes y varios precios; cada precio pertenece a una tienda y puede tener affiliate_url; reviews, favorites y dupe_votes pertenecen a un usuario.

CAMPOS DE PRODUCTO: name, slug, brand, category, description, approx_price, image, gallery, ingredients, features, product_type, shade, finish, texture, quantity, SEO (seo_title, seo_description), is_demo.

SEGURIDAD: RLS en todas las tablas. Lectura pública solo de contenido publicado; escritura de usuarios solo sobre sus propios datos; escritura de catálogo solo para admin (rol en tabla aparte, nunca en el cliente). Ninguna clave privada en el frontend.

DATOS DEMO (todos con is_demo = true y una etiqueta visible "DEMO" en la interfaz): 10 marcas (Charlotte Tilbury, Dior, Chanel, Rare Beauty, Fenty Beauty, NARS, MAC, Sol de Janeiro, Olaplex, Dyson), alternativas de marcas asequibles, 30 productos, 20 relaciones de dupe, categorías, tiendas (Primor, Druni, Amazon, Notino), precios, 3 artículos y algunas opiniones. Aviso: los precios, porcentajes de similitud, ingredientes y opiniones demo son ficticios y no deben presentarse como reales.

FUNCIONES QUE DEBEN FUNCIONAR DE VERDAD:
1. Buscador con sugerencias instantáneas (productos, marcas, categorías, dupes) y página /buscar con ordenación (más populares, mejor valorados, mayor parecido, mayor ahorro, precio más bajo, más recientes) y filtros (categoría, marca, precio, valoración, % similitud, tipo).
2. /marcas con filtro alfabético y /marcas/:slug (descripción, productos, dupes disponibles, comparaciones recientes).
3. /productos/:slug con galería, datos, precio "Desde X €", tiendas, botón "Encontrar dupes", dupes asociados y productos relacionados. Si no hay dupe: "Todavía no tenemos un dupe para este producto." con botones "Avísame cuando encontremos uno" y "Proponer un dupe".
4. /dupes/:slug: comparación lado a lado (precio, ahorro, ingredientes, textura, acabado, duración, color, packaging, cantidad, resultado), similitudes, diferencias, % de similitud, % de usuarios que dicen que es buen dupe, botones Comprar original / Comprar dupe / Favoritos / Compartir (WhatsApp, X, Facebook, copiar enlace).
5. Tarjeta de comparación reutilizable: imagen y nombre del premium, ↓ imagen y nombre del dupe, precios, "92 % similar", "Ahorras 33 €", valoración, botón "Ver comparación".
6. Afiliación: cada tienda con precio, URL y enlace de afiliado; se abre en pestaña nueva con rel="sponsored noopener". Mostrar en las páginas con enlaces: "Algunos enlaces de esta página son enlaces de afiliación. Luxeless puede recibir una comisión si realizas una compra, sin coste adicional para ti."
7. Home y /mejores-dupes alimentadas por la BD con los filtros indicados.

PANEL /admin (solo rol admin, ruta protegida): CRUD completo de productos, marcas, categorías, dupes (original, dupe, % similitud, similitudes, diferencias, puntuación editorial, publicado/borrador), tiendas, precios, enlaces de afiliación, subida de imágenes a Storage, usuarios y contenido de la home. Todo editable sin tocar código.

El primer administrador se crea de forma segura: no incluyas contraseñas en el código; explica cómo asignar el rol admin a mi usuario desde Supabase.
```

---

## PROMPT 2 · Usuarios y comunidad

```
Implementa usuarios y comunidad con funcionamiento real:
- Registro y login con email, recuperar contraseña y Google login si es posible. Rutas privadas protegidas.
- Perfil /perfil/:username: nombre, avatar, bio, opiniones, favoritos, dupes guardados, aportaciones. Edición de perfil con subida de avatar.
- Opiniones con estrellas 1–5 y subvaloraciones (parecido, calidad, relación calidad/precio, duración, resultado), foto opcional, medias calculadas automáticamente. Comentarios y respuestas, marcar como "útil", seguir discusiones.
- Votación en cada comparación: "¿Crees que realmente es un dupe?" Sí / No (un voto por usuario, modificable) y mostrar "91 % de la comunidad dice que sí".
- Favoritos de productos, comparaciones y dupes + página /favoritos.
- "Proponer un dupe": formulario (producto original, alternativo, marca, URL, precio, explicación, imagen opcional) que queda pendiente de aprobación en el admin.
- Moderación en /admin: eliminar comentarios, aprobar o rechazar propuestas, bloquear usuarios, moderar fotos, gestionar reportes. Botón "Reportar" en opiniones y comentarios.
- Validación de formularios y rate limiting básico (por ejemplo, un voto/opinión por usuario y producto, límite de comentarios por minuto).
- Notificaciones en la app: nuevo dupe de un favorito, respuesta a un comentario, propuesta aprobada.
- /comunidad con las opiniones más recientes y más útiles.
```

---

## PROMPT 3 · Blog, SEO, legal y extras

```
Completa la plataforma:
- Blog /blog y /blog/:slug gestionado desde el admin (título, slug, contenido, imagen, SEO, publicado/borrador).
- SEO: title, meta description, canonical y Open Graph editables por producto, marca, dupe y artículo; datos estructurados (Product, Review/AggregateRating, Article, BreadcrumbList); sitemap.xml y robots.txt; URLs limpias. Si el stack no permite renderizado en servidor, dilo claramente y propón la mejor alternativa.
- SEO programático: páginas generadas desde la BD como "Mejores dupes de Dior", "Alternativas a Rare Beauty", "Dupes baratos de perfumes", usando /marcas/:slug y /mejores-dupes/:categoria.
- Páginas legales con placeholders editables (no inventes datos de empresa): aviso legal, privacidad, cookies, términos, política de afiliación. Banner de consentimiento de cookies que bloquee la analítica hasta aceptar.
- Analítica configurable desde el admin (ID de Google Analytics, Search Console, Meta Pixel y TikTok Pixel opcionales).
- Newsletter "Recibe los mejores dupes cada semana": guarda emails en la BD, evita duplicados, deja preparada la integración futura con una herramienta de email marketing.
- Rendimiento: imágenes con lazy loading y tamaños adecuados, caché de consultas, índices en la BD.
- Si algo no se puede implementar de verdad, indícalo claramente en vez de simularlo.
```

---

## Cuando termines

1. Pide a Lovable: "Genera un README con instalación, variables de entorno de ejemplo, esquema de BD, cómo cambiar el branding, gestionar productos y afiliados, y desplegar con dominio propio."
2. Sustituye los datos DEMO por reales desde `/admin`.
3. Antes de publicar, revisa los textos legales con un profesional.
