# scraper

Colección de **sitemaps para [Web Scraper.io](https://webscraper.io/) / [Cloud Web Scraper](https://cloud.webscraper.io/)** usados para extraer publicaciones de autopartes, motopartes y tractopartes desde **eBay** y **Amazon**, como parte del pipeline de publicación masiva en Mercado Libre.

Cada archivo `.js` de este repo **no es código ejecutable**: es un archivo de configuración JSON (sitemap) que se importa directamente en Web Scraper.io mediante *Sitemaps → Import sitemap*.

## 📦 Flujo general

```
Hoja "tiendas eBay" (Google Sheets)
        │  columna "resultado link"
        ▼
 sitemap de LISTADO (paginación por tienda)
        │  extrae: título, precio, vendedor, imágenes, compatibilidad, OEM
        ▼
   Dataset crudo (CSV/JSON exportado de Web Scraper.io)
        │
        ▼
 Prompts de IA (identificación de pieza, título ML, compatibilidad)
        │
        ▼
   Publicación en Mercado Libre (vía Integraly)
```

Existen dos variantes por marketplace: sitemaps de **listado por tienda** (recorren todas las publicaciones de una tienda con paginación) y sitemaps de **producto individual** (extraen un solo ítem a partir de una URL directa).

## 🗂️ Sitemaps incluidos

| Archivo | Marketplace | Modo | Qué extrae | Punto de entrada típico |
|---|---|---|---|---|
| `publicaciones+en+paginacion+ebay.js` | eBay | Listado de tienda + paginación | Título, tienda, precio, hasta 4 imágenes, compatibilidad, OEM | `resultado link` (Hoja "tiendas eBay") |
| `publicaciones+en+paginacion+links+ebay.js` | eBay | Solo recolección de links + paginación | Únicamente los links de cada publicación (sin abrir el detalle) | `resultado link` de varias tiendas a la vez |
| `publicaciones+en+lista+ebay.js` | eBay | Producto individual | Título, tienda, precio, imágenes, compatibilidad, OEM | URL directa de un ítem (`ebay.com/itm/...`) |
| `compatiblidades+en+paginacion+ebay.js` | eBay | Listado de tienda + tabla de compatibilidad | Marca, Chasis, Línea, Modelo, Litros (tabla "Notes" de compatibilidad de vehículos) | `resultado link` de la tienda |
| `compatibilidades+en+lista.js` | eBay | Producto individual | Misma tabla de compatibilidad (Marca, Chasis, Línea, Modelo, Litros) | URL directa de un ítem |
| `publicaciones+en+paginacion+amazon.js` | Amazon | Búsqueda + paginación automática | Opción, tienda, título, precio, hasta 4 imágenes, compatibilidad, OEM | URL de búsqueda de Amazon (`amazon.com/s?k=...`) |
| `publicaciones+en+lista+amazon.js` | Amazon | Producto individual | Igual que el anterior, pero por lista de URLs de producto directas | URL directa de producto (`amazon.com/dp/...`) |

## 🔍 Detalle por sitemap

### eBay — Publicaciones (paginación por tienda)
`publicaciones+en+paginacion+ebay.js`

- **Entrada:** URL de búsqueda de una tienda (`_ssn=` / `store_name=` / `sid=`), con `_pgn=` para páginas adicionales.
- **Paginación:** automática vía `next_page`, sigue el botón *"Go to next search page"* mientras no esté deshabilitado.
- **Campos:** `Opcion`, `Tienda`, `Titulo`, `Precio`, `Imagen 1-4`, `Compatiblidad`, `OEM`.
- **Uso:** este es el sitemap principal para volcar el catálogo completo de cada tienda listada en la hoja de tiendas eBay.

### eBay — Solo links (paginación, multi-tienda)
`publicaciones+en+paginacion+links+ebay.js`

- **Entrada:** lista de URLs `sid=` de **22 tiendas** precargadas (ej. `elite-suspension`, `hookedonsprocketsstore`, `maxpeedingrods-ca`, `turboengineparts-us`, `arkotractorparts`, etc.), cada una con el patrón `isRefine=true&_sop=15&_udlo=10&_udhi=500&_ipg=240`.
- **Salida:** únicamente los **links** de cada publicación (no abre el detalle del ítem) — pensado como paso previo rápido de recolección masiva de URLs antes de scrapear el detalle.

### eBay — Producto individual
`publicaciones+en+lista+ebay.js`

- **Entrada:** lista de URLs `ebay.com/itm/<id>` (ítems puntuales).
- **Campos:** los mismos que la versión con paginación, pero sin recorrer un listado — útil para reprocesar o verificar publicaciones específicas.

### eBay — Compatibilidad (paginación por tienda)
`compatiblidades+en+paginacion+ebay.js`

- **Entrada:** igual que el sitemap de publicaciones (URL de tienda).
- **Mecánica:** dentro de cada `link` de producto, hace clic en `button.pagination__next` (tipo `clickMore`) para expandir la tabla de compatibilidad de vehículos (identificada por contener la palabra `"Notes"`).
- **Campos extraídos por fila de la tabla:** `Modelo` (col. 1), `Marca` (col. 2), `Chasis` (col. 3), `Linea` (col. 4), `Litros` (col. 5).
- **Uso:** alimenta directamente el prompt de "Extracción de datos de eBay" (compatibilidad de vehículos e inferencia de litros).

### eBay — Compatibilidad (producto individual)
`compatibilidades+en+lista.js`

- Misma lógica que el anterior, pero a partir de una lista fija de URLs de producto (`ebay.com/itm/...`) en lugar de una tienda completa.

### Amazon — Publicaciones (búsqueda + paginación)
`publicaciones+en+paginacion+amazon.js`

- **Entrada:** URL de búsqueda de Amazon (ej. `amazon.com/s?k=scitoo&i=automotive...`).
- **Paginación:** automática vía `a.s-pagination-next`.
- **Navegación en 2 niveles:** primero recolecta `product-link` desde los resultados de búsqueda, luego entra a cada `product-page` (`body:has(div#ppd)`) para extraer el detalle.
- **Campos:** `Opcion`, `Tienda`, `Titulo`, `Precio` (con selector de respaldo `SelectorImage`/`a-offscreen`), `Imagen 1-4`, `Compatiblidad`, `OEM`.
- **Uso:** corresponde a las búsquedas por palabra clave de la hoja "Búsquedas de Amazon" (a-premium, air+suspension, axle, bearing, etc.).

### Amazon — Producto individual
`publicaciones+en+lista+amazon.js`

- **Entrada:** lista de URLs de producto de Amazon (`amazon.com/dp/<ASIN>` o con parámetros de búsqueda/sponsor incluidos).
- **Campos:** los mismos que el sitemap de búsqueda paginada, pero sin recorrer resultados — para productos puntuales.

## ⚙️ Notas técnicas

- Todos los sitemaps son formato **Web Scraper.io Sitemap JSON** (`_id`, `startUrl`, `selectors`), compatibles con importación directa en [cloud.webscraper.io](https://cloud.webscraper.io/).
- Los selectores de compatibilidad (`Compatiblidad`, `OEM`) solo devuelven datos si la publicación individual los expone en su página — por eso en la hoja de tiendas eBay se marca por tienda si tiene o no el filtro/dato de "compabilidad".
- El sitemap de compatibilidad usa `clickType: clickMore` con `delay: 2000` para expandir tablas dinámicas antes de leerlas — si eBay cambia el markup de la sección "Notes", este selector deberá actualizarse.
- Los sitemaps de Amazon incluyen selectores de respaldo (`Precio respaldo`, selector alterno con `:visible`) para tolerar variaciones del layout de precio entre productos.

## 🔗 Relación con el resto del proyecto

| Sitemap | Alimenta |
|---|---|
| `publicaciones+en+paginacion+ebay.js` / `+amazon.js` | Prompt 1.0-1.1 (Identificación de autoparte) y 2.0-2.5 (Título ML) |
| `compatiblidades+en+paginacion+ebay.js` / `compatibilidades+en+lista.js` | Prompt 3.0-3.2 (Extracción de compatibilidad de vehículos e inferencia de litros) |
| `publicaciones+en+paginacion+links+ebay.js` | Paso previo de recolección masiva de URLs, antes de correr los sitemaps de detalle |
