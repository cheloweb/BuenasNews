# Buenas Noticias — portal de noticias positivas autoactualizable

Sitio estático generado con Python (solo librería estándar, sin dependencias).
Cada 3 horas un workflow de GitHub Actions:

1. Lee los feeds RSS de `config.json` (fuentes especializadas en noticias positivas).
2. Descarta notas con palabras negativas (`blocklist`).
3. Si hay `ANTHROPIC_API_KEY`, Claude valida que la nota sea positiva, reescribe título y resumen en español (texto original, no copiado) y asigna categoría.
4. Acumula las notas en `data/articles.json` y regenera `public/` (portada, una página por nota, categorías, RSS, sitemap).
5. Publica en GitHub Pages.

Cada nota muestra un resumen corto y enlaza a la fuente original (no se copian artículos completos).

## Probar en local

```bash
python build.py              # descarga, cura y genera public/
python build.py --no-fetch   # solo regenera con lo que ya hay en data/
python -m http.server -d public 8000
```

## Publicar (GitHub Pages)

1. Sube esta carpeta a un repo de GitHub (rama `main`).
2. Settings → Pages → Source: **GitHub Actions**.
3. Settings → Secrets and variables → Actions:
   - Secret `ANTHROPIC_API_KEY` (opcional pero recomendado: mejor filtro y resúmenes en español).
   - Variable `SITE_URL` (ej. `https://tuusuario.github.io/repo`), habilita sitemap y RSS.
4. Actions → "Actualizar y publicar portal" → Run workflow. Desde ahí corre solo.

## Personalizar

Todo está en `config.json`: nombre, categorías, feeds, palabras bloqueadas, notas nuevas por ejecución (`max_new_per_run`) y notas máximas conservadas (`max_articles`).

## Notas

- Verifica las URLs de los feeds; si una fuente cambia su RSS, el script la omite y sigue con las demás.
- Sin API key el sitio funciona, pero mostrará títulos y resúmenes en el idioma original (inglés) y el filtro será solo por palabras.
- Revisa los términos de uso de cada fuente; el diseño usa solo titular + resumen corto + enlace.

## Marca y logo

Los archivos de `assets/` (logo claro y oscuro, favicon, imagen para compartir) se copian solos a la web publicada. Si borras `assets/logo.png`, la cabecera muestra el nombre del sitio en texto. La paleta y el uso del logo están en `BRAND.md`.

## Longitud del texto de cada nota

- `summary_max_chars` (700): extracto de la fuente que se muestra en la nota.
- `ai_summary_max_chars` (1500): longitud máxima del resumen escrito por Claude con sus propias palabras (solo si se usa la IA; para fuentes en español, `"ai_for_spanish": true`).
- `"full_text": true` en un feed: publica el texto completo de esa fuente. Actívalo solo si la fuente lo permite (licencia Creative Commons o permiso expreso). Por defecto está desactivado en todas.

## Modo historias

A los 10 segundos de abrir la portada aparece un modal tipo historias de Instagram con las últimas notas. Toca la derecha para avanzar y la izquierda para volver; mantener pulsado pausa; la X lo cierra y vuelve al portal. Hay un botón "Ver historias" para reabrirlo. Se ajusta en `config.json`, bloque `stories`: `enabled`, `delay_seconds`, `count` (número de historias), `seconds_per_story` y `once_per_session` (si es `true`, no se vuelve a abrir solo en la misma sesión del navegador).
