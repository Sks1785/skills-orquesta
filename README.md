# skills-orquesta

Dos skills en español para Claude (Claude Code / Cowork) pensadas para proyectos técnicos: apps Flutter/Android, Docker y seguridad del hogar (cámaras, Frigate, Home Assistant, NAS, SAI).

| Skill | Para qué sirve |
|---|---|
| `verificar` | Obliga a comprobar de verdad (comandos reales, evidencia) antes de decir "ya funciona". Informe con formato fijo: OK / FALLO / NO VERIFICADO. |
| `director` | Un **director** que clasifica el tema, vigila que Claude no se desvíe y exige justificar cada acción, más varios **investigadores** que construyen un cuaderno de notas `.md` con fuentes reales. El director lo consulta con búsquedas precisas, sin leerlo todo. |

## Instalación

Copia cada carpeta dentro de tu carpeta de skills:

```bash
# Para todos tus proyectos
mkdir -p ~/.claude/skills
cp -r verificar director ~/.claude/skills/

# O solo para un proyecto
mkdir -p .claude/skills
cp -r verificar director .claude/skills/
```

Reinicia la sesión de Claude y comprueba que aparecen `verificar` y `director`.

## Cómo usarlas
- Escribe `/director` al empezar una tarea, o deja que Claude la active por su descripción.
- Tras cada cambio importante, `/verificar`.
- Para corregir al director: escribe `corrige:`, `no es así:` o `apunta esto:` y se anota en `conocimiento/_lecciones.md`.

## Límites honestos
- Director e investigadores son la misma IA con contextos separados: no son expertos independientes. Las fuentes obligatorias y la validación reducen errores, no los eliminan.
- Más agentes implican más tiempo y más tokens.
- La búsqueda en el cuaderno es por texto: depende de las etiquetas y sinónimos de cada nota.
- Las skills están en fase de pruebas; se mejoran con los fallos reales (ver sección de mantenimiento y registro de lecciones).
