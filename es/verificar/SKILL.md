---
name: verificar
description: Verifica de verdad que un cambio funciona antes de darlo por terminado. Úsala al crear o modificar apps (Flutter/Android), configuraciones de servidor, Docker, Frigate, Home Assistant, n8n o cualquier proyecto de seguridad y automatización del hogar. Obliga a ejecutar comprobaciones reales y a mostrar evidencia en vez de afirmar que "debería funcionar".
---

# Skill: verificar

## Principio
No eres el verificador por intuición: eres el verificador por EVIDENCIA.
Nunca digas "ya está arreglado", "debería funcionar" o "encontré el problema" sin haber ejecutado una comprobación real y haber visto su resultado.
Si no puedes ejecutar algo, dilo y marca el punto como NO VERIFICADO.

## Protocolo fijo (siempre los mismos 6 pasos, en este orden)

1. **Identificar el proyecto y su perfil.** Decide cuál aplica (ver "Perfiles" abajo). Si no está claro, pregunta una sola vez.
2. **Leer antes de afirmar.** Abre los archivos y la configuración reales afectados. No supongas su contenido. Si vas a decir "el fallo está en X", primero muestra que leíste X.
3. **Definir el criterio de éxito.** Escribe en una o dos líneas qué significa "funciona" para este cambio (ej.: "la app compila y la pantalla de ajustes abre sin errores").
4. **Ejecutar las comprobaciones del perfil.** Usa los comandos reales (no inventados). Si un comando falla, inclúyelo tal cual en el informe.
5. **Probar el caso que antes fallaba.** Reproduce el problema original y confirma que ya no ocurre. Si no puedes reproducirlo, dilo.
6. **Informar con el formato obligatorio** (ver abajo).

## Formato obligatorio del informe

```
VERIFICACIÓN
Proyecto: <nombre/tipo>
Criterio de éxito: <una línea>

Comprobaciones ejecutadas:
- [OK] <comando o prueba> → <resultado relevante>
- [FALLO] <comando o prueba> → <error>
- [NO VERIFICADO] <qué falta y por qué>

Qué leí: <archivos/configs revisados>
Conclusión: VERIFICADO / PARCIAL / NO VERIFICADO
Riesgos pendientes: <lista corta o "ninguno conocido">
Siguiente paso recomendado: <una línea>
```

Reglas del formato:
- Solo marca VERIFICADO si TODAS las comprobaciones críticas están en OK.
- Cita errores literales y cortos, no los resumas de memoria.
- Separa lo comprobado de lo supuesto. Lo supuesto va como "supongo", nunca como hecho.

## Perfil A: apps (Flutter / Android)

Comprobaciones, de menos a más costosas (para en la primera que falle):
1. `flutter analyze` → sin errores (los avisos se listan).
2. `flutter test` → todos los tests pasan. Si no hay tests para lo cambiado, dilo y propón uno.
3. `flutter build apk --debug` (o `./gradlew assembleDebug` en Android nativo) → compila.
4. Si hay emulador/dispositivo disponible: `flutter run` o instalar el APK y probar el flujo afectado.
5. Revisar `adb logcat` (filtrando por el paquete de la app) buscando excepciones al abrir la pantalla tocada.

Extras según el caso:
- Cambios de dependencias: `flutter pub get` sin conflictos; comprueba la versión mínima de SDK.
- Cambios de permisos o notificaciones: revisa `AndroidManifest.xml` y prueba en un dispositivo con Android reciente.
- Antes de subir a Google Play: build de release firmado y comprobación de `versionCode` incrementado.
- Si el cambio toca almacenamiento de datos: prueba con datos existentes (migración), no solo con instalación limpia.

## Perfil B: seguridad y automatización del hogar

Regla de oro: **primero solo lectura**. No cambies configuración en producción sin (a) copia de seguridad, (b) confirmación explícita del usuario y (c) plan de vuelta atrás.

Comprobaciones típicas (ajusta a la instalación real; no asumas que un comando existe, compruébalo):
- **Docker / docker-compose:** `docker compose config` (valida sintaxis), `docker compose ps` (estado), `docker logs --tail 100 <contenedor>` (errores recientes). Revisa rutas de volúmenes y puertos antes de levantar nada.
- **Frigate:** valida el archivo de configuración con la propia herramienta de Frigate si la hay; comprueba en los logs que cada cámara conecta; confirma que la detección está activa y que las grabaciones se escriben en disco. Comprueba uso de CPU si detecta por CPU.
- **Cámaras:** confirma que el flujo RTSP responde (por ejemplo con `ffprobe` sobre la URL, sin mostrar credenciales en el informe). Revisa el presupuesto PoE del switch (suma de consumo de cámaras frente a la potencia total).
- **Home Assistant:** ejecuta la comprobación de configuración antes de reiniciar; después revisa el registro por errores de integraciones.
- **n8n u otras automatizaciones:** ejecuta el flujo en modo de prueba con datos de ejemplo y comprueba la salida de cada nodo; confirma zona horaria y credenciales.
- **NAS, red y SAI:** comprueba conectividad (`ping`, puertos abiertos), espacio libre en disco, y estado del SAI (por ejemplo con `upsc` si usa NUT). Tras un corte de luz simulado o real, confirma que el apagado ordenado funciona.
- **Seguridad:** no expongas cámaras, paneles ni el NAS a internet sin autenticación fuerte y, mejor, VPN. Revisa puertos abiertos y alertas del propio sistema. Nunca incluyas contraseñas, tokens ni URLs con credenciales en el informe.

Para cualquier cambio de red o firewall: comprueba antes de aplicar que no te quedas sin acceso (ten una vía de vuelta).

## Qué hacer cuando no se puede verificar
- Di exactamente qué no pudiste comprobar y por qué (sin acceso al dispositivo, sin red, sin permisos).
- Propón el comando o prueba concreta que el usuario puede ejecutar y qué resultado esperar.
- Marca la conclusión como PARCIAL o NO VERIFICADO. No rellenes huecos con suposiciones.

## Anti-patrones (prohibido)
- Decir "ya funciona" sin ejecutar nada.
- Dar la causa de un fallo sin haber leído el código o el log relevante.
- Inventar nombres de comandos, opciones, archivos o puertos. Si no estás seguro, compruébalo o pregunta.
- Cambiar varias cosas a la vez: un cambio, una verificación.
- Ocultar comprobaciones fallidas del informe.

## Mantenimiento de esta skill
Cada vez que un fallo se te escape (algo "verificado" que luego falla), añade aquí una comprobación nueva que lo habría detectado. Esta lista se mejora con los errores reales.
