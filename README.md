# marketplace-scraper-sitemaps

🇪🇸 [Español](#español) | 🇬🇧 [English](#english)

---

## Español

Colección de **sitemaps para [Web Scraper.io](https://webscraper.io/) / [Cloud Web Scraper](https://cloud.webscraper.io/)** para extraer publicaciones de productos desde **eBay** y **Amazon**. No está atado a una categoría específica — sirve para sourcing masivo de cualquier tipo de producto (autopartes, electrónica, ropa, hogar, etc.), reemplazando solo la URL de entrada según lo que se quiera extraer.

Cada archivo `.js` de este repo **no es código ejecutable**: es un archivo de configuración JSON (sitemap) que se importa directamente en Web Scraper.io mediante *Sitemaps → Import sitemap*.

### 📦 Flujo general

```
Lista de tiendas / búsquedas (Google Sheets u otra fuente)
        │  URL de tienda o de búsqueda
        ▼
 sitemap de LISTADO (paginación)
        │  extrae: título, precio, vendedor, imágenes, compatibilidad, specs
        ▼
   Dataset crudo (CSV/JSON exportado de Web Scraper.io)
        │
        ▼
   Procesamiento (prompts de IA, normalización, validación)
        │
        ▼
   Publicación en el marketplace destino
```

Existen dos variantes por origen: sitemaps de **listado por tienda/búsqueda** (recorren todas las publicaciones con paginación) y sitemaps de **producto individual** (extraen un solo ítem a partir de una URL directa).

### 🗂️ Sitemaps incluidos

| Archivo | Origen | Modo | Qué extrae | Punto de entrada típico |
|---|---|---|---|---|
| `publicaciones+en+paginacion+ebay.js` | eBay | Listado de tienda + paginación | Título, tienda, precio, hasta 4 imágenes, compatibilidad, specs | URL de búsqueda de tienda |
| `publicaciones+en+paginacion+links+ebay.js` | eBay | Solo recolección de links + paginación | Únicamente los links de cada publicación | URL de búsqueda (múltiples tiendas) |
| `publicaciones+en+lista+ebay.js` | eBay | Producto individual | Título, tienda, precio, imágenes, compatibilidad, specs | URL directa de un ítem |
| `compatiblidades+en+paginacion+ebay.js` | eBay | Listado + tabla de compatibilidad | Marca, Chasis, Línea, Modelo, Litros (tabla de compatibilidad de vehículos) | URL de búsqueda de tienda |
| `compatibilidades+en+lista.js` | eBay | Producto individual | Misma tabla de compatibilidad | URL directa de un ítem |
| `publicaciones+en+paginacion+amazon.js` | Amazon | Búsqueda + paginación automática | Opción, tienda, título, precio, hasta 4 imágenes, compatibilidad, specs | URL de búsqueda |
| `publicaciones+en+lista+amazon.js` | Amazon | Producto individual | Igual al anterior, pero por lista de URLs directas | URL directa de producto |

### 🔍 Detalle por sitemap

**eBay — Publicaciones (paginación por tienda)** — `publicaciones+en+paginacion+ebay.js`
Paginación automática vía el botón "Go to next search page". Campos: Opcion, Tienda, Titulo, Precio, Imagen 1-4, Compatiblidad, OEM.

**eBay — Solo links (multi-tienda)** — `publicaciones+en+paginacion+links+ebay.js`
Recolecta únicamente URLs de publicaciones (sin abrir el detalle) — paso previo rápido antes de scrapear el detalle.

**eBay — Producto individual** — `publicaciones+en+lista+ebay.js`
Mismos campos que el de paginación, pero para una lista fija de URLs puntuales.

**eBay — Compatibilidad (paginación)** — `compatiblidades+en+paginacion+ebay.js`
Hace clic para expandir la tabla de compatibilidad ("Notes") y extrae Modelo, Marca, Chasis, Línea, Litros por fila.

**eBay — Compatibilidad (producto individual)** — `compatibilidades+en+lista.js`
Misma lógica, para una lista fija de URLs de producto.

**Amazon — Publicaciones (búsqueda + paginación)** — `publicaciones+en+paginacion+amazon.js`
Navegación en 2 niveles: recolecta links de resultados y luego entra a cada página de producto. Incluye selectores de respaldo para precio.

**Amazon — Producto individual** — `publicaciones+en+lista+amazon.js`
Mismos campos, para una lista de URLs de producto directas.

### ⚙️ Notas técnicas

- Formato **Web Scraper.io Sitemap JSON** (`_id`, `startUrl`, `selectors`), compatible con importación directa en cloud.webscraper.io.
- Los campos de compatibilidad/specs solo devuelven datos si la publicación individual los expone.
- El sitemap de compatibilidad usa `clickType: clickMore` con `delay: 2000` para expandir tablas dinámicas — si el marketplace cambia el markup, el selector deberá actualizarse.
- Los sitemaps de Amazon incluyen selectores de respaldo para tolerar variaciones de layout entre productos.
- Para usar en una categoría distinta a la original, solo se necesita cambiar la `startUrl` por la tienda o búsqueda deseada; los selectores son agnósticos a la categoría del producto.

---

## English

A collection of **sitemaps for [Web Scraper.io](https://webscraper.io/) / [Cloud Web Scraper](https://cloud.webscraper.io/)** to extract product listings from **eBay** and **Amazon**. Not tied to a specific product category — it works for bulk sourcing of any product type (auto parts, electronics, apparel, home goods, etc.); just swap the input URL for whatever you want to scrape.

Each `.js` file in this repo is **not executable code**: it's a JSON configuration file (sitemap) that gets imported directly into Web Scraper.io via *Sitemaps → Import sitemap*.

### 📦 General flow

```
Store/search list (Google Sheets or another source)
        │  store or search URL
        ▼
 LISTING sitemap (pagination)
        │  extracts: title, price, seller, images, compatibility, specs
        ▼
   Raw dataset (CSV/JSON exported from Web Scraper.io)
        │
        ▼
   Processing (AI prompts, normalization, validation)
        │
        ▼
   Publishing to the target marketplace
```

There are two variants per source: **store/search listing** sitemaps (crawl every listing with pagination) and **single product** sitemaps (extract one item from a direct URL).

### 🗂️ Included sitemaps

| File | Source | Mode | What it extracts | Typical entry point |
|---|---|---|---|---|
| `publicaciones+en+paginacion+ebay.js` | eBay | Store listing + pagination | Title, seller, price, up to 4 images, compatibility, specs | Store search URL |
| `publicaciones+en+paginacion+links+ebay.js` | eBay | Link collection only + pagination | Just the listing links | Search URL (multiple stores) |
| `publicaciones+en+lista+ebay.js` | eBay | Single product | Title, seller, price, images, compatibility, specs | Direct item URL |
| `compatiblidades+en+paginacion+ebay.js` | eBay | Listing + compatibility table | Make, Chassis, Line, Model, Liters (vehicle compatibility table) | Store search URL |
| `compatibilidades+en+lista.js` | eBay | Single product | Same compatibility table | Direct item URL |
| `publicaciones+en+paginacion+amazon.js` | Amazon | Search + auto pagination | Option, seller, title, price, up to 4 images, compatibility, specs | Search URL |
| `publicaciones+en+lista+amazon.js` | Amazon | Single product | Same as above, for a list of direct URLs | Direct product URL |

### 🔍 Sitemap details

**eBay — Listings (store pagination)** — `publicaciones+en+paginacion+ebay.js`
Auto-pagination via the "Go to next search page" button. Fields: Opcion, Tienda, Titulo, Precio, Imagen 1-4, Compatiblidad, OEM.

**eBay — Links only (multi-store)** — `publicaciones+en+paginacion+links+ebay.js`
Collects only listing URLs (doesn't open item detail) — a fast pre-step before scraping details.

**eBay — Single product** — `publicaciones+en+lista+ebay.js`
Same fields as the pagination version, for a fixed list of specific URLs.

**eBay — Compatibility (pagination)** — `compatiblidades+en+paginacion+ebay.js`
Clicks to expand the compatibility ("Notes") table and extracts Model, Make, Chassis, Line, Liters per row.

**eBay — Compatibility (single product)** — `compatibilidades+en+lista.js`
Same logic, for a fixed list of product URLs.

**Amazon — Listings (search + pagination)** — `publicaciones+en+paginacion+amazon.js`
Two-level navigation: collects result links, then visits each product page. Includes fallback selectors for price.

**Amazon — Single product** — `publicaciones+en+lista+amazon.js`
Same fields, for a list of direct product URLs.

### ⚙️ Technical notes

- **Web Scraper.io Sitemap JSON** format (`_id`, `startUrl`, `selectors`), ready for direct import into cloud.webscraper.io.
- Compatibility/specs fields only return data if the individual listing exposes them.
- The compatibility sitemap uses `clickType: clickMore` with a `2000ms` delay to expand dynamic tables — if the marketplace changes its markup, the selector will need updating.
- The Amazon sitemaps include fallback selectors to tolerate layout variations between products.
- To use this for a different category than the original one, just swap `startUrl` for the desired store or search; the selectors are category-agnostic.
