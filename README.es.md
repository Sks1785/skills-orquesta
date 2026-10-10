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

La carpeta `plugin/` es un plugin de Claude Code llamado `orquesta`. Incluye las skills `director` y `verificar` en español y dos hooks que recuerdan a Claude usarlas: uno al empezar la sesión y otro en cada mensaje. Ya no hace falta escribir `/director` ni `/verificar`. El plugin es **solo en español**; para otros idiomas usa la instalación manual de arriba.

```bash
claude plugin marketplace add Sks1785/skills-orquesta
claude plugin install orquesta@skills-orquesta
```

O dentro de una sesión de Claude Code: `/plugin marketplace add Sks1785/skills-orquesta` y después `/plugin install orquesta@skills-orquesta`. Luego ejecuta `/reload-plugins` o abre una sesión nueva.

Conviene saber:
- Las skills llevan el prefijo del plugin: `/orquesta:director` y `/orquesta:verificar`.
- Si ya habías copiado `director` / `verificar` a `~/.claude/skills`, muévelas antes, o tendrás duplicados.
- Los hooks solo ejecutan un `echo`. En Windows, Claude Code usa Git Bash si está instalado (probado: una línea limpia). Con el PowerShell de reserva el mismo texto sale una palabra por línea, aunque se sigue leyendo.
- El hook añade un recordatorio corto en cada mensaje (unos pocos tokens cada vez). Para quitarlo, edita o borra `plugin/hooks/hooks.json`. Para desinstalar: `claude plugin uninstall orquesta@skills-orquesta`.
- `plugin/skills/` es una copia de `es/` (mismo texto; solo cambian los finales de línea). Si cambias una, cambia la otra.
- Probado en Windows 11 con Claude Code 2.1.278: `claude plugin validate` pasa, los dos hooks saltan y las dos skills se cargan. Solo se ha probado en una tarea real.

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
