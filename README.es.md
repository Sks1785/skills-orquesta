# skills-orquesta

Dos skills multi-idioma para Claude (Claude Code / Cowork) pensadas para proyectos técnicos: apps Flutter/Android, Docker y seguridad del hogar (cámaras, Frigate, Home Assistant, NAS, SAI).

| Skill | Para qué sirve |
|---|---|
| `verificar` (`verify`) | Obliga a comprobar de verdad antes de decir "ya está": comandos reales, evidencia y un informe fijo (OK / FALLO / NO VERIFICADO). |
| `director` | Un **director** que clasifica la tarea, corta los desvíos y exige justificar cada acción, más varios **investigadores** que construyen un cuaderno de notas en Markdown (`conocimiento/`) con fuentes reales. El director lo consulta con búsquedas precisas, sin leerlo todo. Los investigadores buscan en todos los lugares posibles (documentación oficial, código e incidencias, foros, reseñas, artículos, tiendas, otros idiomas) y etiquetan cada fuente por tipo y confianza. |

> **Estado: pruebas iniciales.** English: [README.md](README.md)

## Idiomas

Cada idioma es una carpeta completa e independiente (lista en el [README en inglés](README.md#languages)): `en`, `es`, `pt`, `fr`, `de`, `it`, `zh`, `ja`, `ko`, `ru`, `ar`, `hi`, `tr`, `nl`, `pl`, `id`. **Instala un solo idioma**: las skills tienen los mismos nombres y dos idiomas se pisarían.

**Calidad de las traducciones:** el original está en español. El resto lo ha traducido una IA y **no lo han revisado hablantes nativos**. Son estructuralmente idénticas al original, pero el texto puede mejorarse. Se agradecen correcciones y nuevos idiomas mediante pull request.

## Instalación

```bash
# Para todos tus proyectos
mkdir -p ~/.claude/skills
cp -r es/verificar es/director ~/.claude/skills/

# O solo para un proyecto
mkdir -p .claude/skills
cp -r es/verificar es/director .claude/skills/
```

Reinicia la sesión de Claude y comprueba que aparecen `verificar` y `director`.

## Instalar como plugin (español, con recordatorios automáticos)

La carpeta `plugin/` es un plugin de Claude Code llamado `orquesta`. Incluye las skills `director` y `verificar` en español y un hook que recuerda a Claude usarlas al empezar cada sesión. Ya no hace falta escribir `/director` ni `/verificar`. El plugin es **solo en español**; para otros idiomas usa la instalación manual de arriba.

```bash
claude plugin marketplace add Sks1785/skills-orquesta
claude plugin install orquesta@skills-orquesta
```

O dentro de una sesión de Claude Code: `/plugin marketplace add Sks1785/skills-orquesta` y después `/plugin install orquesta@skills-orquesta`. Luego ejecuta `/reload-plugins` o abre una sesión nueva.

Conviene saber:
- Las skills llevan el prefijo del plugin: `/orquesta:director` y `/orquesta:verificar`.
- Si ya habías copiado `director` / `verificar` a `~/.claude/skills`, muévelas antes, o tendrás duplicados.
- El hook solo ejecuta un `echo`. En Windows, Claude Code usa Git Bash si está instalado (probado: una línea limpia). Con el PowerShell de reserva el mismo texto sale una palabra por línea, aunque se sigue leyendo.
- El hook añade un recordatorio corto al empezar la sesión. Para quitarlo, edita o borra `plugin/hooks/hooks.json`. Para desinstalar: `claude plugin uninstall orquesta@skills-orquesta`.
- `plugin/skills/` es una copia de `es/` (mismo texto; solo cambian los finales de línea). Si cambias una, cambia la otra.
- Probado en Windows 11 con Claude Code 2.1.278: `claude plugin validate` pasa, el hook de inicio salta y las dos skills se cargan (la versión 0.1.0 tenía además un hook por mensaje; se quitó en la 0.1.1; la 0.2.0 solo cambia el texto de las skills). Solo se ha probado en una tarea real.

## Novedades de la 0.2.0
- **Tareas triviales:** `director` y `verificar` ya no montan todo el aparato para cambiar un texto o una línea (una comprobación real y una línea de conclusión).
- **Reproducir antes de editar:** `verificar` exige ejecutar el caso que falla y mostrar su salida antes de tocar nada, y repetirlo después.
- **Informe compacto:** cinco líneas obligatorias (criterio, antes del cambio, comprobaciones, NO VERIFICADO y conclusión), máximo 15 líneas.
- **Modo rápido del director:** sin cuaderno ni investigadores, pero con un mínimo obligatorio: fecha de la consulta, URLs realmente abiertas, bloque CONFIRMADO / INFERIDO / NO VERIFICADO y precios o versiones con fecha.
- **Vigencia de las fuentes:** solo es CONFIRMADO lo que viene de una página abierta, con su fecha y versión, y cuya versión encaja con la del usuario. Si la fuente es antigua, de otra versión o sin fecha, va a INFERIDO como "posiblemente desactualizado".

**Qué se midió (y qué no):** en una comparación ciega pequeña (una ejecución por caso, un solo modelo, un juez), esta versión puntuó 44 sobre 50 frente a 37 de la 0.1.1 en dos consultas sensibles a la versión (Home Assistant y Tailscale en un Synology), con el mismo coste en tokens. En una tarea trivial puntuó 20 frente a 16. Es una señal, no una prueba: no se ha medido con otros modelos ni en tareas largas, y los modelos a veces cumplen la regla de fechas solo a medias.

## Actualizar
```bash
claude plugin marketplace update skills-orquesta
claude plugin update orquesta@skills-orquesta
```
Después abre una sesión nueva (o `/reload-plugins`) y comprueba que `/orquesta:director` aparece.

## Opcional: que se use sin escribir `/director`
El plugin solo avisa al empezar la sesión; no obliga a invocar las skills. Para que Claude las use con más fiabilidad, pega esto en tu `CLAUDE.md` global (`~/.claude/CLAUDE.md`). No se ha medido todavía en sesiones reales:

```
## Regla orquesta (obligatoria)

En tareas técnicas (apps, Docker, servidores, cámaras, red, IA, automatización):

1. Antes de investigar o planificar, invoca la skill `orquesta:director` con la herramienta Skill. No lo hagas en cambios triviales (un texto, un nombre, una línea sin datos externos): en ese caso haz el cambio y una comprobación real.
2. Antes de decir "listo", "arreglado" o "terminado", invoca la skill `orquesta:verificar`.
3. Si hay un fallo concreto, reprodúcelo y muestra su salida ANTES de editar nada; repite el mismo caso después.
4. En consultas con búsqueda web, escribe la fecha de la consulta y un bloque final CONFIRMADO / INFERIDO / NO VERIFICADO. Solo es CONFIRMADO lo que viene de una página que abriste, con su fecha y versión, y que encaja con mi versión. Si la fuente es antigua, de otra versión o sin fecha, va a INFERIDO como "posiblemente desactualizado". Precios, stock y versiones llevan fecha y "revalidar antes de comprar o instalar".
5. Informe corto (máximo 15 líneas): criterio de éxito, antes del cambio, comprobaciones [OK]/[FALLO] con el comando, qué NO verificaste y conclusión (VERIFICADO / PARCIAL / NO VERIFICADO).
6. Sin evidencia, di NO VERIFICADO. Nunca "debería funcionar".
```

## Uso
- Empieza una tarea con `/director`, o deja que Claude la active por su descripción.
- Tras cada cambio importante, `/verificar`.
- Para corregir al director: escribe `corrige:`, `no es así:` o `apunta esto:` y se anota en `conocimiento/_lecciones.md`.

## Límites honestos
- Director e investigadores son la misma IA con contextos separados, no expertos independientes. Las fuentes obligatorias y la validación reducen errores, no los eliminan.
- Más agentes implican más tiempo y más tokens.
- La búsqueda en el cuaderno es por texto: depende de las etiquetas y sinónimos, no del significado.
- Las fuentes de comunidad (foros, Reddit, vídeos) cuentan como experiencia de usuarios, no como documentación, y necesitan dos fuentes independientes para darse por confirmadas.

## Licencia
MIT (ver `LICENSE`).
